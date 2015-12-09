% No stdlib

Por defecto, `std` es enlazada a cada crate Rust. En algunos contextos, esto es indeseable, y puede ser evitado agregando el attributo `#![no_std]` a el crate.

Obviamente hay mas vida que solo bibliotecas: uno podria usar `#[no_std]` co un ejecutable, controlar el punto de entrada es posible de dos maneras: el atributo `#[start]`, o sobrescribiendo el shim por defecto para la funcion `main` en C con el tuyo.

Los argumentos de linea de comandos son pasados a la funcion marcada como `#[start]`  en el mismo formato de C:

```rust
# #![feature(libc)]
#![feature(lang_items)]
#![feature(start)]
#![no_std]

// Halando la biblioteca libc del sistema para lo que crt0.o potencialmente requiere
extern crate libc;

// Punto de entrada para este programa
#[start]
fn start(_argc: isize, _argv: *const *const u8) -> isize {
    0
}

// Estas funciones y traits son usados por el compilador, pero
// no para un hola-mundo plano. Estas son normalmente proporcionadas
// por libstd.
#[lang = "eh_personality"] extern fn eh_personality() {}
#[lang = "panic_fmt"] fn panic_fmt() -> ! { loop {} }
# #[lang = "eh_unwind_resume"] extern fn rust_eh_unwind_resume() {}
# #[no_mangle] pub extern fn rust_eh_register_frames () {}
# #[no_mangle] pub extern fn rust_eh_unregister_frames () {}
# // fn main() {} tricked you, rustdoc!
```

Para sobreescribir el shim `main` insertado por el compilador, debemos desabilitarlo con el atributo `#![no_main]` y luego crear el simbolo apropiado con el ABI correcto y el nombre correcto, lo cual requiere tambien sobreescribir el mangling de nombres del compilador:

```rust
# #![feature(libc)]
#![feature(lang_items)]
#![feature(start)]
#![no_std]
#![no_main]

extern crate libc;

#[no_mangle] // asegurate de que este simbolo se llame `main` en la salida tambien
pub extern fn main(argc: i32, argv: *const *const u8) -> i32 {
    0
}

#[lang = "eh_personality"] extern fn eh_personality() {}
#[lang = "panic_fmt"] fn panic_fmt() -> ! { loop {} }
# #[lang = "eh_unwind_resume"] extern fn rust_eh_unwind_resume() {}
# #[no_mangle] pub extern fn rust_eh_register_frames () {}
# #[no_mangle] pub extern fn rust_eh_unregister_frames () {}
# // fn main() {} te tengo, rustdoc!
```

El compilador actualmente hace unas pocas asunciones acerca de los simbolos que estan disponibles para llamar en el ejecutable . Normalmente estas funciones son proporcionadas por la biblioteca estandar, per sin ellas debes definir las tuyas.

La primera de esas funciones, `eh_personality`, es usada por el mecanismo de fallas del compilador. Esta es comunmente mapedada a la funcion de personalidad de GCC (echa un vistazo a [implementacion en libstd](../std/rt/unwind/index.html) para mas informacion), pero los crates que no disparan panicos pueden tener seguro que esta funcion nunca sera llamada. La segunda funcion, `panic_fmt`, es tambien usada por los mecanismo de falla del compilador.

## Usando libcore

> **Note**: la estructura de la biblioteca core es inestable, y es recomendado
> usar la biblioteca estandar donde sea posible.

Con las tecnicas anteriores, obtuvimos un ejecutable plano corriendo algo de codigo Rust. Sin embargo, hay una buena cantidad de funcionalidad proporcionada por la biblioteca estandar, que es necesaria para ser productivo en Rust. Si la biblioteca estandar no es suficiente, entonces [libcore](../core/index.html) esta disenada para ser usada en su lugar.

La biblioteca core posee pocas dependencias y es mucho mas portable que la bibliotecaq estandar. Adicionalmente, la biblioteca core posee la mayoria de la funcionlidad para escribir codigo Rust idiomatico y efectivo. Cuando se usa `#![no_std]`, Rust automaticamente inyectara el crate `core`, jsuto como hacemos con `std` cuando la estamos usando.

Como un ejemplo, he aqui un programa que calculara el producto escalar de dos vectores provenientes de C, usando paracticas idiomatic Rust.

```rust
# #![feature(libc)]
#![feature(lang_items)]
#![feature(start)]
#![feature(raw)]
#![no_std]

extern crate libc;

use core::mem;

#[no_mangle]
pub extern fn producto_escalar(a: *const u32, a_len: u32,
                               b: *const u32, b_len: u32) -> u32 {
    use core::raw::Slice;

    // Convirtiendo los arrays proporcionados en slices Rust.
    // El modulo core::raw garantiza que la estructura Slice
    // posea la misma representacion en memoria que un slice
    // &[T].
    //
    // Lo siguiente es una operacion unsafe puesto a que el
    // compilador no puede determinados que los apuntadores
    // son validos.
    let (a_slice, b_slice): (&[u32], &[u32]) = unsafe {
        mem::transmute((
            Slice { data: a, len: a_len as usize },
            Slice { data: b, len: b_len as usize },
        ))
    };

    // Iterando por sobre los slices, recolectando el resultado
    let mut ret = 0;
    for (i, j) in a_slice.iter().zip(b_slice.iter()) {
        ret += (*i) * (*j);
    }
    return ret;
}

#[lang = "panic_fmt"]
extern fn panic_fmt(args: &core::fmt::Arguments,
                    file: &str,
                    line: u32) -> ! {
    loop {}
}

#[lang = "eh_personality"] extern fn eh_personality() {}
# #[start] fn start(argc: isize, argv: *const *const u8) -> isize { 0 }
# #[lang = "eh_unwind_resume"] extern fn rust_eh_unwind_resume() {}
# #[no_mangle] pub extern fn rust_eh_register_frames () {}
# #[no_mangle] pub extern fn rust_eh_unregister_frames () {}
# fn main() {}
```

Notese que hay un item extra aqui que difiere de los ejemplos anteriores, `panic_fmt`. La cual debe ser definida por los consumidores de libcore debido a que libcore declara panicos, pero no los define. El item de lenguaje `panic_fmt` es la definicion de panico de este crate, y debe garantizar que nunca retorna.

Como se puede observar en este ejemplo, la biblioteca core tiene como objetivo proporcionar el poder de Rust en todas las circunstancias, sin importar los requerimientos de plataforma. Otras bibliotecas, como liballoc, agregan funcionalidad a libcore la cual hace otras asunciones especificas de la plataforma, pero continua siendo mas portable que la biblioteca estandar en si misma,
