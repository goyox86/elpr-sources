% Unsafe

La principal atracción de Rust son sus poderosas garantías estáticas acerca de comportamiento. Pero los chequeos de seguridad son conservadores por naturaleza: existen programas que son en efecto seguros, pero el compilador no es capaz de de verificar que esto sea cierto. Para escribir ese tipo de programas, debemos decirle al compilador que relaje un poco sus restricciones. Para ello, Rust posee una palabra reservada, `unsafe`. El código que hace uso de `unsafe` posee menos restricciones que el código normal.

Repasemos la sintaxis, y luego hablaremos de la semántica. `unsafe` es usado en cuatro contextos. El primero es para marcar una función como insegura:

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

Y la cuarta es para la `impl`ementación de uno de dichos traits:


```rust
# unsafe trait Peligroso { }
unsafe impl Peligroso for i32 {}
```

Es importante poder tener la capacidad de delinear código que podría posiblemente contener bugs que originen problemas graves. Si un programa Rust termina de manera abrupta (un segfault), puedes tener por seguro que es en algún lugar de las secciones marcadas como `unsafe`.

# Que significa ‘seguro’?

Seguro, en el contexto de Rust, significa ‘no hacer nada inseguro’. Es importante saber que hay ciertos comportamientos que son probablemente indeseables en tu código, pero son expresamente _no_ inseguros:

* Deadlocks.
* Perdida de memoria u otros recursos.
* Salida sin llamada a los destructores.
* Desbordamiento de enteros.

Rust no puede prevenir todos los tipos de problemas de software. Las cosas de la lista anterior no son buenas, pero tampoco califican como `unsafe` específicamente.

En adición, los siguientes son todos comportamiento indefinido en Rust, y deben ser evitadas, incluso cuando se escribe código `unsafe`:

* Condiciones de carrera.
* Deereferenciar un apuntador nulo/colgante.
* Lectura de memoria [undef][undef] (memoria no inicializada)
* Violación de las [reglas de aliasing de apuntadores][aliasing] a través de apuntadores planos.
* `&mut T` y `&T` siguen el modelo [noalias][noalias] de LLVM, excepto cuando el `&T` contiene un `UnsafeCell<U>`. El código unsafe no debe violar esas garantías de aliasing.
* Mutar un valor/referencia inmutable sin un `UnsafeCell<U>`.
* Invocar comportamiento indefinido a través de intrínsecos del compilador:
  * Indexar por fuera de los limites de un objeto con `std::ptr::offset` (intrínseco `offset`), con la excepción de un solo byte después del final lo cual es permitido.
  * Usar `std::ptr::copy_nonoverlapping_memory` (intrínsecos `memcpy32`/`memcpy64`) en buffers que se solapen.
* Valores inválidos en tipos primitivos, incluso en campos/variables locales privadas:
  * Referencias null, referencias colgantes o boxes.
  * Un valor distinto que `false` (0) o `true` (1) en un `bool`.
  * Un discriminante en un `enum` que no este incluido en su definición de tipo.
  * Un valor en un `char` el cual es un sustituto o por encima de `char::MAX`.
  * Una secuencia de bytes no-UTF en un `str`.
* Unwinding en Rust desde código foraneo o unwinding desde Rust a código foraneo.

[noalias]: http://llvm.org/docs/LangRef.html#noalias
[undef]: http://llvm.org/docs/LangRef.html#undefined-values
[aliasing]: http://llvm.org/docs/LangRef.html#pointer-aliasing-rules

# Superpoderes Unsafe

En ambos funciones y bloques unsafe, Rust te permitirá hacer tres cosas que normalmente no podrías hacer. Solo tres. Y son:

1. Acceder o actualizar una [variable mutable estética][static].
2. Dereferenciar un apuntador plano.
3. Llamar a funciones `unsafe`. Esta es la habilidad mas importante.

Eso es todo. Es importante destacar que `unsafe`, por ejemplo, no ‘apaga el comprobador de prestamos’. Agregar de manera aleatoria `unsafe` a algún código no cambia su semántica, no comenzara a aceptar algo. Pero te permitirá escribir cosas que _si rompen_ algunas de las reglas.

También encontraras la palabra reservada `unsafe` cuando escribas bindings a interfaces foráneas (no-Rust). Lo mas recomendable es escribir una segura interfaz nativa en Rust alrededor de los métodos proporcionados por la librería.

Echemos un vistazo a las tres habilidades listadas, en orden.

## Acceder o actualizar una `static mut`

Rust posee una facilidad denominada ‘`static mut`’ que te permite hacer estado global mutable. Hacerlo puede causar una condición de carrera, y en consecuencia es inherentemente inseguro. Para mayor detalle, dirígete a la sección [static][static] del libro.

[static]: const-and-static.html#static

## Dereferenciar un apuntador plano

Los apuntadores planos te permiten llevar a cabo aritmética de punteros arbitraria, y pueden causar un numero de problemas de seguridad. En algunos sentidos, la habilidad de dereferenciar un apuntador arbitrario es una de las cosas mas peligrosas que puedes hacer, mas información en [su sección en el libro][rawpointers].

[rawpointers]: raw-pointers.html

## Llamar funciones unsafe

Esta ultima habilidad funciona con ambos aspectos de `unsafe`: puedes solo llamar a funciones marcadas como `unsafe` desde dentro de un bloque unsafe.

Esta habilidad es poderosa y variada. Rust expone algunos [intrínsecos del compilador][intrinsics] como funciones unsafe, y algunas funciones unsafe hacen bypass de algunos chequeos de seguridad, intercambiando seguridad por velocidad.

Lo repetiré de nuevo: aun y cuando _puedes_ hacer cosas arbitrarias en bloques unsafe y funciones no significa que debas hacerlo. El compilador actuara como si tu eres el responsable estuvieses de mantener arriba todas las invariantes, así que debes ser cuidadoso!

[intrinsics]: intrinsics.html
