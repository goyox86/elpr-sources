% Tipos sin Tamaño

La mayoría de los tipos posee un tamaño particular, en bytes, que es conocido en tiempo de compilación. Por ejemplo un `i32` tiene un tamaño de treinta y dos bits, o cuatro bytes. Sin embargo, hay algunos tipos que son útiles de expresar, pero no tienen un tamaño definido. Dichos tipos son denominados tipos ‘sin tamaño’ (‘unsized’) o ‘de tamaño dinámico’ (‘dynamically sized’). Un ejemplo es `[T]`. Dicho tipo representa cierto numero de `T` en secuencia. Pero no sabemos cuantos de ellos hay, y en consecuencia el tamaño es desconocido.

Rust entiende algunos de estos tipos, pero tienen algunas restricciones. Son tres:

1. Solo podemos manipular una instancia de un tipo sin tamaño a través de un apuntador. Un `&[T]` funciona perfecto, pero un `[T]` no.
2. Las variables y argumentos no pueden tener tipos de tamaño dinámico.
3. Solo el ultimo campo en un `struct` puede tener un tipo de datos de tamaño dinámico; los otros campos no. Las variantes de enums no deben tener tipos de tamaño dinámico como data.

Entonces, porque molestarse? Bueno, debido a que `[T]` puede solo ser usado detrás de un apuntador, de no tener soporte a nivel de lenguaje para tipos sin tamaño, seria imposible escribir lo siguiente:

```rust,ignore
impl Foo for str {
```

o

```rust,ignore
impl<T> Foo for [T] {
```

En su lugar, tendrías que escribir:

```rust,ignore
impl Foo for &str {
```

Significando que esta implementación solo funcionaria para [referencias][ref], y no otro tipos de apuntadores. Con el `impl for str`, todos los apuntadores, incluyendo (en algún punto, hay algunos bugs que arreglar primero) apuntadores inteligentes definidos por el usuario, pueden usar este `impl`.

[ref]: references-and-borrowing.html

# ?Sized

Si deseas escribir una función que acepte un tipo de datos de tamaño dinámico, puedes usar limite de trait especial, `?Sized`:

```rust
struct Foo<T: ?Sized> {
    f: T,
}
```

Este `?`, leído como “T puede ser `Sized`”, significa que este limite de trait es especial: nos permite hacer match con mas tipos, no menos. Es justo casi como que cualquier `T` implícitamente es `T: Sized`, y el `?` deshace este comportamiento por defecto.
