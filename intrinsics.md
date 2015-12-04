% Intrínsecos

> **Nota**: los intrínsecos por siempre tendrán una interfaz inestable, se recomienda
> usar las interfaces estables de libcore en lugar de intrínsecos directamente.

Estas son importadas como si fuesen funciones FFI, con el ABI especial `rust-intrinsic`. Por ejemplo, si uno estuviese en un contexto libre, pero desease poder hacer `transmute` entre tipos, y llevar a cabo aritmética de apuntadores eficiente, uno importaría dichas funciones via una declaración como:

```rust
#![feature(intrinsics)]
# fn main() {}

extern "rust-intrinsic" {
    fn transmute<T, U>(x: T) -> U;

    fn offset<T>(dst: *const T, offset: isize) -> *const T;
}
```

Al igual que cualquier otra función FFI, estas son siempre `unsafe` de llamar.
