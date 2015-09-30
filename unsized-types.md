% Tipos sin Tamaño

La mayoria de los tipos posee un tamano particular, en bytes, que es conocido en tiempo de compilacion. Por ejemplo un `i32` tiene un tamano de treinta y dos bits, o cuatro bytes. Sin embargo, hay algunos tipos que son utilies de expresar, pero no teiene un tamano definido. Dichos tipos son denominados tipos ‘sin tamano’ (‘unsized’) o ‘de tamano dinamico’ (‘dynamically sized’). Un ejemplo es `[T]`. Dicho tipo representa cierto numero de `T` en secuencia. Pero no sabemos cuantos de ellso hay, y en consecuancia el tamano es desconocido.

Rust entiende algunos de estos tipos, pero tienen algunas restricciones. Son tres:

1. Solo podemos manipular una instancia de un tipo sin tamano a traves de un apuntador. Un `&[T]` funciona perfecto, pero un `[T]` no.
2. las variables y argumentos no pueden tener tipos de tamano dinamico.
3. Solo el ultimo campo en un `struct` puede tener un tipo de datos de tamano dinamico; los otros campos no. Las variantes de enums no deben tener tipos de tamano dinamico como data.

Entonces, porque moletarse? Bueno, debido a que `[T]` puede solo ser usado detras de un apuntador, de no tener soporte a nivel de lenguaje para tipos sin tamano, seria imposible escribir lo siguiente:

```rust,ignore
impl Foo for str {
```

o

```rust,ignore
impl<T> Foo for [T] {
```

En su lugar, tendrias que escribir:

```rust,ignore
impl Foo for &str {
```

Significando que esta implementacion solo funcionaria para [referencias][ref], y no otro tipos de apuntadores. Con el `impl for str`, todos los apuntadores, incluyendo (en algun punto, hay algunos bugs que arreglar primero) apuntadores inteligentes definidos por el usuario, pueden usar este `impl`.

[ref]: references-and-borrowing.html

# ?Sized

Si deseas escribir una funcion que acepte un tipo de datos de tamano dinamico, puedes usar  limite de trait especial, `?Sized`:

```rust
struct Foo<T: ?Sized> {
    f: T,
}
```

Este `?`, leido como “T puede ser `Sized`”, significa qye este limite de trait es especial: nos permite hacer match con mas tipos, no menos. Es justo casi como que cualquier `T` implicitamente es `T: Sized`, y el `?` deshace este comportamiento por defecto.
