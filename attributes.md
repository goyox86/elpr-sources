% Atributos

Las declaraciones pueden ser anotadas con ‘atributos’ en Rust. Los atributos lucen así:

```rust
#[test]
# fn foo() {}
```

o de esta manera:

```rust
# mod foo {
#![test]
# }
```

La diferencia entre los dos es el `!`, que cambia a que cosa aplica el atributo:

```rust,ignore
#[foo]
struct Foo;

mod bar {
    #![bar]
}
```

El atributo `#[foo]` aplica a el siguiente item, que es la declaración del `struct`. El atributo `#![bar]` aplica a el item que lo encierra, la declaración `mod`. De resto, son lo mismo. Ambos cambian de alguna forma el significado de el item al cual están asociados.

Por ejemplo, considera una función como esta:

```rust
#[test]
fn comprobar() {
    assert_eq!(2, 1 + 1);
}
```

Esta marcado con `#[test]`. Esto significa que es especial: cuando ejecutes las [pruebas][tests], esta función sera ejecutada. Cuando compilas normalmente, no sera incluida ni siquiera. La función es ahora una función de prueba.


[tests]: testing.html

Los atributos pueden tener también data adicional:

```rust
#[inline(always)]
fn fn_super_rapida() {
# }
```

O incluso claves y valores:

```rust
#[cfg(target_os = "macos")]
mod solo_macos {
# }
```

Los atributos son usados para un numero de cosas diferentes en Rust. Hay una lista de atributos completa en la [referencia][reference]. Actualmente, no tienes permitido crear tus propios atributos, el compilador de Rust los define.

[reference]: ../reference.html#attributes
