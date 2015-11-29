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
  code.

[noalias]: http://llvm.org/docs/LangRef.html#noalias
[undef]: http://llvm.org/docs/LangRef.html#undefined-values
[aliasing]: http://llvm.org/docs/LangRef.html#pointer-aliasing-rules

# Unsafe Superpowers

In both unsafe functions and unsafe blocks, Rust will let you do three things
that you normally can not do. Just three. Here they are:

1. Access or update a [static mutable variable][static].
2. Dereference a raw pointer.
3. Call unsafe functions. This is the most powerful ability.

That’s it. It’s important that `unsafe` does not, for example, ‘turn off the
borrow checker’. Adding `unsafe` to some random Rust code doesn’t change its
semantics, it won’t just start accepting anything. But it will let you write
things that _do_ break some of the rules.

You will also encounter the `unsafe` keyword when writing bindings to foreign
(non-Rust) interfaces. You're encouraged to write a safe, native Rust interface
around the methods provided by the library.

Let’s go over the basic three abilities listed, in order.

## Access or update a `static mut`

Rust has a feature called ‘`static mut`’ which allows for mutable global state.
Doing so can cause a data race, and as such is inherently not safe. For more
details, see the [static][static] section of the book.

[static]: const-and-static.html#static

## Dereference a raw pointer

Raw pointers let you do arbitrary pointer arithmetic, and can cause a number of
different memory safety and security issues. In some senses, the ability to
dereference an arbitrary pointer is one of the most dangerous things you can
do. For more on raw pointers, see [their section of the book][rawpointers].

[rawpointers]: raw-pointers.html

## Call unsafe functions

This last ability works with both aspects of `unsafe`: you can only call
functions marked `unsafe` from inside an unsafe block.

This ability is powerful and varied. Rust exposes some [compiler
intrinsics][intrinsics] as unsafe functions, and some unsafe functions bypass
safety checks, trading safety for speed.

I’ll repeat again: even though you _can_ do arbitrary things in unsafe blocks
and functions doesn’t mean you should. The compiler will act as though you’re
upholding its invariants, so be careful!

[intrinsics]: intrinsics.html
