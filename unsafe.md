% Unsafe

La principal atraccion de Rust son sus poderosas garantias estaticas acerca de comportamiento. Pero los cheuqeos de seguridad son conservadores por naturaleza: existen programas que son efecto seguros, pero el compilador no es capaz de de verificar que esto es cierto. Para escribir ese tipo de programas, debemos decirle al compilador que relaje un poco sus restricciones. Para ello, Rust posee una palabra reservada, `unsafe`. El codigo que hace uso de `unsafe` posee menos restricciones que las que el codigo normal.

Repasemos la sintaxis, y luego hablaremos de la semantica. `unsafe` es usado en cuatro contextos. El primero es para marcar ua funcion como insegura:

```rust
unsafe fn peligro_will_robinson() {
    // cosas peligrosas
}
```

Todas las funciones llamadas desde [FFI][ffi] deben ser marcadas como `unsafe`, por ejemplo. El segundo uso de `unsafe` es un bloque unsafe:

[ffi]: ffi.html

```rust
unsafe {
    // osas peligrosas
}
```

El tercero es para traits unsafe:

```rust
unsafe trait Peligroso { }
```

Y la cuarta es para la `impl`ementacion de uno de esos traits:


```rust
# unsafe trait Peligroso { }
unsafe impl Peligroso for i32 {}
```

Es importante poder tener la capacidad de delinear codigo que posiblemente podria contener bugs que causen problemas graves. Si un programa Rust termina de manera abrupta (a segfault), puedes tener por seguro que es en algun lugar de las secciones marcadas como `unsafe`.

# Que significa ‘seguro’?

Seguro, en el contexto de Rust, singifica ‘no hace nada inseguro’. Es importante saber que hay ciertos comportamientos que son probablemente indeseables en tu codigo, pero son expresamente _no_ inseguros:

* Deadlocks
* Perdida de memoria u otros recursos
* Salida sin llamada a los destructores
* Desbordamiento de enteros

Rust no puede prevenir todos los tipos de problemas de software. Esas cosas no son buenas, pero tampoco califican como `unsafe` especificamente.

En adicion, las siguientes son todas comportamiento indefinido en Rust, y deben ser evitadas, incluso cuando se escribe codigo `unsafe`:

* Condiciones de carrera.
* Deereferenciar un apundador nulo/colgante.
* Lectura de memoria [undef][undef] (memoria no inicializada)
* Ruptura de las [reglas de aliasing de apuntadores][aliasing] a traves de apuntadores planos.
* `&mut T` y `&T` siguen LLVM’s el model [noalias][noalias], excepto cuando el `&T` contiene un `UnsafeCell<U>`. Codigo unsafe no debe violar esas garantias de aliasing.
* Mutar un valor/referencia inmutable sin un `UnsafeCell<U>`
* Invocar comportamiento indefinido a traves de intrinsecos del compilador:
  * Indexar por fuera de los limites de un objeto con `std::ptr::offset` (intrinseco `offset`), con la excepcion de un solo byte despues del final lo cual es permitido.
  * Usar `std::ptr::copy_nonoverlapping_memory` (intrinsecos `memcpy32`/`memcpy64`) en buffers hagan overlapping.
  * Using `std::ptr::copy_nonoverlapping_memory` (`memcpy32`/`memcpy64`
* Valores invalidos en tipos primitivos, incluso en campos/locales privados:
  * Referencias null, referencias colgantes o boxes.
  * Un valor distinto que `false` (0) o `true` (1) en un `bool`
  * Un discriminante en un `enum` que no este incluido en su definicion de tipo.
  * Un valor en un `char` el cual es un susituto o por encima de `char::MAX`.
  * Una secuencia de bytes no-UTF  en un `str`.
* Unwinding en Rust desde codigo foraneo o unwinding desde Rust a codigo foraneo.

[noalias]: http://llvm.org/docs/LangRef.html#noalias
[undef]: http://llvm.org/docs/LangRef.html#undefined-values
[aliasing]: http://llvm.org/docs/LangRef.html#pointer-aliasing-rules

# Superpoderes Unsafe

En ambos funciones y bloques unsafe, Rust te permitira hacer tres cosas que normalmente no podrias hacer. Solo tres. Son estas:

1. Acceder o actualizar una [variable mutable estatica][static].
2. Dereferenciar un apuntador plano.
3. Llamar a funciones `unsafe`. Esta es la habilidad mas importante.

Eso es todo. Es importante que `unsafe` no, por ejemplo, ‘apaga el comprobador de prestamos’. Agregar `unsafe` a algun codigo al azar no cambia su semantica, no comenzara a aceptar algo. Pero te permitira escribir cosas que _si rompen_ algunas de las reglas.

Tambien encontraras la palabra reservada `unsafe` cuando escribas bindings a interfaces foraneas (no-Rust). Lo mas recomendable es escribir una segura interfaz nativa en Rust alrededor de los metodos proporcionados por la libreria.

Echemos un vistazo a las tres habilidades listadas, en orden.

## Acceder o actualizar una `static mut`

Rust posee una facilidad denominada ‘`static mut`’ que te permite hacer estado global mutable. Hacerklo puede causar una condicion de carrera, y en consecuencia es inhrerentemente inseguro. Para mayor detalle, dirigete a la seccion [static][static] del libro.

[static]: const-and-static.html#static

## Dereferenciar un apuntador plano

Los apuntadores planos te permiten levvar a cabo aritmetica de punteros arbitraria, y pueden causar un numero de problemas de seguridad. En algunos sentidos, la habilidad de dereferenciar un apuntador arbitrario es una de las cosas mas peligrosas que puedes hacer, mas informacion en [su seccion en el libro][rawpointers].

[rawpointers]: raw-pointers.html

## Llamar funciones unsafe

Esta ultima habilidad funciona con ambos aspectos de `unsafe`: puedes solo llamar a funciones marcadas como `unsafe` desde dentro de un bloque unsafe.

Esta habilidad es poderosa y variada. Rust expone algunos [intrinsecos del compilador][intrinsics] como funciones unsafe, y algunas funciones unsafe hacen bypass de algunos chequeos de seguridad, intercambiando seguridad por velocidad.

Lo repetire de nuevo: aun cuando _puedes_ hacer cosas arbitrarias en bloques unsafe y funciones no significa que debas hacerlo. El compilador actuara como si tu estuvieses manteniendo las invariantes, asi que se cuidadoso!

[intrinsics]: intrinsics.html
