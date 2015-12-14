% Items de Lenguaje

> **Nota**: los items de lenguaje son normalmente proporcionados por crates en
> la distribución Rust, dichos items en si mismos poseen una interfaz inestable.
> Es recomendable usar crates distribuidos oficialmente en lugar de definir tus
> propios.

El compilador `rustc` posee ciertas operaciones plugables, esto es, funcionalidad que no esta escrita de manera definitiva en el lenguaje, al contrario, esta implementada en bibliotecas, con un marcador especial para informar al compilador acerca de su existencia. Dicho marcador es el atributo  `#[lang = "..."]` y hay varios valores diferentes de `...`, e.j. diferentes 'items de lenguaje'.

Por ejemplo, los apuntadores `Box` requieren dos items de lenguaje, uno para la asignación de memoria y otro para la liberación. Un programa libre (sin la biblioteca estándar) que hace uso de la sintaxis `Box` para asignación dinámica de memoria a través de `malloc` y `free` luce así:

```rust
#![feature(lang_items, box_syntax, start, libc)]
#![no_std]

extern crate libc;

extern {
    fn abort() -> !;
}

#[lang = "owned_box"]
pub struct Box<T>(*mut T);

#[lang = "exchange_malloc"]
unsafe fn asignar(tamano: usize, _alineacion: usize) -> *mut u8 {
    let p = libc::malloc(tamano as libc::size_t) as *mut u8;

    // malloc fallo
    if p as usize == 0 {
        abort();
    }

    p
}
#[lang = "exchange_free"]
unsafe fn liberar(ptr: *mut u8, _tamano: usize, _alineacion: usize) {
    libc::free(ptr as *mut libc::c_void)
}

#[start]
fn main(argc: isize, argv: *const *const u8) -> isize {
    let x = box 1;

    0
}

#[lang = "eh_personality"] extern fn eh_personality() {}
#[lang = "panic_fmt"] fn panic_fmt() -> ! { loop {} }
# #[lang = "eh_unwind_resume"] extern fn rust_eh_unwind_resume() {}
# #[no_mangle] pub extern fn rust_eh_register_frames () {}
# #[no_mangle] pub extern fn rust_eh_unregister_frames () {}
```

Nota el uso de `abort`: se asume que el item de lenguaje `exchange_malloc` retorna un apuntador valido, y es por ello que es necesario hacer el chequeo interno.

Otras facilidades proporcionadas por items de lenguaje son:

- operadores sobrecargables via traits: los traits correspondientes a los operadores `==`, `<`, deferencia (`*`) y `+` (etc.) están todos marcados con items de lenguaje; esos cuatro en especifico son `eq`, `ord`, `deref` y `add` respectivamente.
- el unwinding de la pila y fallas generales; los items de lenguaje `eh_personality`, `fail` y `fail_bounds_checks`.
- los traits en `std::marker` usados para indicar tipos de varias classes; items de lenguaje `send`, `sync` y `copy`.
- los tipos marcador e indicadores de varianza encontrados en `std::marker`; items de lenguaje `covariant_type`, `contravariant_lifetime`, etc.

Los items de lenguaje son cargados perezosamente por el compilador; e.j. si alguien no necesita `Box` entonces no hay necesidad de definir funciones para `exchange_malloc` y `exchange_free`. `rustc` emitirá un error cuando un item de lenguaje sea necesario pero no este presente en el crate actual o alguno en el cual este dependa.
