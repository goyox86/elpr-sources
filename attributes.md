% Atributos

Las declaraciones pueden ser anotadas con ‘atributos’ en Rust. Los atributos lucen asi:

```rust
#[test]
# fn foo() {}
```

o asi:

```rust
# mod foo {
#![test]
# }
```

La diferencia entre los dos es el `!`, que cambia a que aplica el atributo:

```rust,ignore
#[foo]
struct Foo;

mod bar {
    #![bar]
}
```

El atributo `#[foo]` aplica a el siguente item, que es la declaracion del `struct`. El atributo `#![bar]` aplica a el item que lo encierra, la declaracion `mod`. De resto, son lo mismo. Ambos cambian de alguna forma el significado de el item al cual estan asociados.

Por ejemplo, considera una funcion como esta:

```rust
#[test]
fn check() {
    assert_eq!(2, 1 + 1);
}
```

Esta marcado con `#[test]`. Esto singifica que es especial: cuando ejecutes las [pruebas][tests], esta funcion sera ejecutada. Cuando compilas normalmente, no sera incluida ni siquiera . La funcion es ahora una funcion de prueba.


[tests]: testing.html

Los atributos pueden tener tambien data adicional:

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
