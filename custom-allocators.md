% Asignadores de Memoria Personalizados

La asignación de memoria no es siempre la cosa mas fácil de hacer, y si bien Rust generalmente lo hace por defecto en algunas oportunidades se hace necesario personalizar como ocurre la asignación de memoria. El compilador y la biblioteca estándar actualmente permiten cambiar el asignador global de memoria por defecto a usar en tiempo de compilación. El diseño esta actualmente delineado en el [RFC 1183][rfc] pero esta sección te dará un tour acerca de como poner tu asignador de memoria personalizado en funcionamiento.

[rfc]: https://github.com/rust-lang/rfcs/blob/master/text/1183-swap-out-jemalloc.md

# Asignador por defecto

El compilador actualmente viene con dos asignadores de memoria por defecto: `alloc_system` y `alloc_jemalloc` (algunos sistemas, sin embargo, no soportan jemalloc). Dichos asignadores son crates regulares Rust y contienen la implementación para la asignación y liberación de memoria. La biblioteca estándar no es compilada asumiendo ningún asignador en particular, y el compilador decidirá cual asignador esta en uso en tiempo de compilación dependiendo del tipo de artefacto de salida que este siendo producido.

Los ejecutables generados por el compilador usaran `alloc_jemalloc` por defecto (donde este disponible). En esta situación el compilador "controla el mundo" en el sentido que posee poder acerca del enlace final. Esto principalmente significa que la decisión de cual asignador es usado puede ser delegada al compilador.

Las bibliotecas estáticas y dinámicas, sin embargo, harán uso de el asignador `alloc_system` por defecto. En esta situación Rust es típicamente un 'invitado' dentro de otra aplicación u otro mundo en el cual no puede de manera autoritaria decidir cual asignador de memoria usar. Como resultado recurre a las APIs estándar (e.j. `malloc` y `free`) para adquirir y liberar memoria.

# Cambiando Asignadores

Si bien las elecciones del compilador pueden funcionar la mayoría del tiempo, algunas veces es necesario personalizar ciertos aspectos. Sobreescribir la decisión acerca de cual asignador se debe usar puede hacerse simplemente enlazando con el asignador deseado:

```rust,no_run
#![feature(alloc_system)]

extern crate alloc_system;

fn main() {
    let a = Box::new(4); // asigna desde el asignador de memoria del sistema
    println!("{}", a);
}
```

En este ejemplo el ejecutable generado no enlazara con jemalloc por defecto y en su lugar usara el asignador del sistema. De manera similar, para generar una biblioteca dinámica que use jemalloc por defecto uno podría escribir:

```rust,ignore
#![feature(alloc_jemalloc)]
#![crate_type = "dylib"]

extern crate alloc_jemalloc;

pub fn foo() {
    let a = Box::new(4); // asigna desde jemalloc
    println!("{}", a);
}
# fn main() {}
```

# Escribiendo un asignador personalizado

Algunas veces las opciones de jemalloc vs el asignador del sistema no son suficientes y un asignador completamente nuevo se hace necesario. En este caso escribirías tu propio crate implementando el API de asignación de memoria. (e.j lo mismo que `alloc_system` o `alloc_jemalloc`). Como ejemplo, echemos un vistazo a una version simplificada y anotada de `alloc_system`

```rust,no_run
# // necesario solo para rustdoc --test
# #![feature(lang_items)]
// El compilador necesita ser instruido acerca del hecho que este crate es un
// asignador para que entienda que cuando se enlace con el no se debe enlazar
// con otro asignador como jemalloc

#![feature(allocator)]
#![allocator]

// Los asignadores de memoria no tienen permitido depender de la biblioteca
// estándar que a su vez requiere un asignador con la finalidad de evitar
// dependencias cíclicas. Este crate, sin embargo, puede hacer uso de todo
// en libcore.
#![no_std]

// Démosle un nombre a nuestro asignador personalizado
#![crate_name = "my_allocator"]
#![crate_type = "rlib"]

// Nuestro asignador de sistema hará uso de el crate libc que vive en el árbol
// de Rust. Nota que actualmente el crate libc externo (crates.io) no puede ser
// usado debido a que este enlaza con la biblioteca estándar (e.j. `#![no_std]`
// no esta estable todavía). Es por ello que requiere específicamente la 
// version en el arbol.
#![feature(libc)]
extern crate libc;

// A continuación se listan las cinco funciones de asignación requeridas
// actualmente por los asignadores de memoria. Los tipos en sus firmas y
// nombres de símbolo actualmente no son chequeados por el compilador,
// pero esta es una extension futura y son requeridas para coincidir con
// lo que sigue a continuación.
//
// Nota que las funciones `malloc` y `realloc` estándar no proveen una
// via para comunicar la alineación y es por ello que esta implementación
// necesitaría ser mejorada en lo que respecta a la alineación.

#[no_mangle]
pub extern fn __rust_allocate(size: usize, _align: usize) -> *mut u8 {
    unsafe { libc::malloc(size as libc::size_t) as *mut u8 }
}

#[no_mangle]
pub extern fn __rust_deallocate(ptr: *mut u8, _old_size: usize, _align: usize) {
    unsafe { libc::free(ptr as *mut libc::c_void) }
}

#[no_mangle]
pub extern fn __rust_reallocate(ptr: *mut u8, _old_size: usize, size: usize,
                                _align: usize) -> *mut u8 {
    unsafe {
        libc::realloc(ptr as *mut libc::c_void, size as libc::size_t) as *mut u8
    }
}

#[no_mangle]
pub extern fn __rust_reallocate_inplace(_ptr: *mut u8, old_size: usize,
                                        _size: usize, _align: usize) -> usize {
    old_size // this api is not supported by libc
}

#[no_mangle]
pub extern fn __rust_usable_size(size: usize, _align: usize) -> usize {
    size
}

# // necesario solo para hacer que rustdoc corra las pruebas
# fn main() {}
# #[lang = "panic_fmt"] fn panic_fmt() {}
# #[lang = "eh_personality"] fn eh_personality() {}
# #[lang = "eh_unwind_resume"] extern fn eh_unwind_resume() {}
# #[no_mangle] pub extern fn rust_eh_register_frames () {}
# #[no_mangle] pub extern fn rust_eh_unregister_frames () {}
```

Despues que compilamos este crate, puede ser usado como sigue:

```rust,ignore
extern crate my_allocator;

fn main() {
    let a = Box::new(8); // asigna memoria via nuestro asignador
    println!("{}", a);
}
```

# Limitaciones de los asignadores personalizados

Hay algunas restricciones cuando se trabaja con asignadores de memoria personalizados que pueden causar errores de compilación:

* Cualquier artefacto puede ser enlazado con un máximo de un asignador. Los ejecutables, dylibs y staticlibs deben ser enlazados con exactamente un asignador, de no haber sido seleccionado uno de manera explicita el compilador seleccionara uno. Por otro lado rlibs no necesitan enlazar con un asignador (pero igual pueden hacerlo).

* Un consumidor de un asignador se etiqueta con `#![needs_allocator]` (e.j. el crate `liballoc` actualmente) y un crate `#[allocator]` no puede depender transitivamente en un crate que necesita un asignador (e.j las dependencias circulares no estan permitidas). Esto significa básicamente que los asignadores de memoria en la actualidad deben restringirse a libcore.
