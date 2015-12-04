% Intrínsecos

> **Nota**: los intrinsecos por siempre tendran una interfaz inestable, se recomienda
> usar las interfaces estables de libcore en lugar de intrinsecos directamente.


Estas son importadas como si fueren funciones FFI, con el ABI especial `rust-intrinsic`. Por ejemplo, si uno estuviese en un contexto libre, pero desease porder hacer `transmute` entre tipos, y llevar a cabo aritmetica de apuntadores eficiente, uno importria dichas funciones via una declaracion como:

```rust
#![feature(intrinsics)]
# fn main() {}

extern "rust-intrinsic" {
    fn transmute<T, U>(x: T) -> U;

    fn offset<T>(dst: *const T, offset: isize) -> *const T;
}
```

Al igual que cualquier otra funcion FFI, estas son siempre `unsafe`.
