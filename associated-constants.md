% Constantes Asociadas

Con el feature `associated_consts`, puedes definir constantes como:

```rust
#![feature(associated_consts)]

trait Foo {
    const ID: i32;
}

impl Foo for i32 {
    const ID: i32 = 1;
}

fn main() {
    assert_eq!(1, i32::ID);
}
```

Cualquier implementador de `Foo` tendrá que definir `ID`. Sin la definición:

```rust,ignore
#![feature(associated_consts)]

trait Foo {
    const ID: i32;
}

impl Foo for i32 {
}
```

resulta en

```text
error: not all trait items implemented, missing: `ID` [E0046]
     impl Foo for i32 {
     }
```

Un valor por defecto puede también ser implementado:

```rust
#![feature(associated_consts)]

trait Foo {
    const ID: i32 = 1;
}

impl Foo for i32 {
}

impl Foo for i64 {
    const ID: i32 = 5;
}

fn main() {
    assert_eq!(1, i32::ID);
    assert_eq!(5, i64::ID);
}
```

Como puedes ver, al implementar `Foo`, puedes dejarlo sin implementación, como en el caso de `i32`. Entonces este usara el valor por defecto. Pero, también y al igual que como en `i64`, podemos agregar nuestra propia definición.

Las constantes asociadas no necesitan solo funcionan con traits. Un bloque `impl` para una `struct` o un `enum` es también valido:

```rust
#![feature(associated_consts)]

struct Foo;

impl Foo {
    const FOO: u32 = 3;
}
```
