% Interfaz de Funciones Foráneas/Externas

# Introducción

Esta guía usara la biblioteca de compresión/descompresión [snappy](https://github.com/google/snappy) como introducción a la escritura de bindings a código externo. Rust actualmente no puede llamar código en una biblioteca C++ de manera directa, pero snappy incluye una interfaz en C (documentada en
[`snappy-c.h`](https://github.com/google/snappy/blob/master/snappy-c.h)).

El siguiente es un ejemplo minino de como llamar una función foránea que compilara asumiendo que snappy esta instalada:


```no_run
# #![feature(libc)]
extern crate libc;
use libc::size_t;

#[link(name = "snappy")]
extern {
    fn snappy_max_compressed_length(source_length: size_t) -> size_t;
}

fn main() {
    let x = unsafe { snappy_max_compressed_length(100) };
    println!("longitud maxima de un buffer the 100 bytes comprimido: {}", x);
}
```

El bloque `extern` es una lista de firmas de función en una biblioteca foránea, en este caso con la interfaz binaria de aplicación C (ABI en ingles) de la plataforma. El atributo `#[link(...)]` es usado para instruir al enlazador a enlazar con la biblioteca snappy de modo que los símbolos puedan ser resueltos.

Se asume que las interfaces a funciones foráneas son inseguras, es por ello que las llamadas a ellas deben estar dentro de un bloque `unsafe {}` como una promesa al compilador de que todo lo contenido en el es realmente seguro. Bibliotecas en C a algunas veces exponen interfaces que no son thread-safe, y casi cualquier función que toma un apuntador como argumento no es valida para todas las entradas posibles debido a que el apuntador podría ser un apuntador colgante, y los apuntadores planos quedan por fuera del modelo de memoria segura de Rust.

Cuando se declaran los tipos de los argumentos de una función foránea, el compilador de Rust no puede chequear si la declaración es correcta, a consecuencia de esto, especificarla de manera correcta forma parte de mantener el binding correcto en tiempo de ejecución.

El bloque `extern` puede ser extendido para cubrir la API completa de snappy:

```no_run
# #![feature(libc)]
extern crate libc;
use libc::{c_int, size_t};

#[link(name = "snappy")]
extern {
    fn snappy_compress(input: *const u8,
                       input_length: size_t,
                       compressed: *mut u8,
                       compressed_length: *mut size_t) -> c_int;
    fn snappy_uncompress(compressed: *const u8,
                         compressed_length: size_t,
                         uncompressed: *mut u8,
                         uncompressed_length: *mut size_t) -> c_int;
    fn snappy_max_compressed_length(source_length: size_t) -> size_t;
    fn snappy_uncompressed_length(compressed: *const u8,
                                  compressed_length: size_t,
                                  result: *mut size_t) -> c_int;
    fn snappy_validate_compressed_buffer(compressed: *const u8,
                                         compressed_length: size_t) -> c_int;
}
# fn main() {}
```

# Creando una interfaz segura

La interfaz plana en C necesita ser envuelta con el objetivo de proveer seguridad en el manejo de memoria así como el uso de conceptos de alto nivel tales como vectores. Una biblioteca puede escoger entre exponer la interfaz segura de alto nivel y esconder los detalles inseguros internos.

Envolver las funciones que esperan bufers involucra el uso de el modulo `slice::raw` para la manipulación de vectores Rust como apuntadores a memoria. Los vectores de Rust están garantizados a ser un bloque de memoria contiguo. La longitud es el numero de elementos actualmente contenidos, y la capacidad es el tamaño total de la memoria asignada. La longitud es menor o igual a la capacidad.


```rust
# #![feature(libc)]
# extern crate libc;
# use libc::{c_int, size_t};
# unsafe fn snappy_validate_compressed_buffer(_: *const u8, _: size_t) -> c_int { 0 }
# fn main() {}
pub fn validar_buffer_comprimido(src: &[u8]) -> bool {
    unsafe {
        snappy_validate_compressed_buffer(src.as_ptr(), src.len() as size_t) == 0
    }
}
```

La función envoltorio `validar_buffer_comprimido` hace uso de un bloque `unsafe`, pero hace la garantía de que llamarla es segura para todas las entradas a través de la exclusion del `unsafe` de la firma de la función.

Las funciones  `snappy_compress` y `snappy_uncompress` son mas complejas, debido a que un bufer tiene que ser asignado para mantener la salida.

La función `snappy_max_compressed_length` puede ser usada para asignar un vector con la capacidad maxima requerida para almacenar la salida comprimida. El vector entonces puede ser pasado a la función `snappy_compress` como un parámetro de salida. Un parámetro de salida es también pasado para obtener la longitud real después de la compresión para asignar la longitud.


```rust
# #![feature(libc)]
# extern crate libc;
# use libc::{size_t, c_int};
# unsafe fn snappy_compress(a: *const u8, b: size_t, c: *mut u8,
#                           d: *mut size_t) -> c_int { 0 }
# unsafe fn snappy_max_compressed_length(a: size_t) -> size_t { a }
# fn main() {}
pub fn comprimir(orig: &[u8]) -> Vec<u8> {
    unsafe {
        let long_orig = orig.len() as size_t;
        let porig = orig.as_ptr();

        let mut long_dest = snappy_max_compressed_length(long_orig);
        let mut dest = Vec::with_capacity(long_dest as usize);
        let pdest = dest.as_mut_ptr();

        snappy_compress(porig, long_orig, pdest, &mut long_dest);
        dest.set_len(long_dest as usize);
        dest
    }
}
```

La descompresion es similar, debido a que snappy almacena la longitud descomprimida como parte del formato de compresión y `snappy_uncompressed_length` obtendrá el tamaño exacto del bufer requerido.


```rust
# #![feature(libc)]
# extern crate libc;
# use libc::{size_t, c_int};
# unsafe fn snappy_uncompress(compressed: *const u8,
#                             compressed_length: size_t,
#                             uncompressed: *mut u8,
#                             uncompressed_length: *mut size_t) -> c_int { 0 }
# unsafe fn snappy_uncompressed_length(compressed: *const u8,
#                                      compressed_length: size_t,
#                                      result: *mut size_t) -> c_int { 0 }
# fn main() {}
pub fn descomprimir(orig: &[u8]) -> Option<Vec<u8>> {
    unsafe {
        let long_orig = orig.len() as size_t;
        let porig = orig.as_ptr();

        let mut long_dest: size_t = 0;
        snappy_uncompressed_length(porig, long_orig, &mut long_dest);

        let mut dest = Vec::with_capacity(long_dest as usize);
        let pdest = dest.as_mut_ptr();

        if snappy_uncompress(porig, long_orig, pdest, &mut long_dest) == 0 {
            dest.set_len(long_dest as usize);
            Some(dest)
        } else {
            None // SNAPPY_INVALID_INPUT
        }
    }
}
```

Como referencia, los ejemplos usados aquí están disponibles también como una [biblioteca en Github](https://github.com/thestinger/rust-snappy).

# Destructores

La bibliotecas externas usualmente transfieren la pertenencia de los recursos a el código llamador. Cuando esto ocurre, debemos usar los destructores de Rust para proveer seguridad y la garantía de la liberación de dichos recursos (especialmente en el caso de un pánico).

Para mas información acerca de los destructores, echa un vistazo a la sección del [trait Drop](../std/ops/trait.Drop.html).

# Callbacks desde código C a funciones Rust

Algunas bibliotecas externas requieren el uso de callbacks para reportar de vuelta su estado o data parcial a el llamador. Es posible pasar funciones definidas en Rust a una biblioteca externa. El requerimiento para esto es que la función callback este marcada como `extern` y con la convención de llamadas correcta para hacer posible su llamado desde código C.

La función callback puede entonces ser enviada a través de una llamada de registro a la biblioteca en C y su posterior invocación desde allá.

Un ejemplo básico es:

Código Rust:

```no_run
extern fn callback(a: i32) {
    println!("He sido llamado desde C con el valor {0}", a);
}

#[link(name = "extlib")]
extern {
   fn registrar_callback(cb: extern fn(i32)) -> i32;
   fn disparar_callback();
}

fn main() {
    unsafe {
        registrar_callback(callback);
        disparar_callback(); // Dispara el callback
    }
}
```

Código C:

```c
typedef void (*callback_rust)(int32_t);
callback_rust cb;

int32_t registrar_callback(callback_rust callback) {
    cb = callback;
    return 1;
}

void disparar_callback() {
  cb(7); // Llamara a callback(7) en Rust
}
```

En este ejemplo el `main()` de Rust llamara a `disparar_callback()` en C, que a su vez llamara de vuelta a `callback()` en Rust.

## Apuntando callbacks a objetos Rust

El ejemplo anterior demostró como una función global puede ser llamada desde código en C. Si embrago a veces se desea que el callback apunte a un objeto Rust especial. Este podría ser el objeto que representa el envoltorio para el objeto respectivo en C.

Todo esto puede ser logrado a través del paso de un apuntador plano a la biblioteca en C. La biblioteca en C entonces puede incluir el apuntador a el objeto Rust en la notificación. Esto permitirá a el callback acceder de manera insegura el objeto Rust referenciado.

Código Rust:

```no_run
#[repr(C)]
struct ObjetoRust {
    a: i32,
    // otros miembros
}

extern "C" fn callback(objetivo: *mut ObjetoRust, a: i32) {
    println!("He sido llamado desde C con el valor {0}", a);
    unsafe {
        // Actualiza el valor en ObjetoRust con el valor recibido desde el callback
        (*objetivo).a = a;
    }
}

#[link(name = "extlib")]
extern {
   fn registrar_callback(objetivo: *mut ObjetoRust,
                        cb: extern fn(*mut ObjetoRust, i32)) -> i32;
   fn disparar_callback();
}

fn main() {
    // Creando el objeto que sera referenciado en el callback
    let mut objeto_rust = Box::new(ObjetoRust { a: 5 });

    unsafe {
        registrar_callback(&mut *objeto_rust, callback);
        disparar_callback();
    }
}
```

Código C:

```c
typedef void (*callback_rust)(void*, int32_t);
void* objetivo_cb;
callback_rust cb;

int32_t registrar_callback(void* objetivo_callback, callback_rust callback) {
    objetivo_cb = objetivo_callback;
    cb = callback;
    return 1;
}

void disparar_callback() {
  cb(objetivo_cb, 7); // Llamare a callback(&ObjetoRust, 7) in Rust
}
```

## Callbacks Asíncronos

En los ejemplos anteriores los callbacks son invocados como una reacción directa a una llamada a función a la biblioteca externa en C. El control sobre el hilo actual es cambiado de Rust a C para la ejecución del callback, pero al final el callback es ejecutado en el mismo hilo que llamo a la función que disparo el callback.

Las cosas se vuelven mas complicadas cuando la biblioteca externa crea sus propios hilos e invoca los callbacks desde ellos. En estos casos el acceso a las estructuras de datos Rust dentro de los callbacks es especialmente inseguro y mecanismos de sincronización apropiados deben ser usados. Ademas de los mecanismos clásicos de sincronización como mutexes, una posibilidad en Rust es el uso de canales (en `std::sync::mpsc`) para transferir data desde el hilo C que invoco el callback a un hilo Rust.

Si un callback asíncrono tiene como objetivo un objeto especial en el espacio de direcciones de Rust es absolutamente necesario también que ningún otro callback sea ejecutados por la biblioteca en C después de que el respectivo objeto Rust haya sido destruido. Esto puede ser llevado a cabo de-registrando el callback en el destructor del objeto y diseñando la biblioteca de manera tal que garantice que ningún callback sera ejecutado después de la de-registracion.

# Enlace

El atributo `link` en bloques `extern` proporciona el bloque de construcción basico para instruir a rustc como enlazar con bibliotecas nativas. Hoy en dia hay dos formas del atributo link:

* `#[link(name = "foo")]`
* `#[link(name = "foo", kind = "bar")]`

En ambos casos, `foo` es el nombre de la biblioteca nativa con la cual estamos enlazando, y en el segundo caso `bar` es el tipo de biblioteca nativa a la cual esta enlazando el compilador. Actualmente hay tres tipos de bibliotecas nativas conocidos:

* Dinamicas - `#[link(name = "readline")]`
* Estaticas - `#[link(name = "my_build_dependency", kind = "static")]`
* Frameworks - `#[link(name = "CoreFoundation", kind = "framework")]`

Nota que los frameworks solo están disponibles en objetivos OSX.

Los diferentes valores `kind` tienen como objeto diferenciar como la biblioteca nativa participa en el enlace. Desde la perspectiva del enlace, el compilador de Rust crea dos tipos de artefactos: parciales (rlib/staticlib) y finales (dylib/binary). Las dependencias en bibliotecas nativas y frameworks son propagadas a la frontera de artefacto final, mientras que las dependencias estáticas no son propagadas del todo, debido a que las bibliotecas estáticas están integradas directamente dentro del artefacto subsecuente.

Unos pocos ejemplos de como este modelo puede ser usado son:

* Una dependencia de construcción nativa: Algunas veces algún C/C++ de pegamento es necesario cuando se escribe un tipo especifico de código Rust, pero la distribución del código en un formato de biblioteca es simplemente una carga. En este caso, el código sera archivado en un `libfoo.a` y luego el crate rust declarara una dependencia via `#[link(name = "foo", kind = "static")]`.

Independientemente del tipo de salida para el crate, la biblioteca nativa estática sera incluida en la salida, significando que la distribución de la biblioteca estática nativa no es necesaria.

* Una dependencia dinámica normal: Bibliotecas comunes del sistema (como `readline`) están disponibles en un gran numero de sistemas, y a menudo una copia estática de esas biblioteca puede no existir. Cuando esta dependencia es incluida en un crate Rust, objetivos parciales (como rlibs) no enlazaran a la biblioteca, pero cuando el rlib  es incluido en un objetivo final (como un binario), la biblioteca sera enlazada.

En OSX, los frameworks se comportan con la misma semantica que una biblioteca dinámica.

# Bloques Unsafe

Algunas operaciones, como dereferenciar punteros planos o llamadas a funciones que han sido marcadas como inseguras solo son permitidas dentro de bloques unsafe. Los bloques unsafe aíslan la inseguridad y son una promesa al compilador que la inseguridad no se filtrara hacia el exterior del bloque.

Las funciones unsafe, por otro lado, hacen publicidad de la inseguridad a el mundo exterior. Una función insegura se escribe de la siguiente manera:

```rust
unsafe fn kaboom(ptr: *const i32) -> i32 { *ptr }
```

Esta función puede ser invocada solo desde un bloque `unsafe` u otra función `unsafe`.


# Accediendo globales externas

APIs foráneas algunas veces exportan una variable global que podría hacer algo como llevar registro de algun estado global. Con la finalidad de acceder dichas variables, debes declararlas en bloques `extern` con la palabra reservada `static`:

```no_run
# #![feature(libc)]
extern crate libc;

#[link(name = "readline")]
extern {
    static rl_readline_version: libc::c_int;
}

fn main() {
    println!("Tienes la version {} de readline.",
             rl_readline_version as i32);
}
```

Alternativamente, podrías necesitar alterar estado global proporcionado por una interfaz foránea. Para hacer esto, los estáticos pueden ser declarados con `mut` con la finalidad de poder mutarlos.

```no_run
# #![feature(libc)]
extern crate libc;

use std::ffi::CString;
use std::ptr;

#[link(name = "readline")]
extern {
    static mut rl_prompt: *const libc::c_char;
}

fn main() {
    let prompt = CString::new("[my-awesome-shell] $").unwrap();
    unsafe {
        rl_prompt = prompt.as_ptr();

        println!("{:?}", rl_prompt);

        rl_prompt = ptr::null();
    }
}
```

Nota que toda la interacción con un `static mut` es insegura, ambos lectura y escritura. Lidiar con estado global mutable requiere de un gran cuidado.

# Convenciones de llamadas foráneas

La mayoría de el código foráneo expone un ABI C, y Rust usa por defecto la convención de llamadas de la plataforma al momento de llamar funciones externas. Algunas funciones foráneas, notablemente el API de Windows, usan otra convenión de llamadas. Rust provee una forma de informar al compilador acerca de cual convención debe ser usada:

```rust
# #![feature(libc)]
extern crate libc;

#[cfg(all(target_os = "win32", target_arch = "x86"))]
#[link(name = "kernel32")]
#[allow(non_snake_case)]
extern "stdcall" {
    fn SetEnvironmentVariableA(n: *const u8, v: *const u8) -> libc::c_int;
}
# fn main() { }
```

Esto aplica a el bloque `extern` completo. La lista de restricciones ABI soportadas son:

* `stdcall`
* `aapcs`
* `cdecl`
* `fastcall`
* `Rust`
* `rust-intrinsic`
* `system`
* `C`
* `win64`

La mayoría de las abis son auto-explicativas, pero el abi `system` puede parecer un poco raro. Esta restricción selecciona cualquiera que sea el ABI apropiado para interoperar con las bibliotecas objetivo. Por ejemplo, en win32 con una arquitectura x86, significa que el abi usado sera `stdcall`. En x86_64, sin embargo, Windows usa la convención de llamadas `C`, entonces se usa `C`. Esto se traduce a que en nuestro ejemplo anterior, pudimos haber hecho uso de `extern "system" { ... }` para definir un bloque para todos los sistemas Windows, no solo los x86.

# Interoperabilidad con código externo

Rust garantiza que la distribución de un `struct` sea compatible con la representación de la plataforma en C solo si el atributo `#[repr(C)]` es aplicado. `#[repr(C, packed)]` puede ser usado para distribuir los miembros sin padding. `#[repr(C)]` puede ser aplicado también a un enum.

Los boxes Rust (`Box<T>`) usan apuntadores no-nulables como handles que apuntan a el objeto contenido. Sin embargo, no deben ser creados manualmente debido que son manejados por asignadores internos. La referencias pueden ser asumidas de manera segura como apuntadores no-nulables directos al tipo. Sin embargo, romper el chequeo de préstamo (borrowing) o las reglas de mutabilidad no se garantiza ser seguro, es por ello que se prefiere el uso de apuntadores planos (`*`) de ser necesario debido a que el compilador no puede asumir muchas cosas acerca de ellos.

Los vectores y las cadenas de caracteres comparten la misma distribución en memoria, y utilidades para interactuar con APIs C están disponibles en los módulos `vec` y `str`. Sin embargo, las cadenas de caracteres no son terminadas en `\0`. Si necesitas una cadena de caracteres que termine en NUL para interactuar con C, debes entonces hacer uso de el tipo `CString` en el modulo `std::ffi`.

La biblioteca estándar incluye alias de tipo y definiciones de funciones para la biblioteca estándar de C en el modulo `libc`, y Rust enlaza por defecto con `libc` y `libm`.

# La "optimizacion de apuntador nulable"

Ciertos tipos están definidos para no ser `null`. Esto incluye referencias (`&T`,`&mut T`), boxes (`Box<T>`), y apuntadores a funciones (`extern "abi" fn()`). Cuando haces interfaz con C, apuntadores que pueden ser null son usados algunas veces. Como un caso especial, un `enum` genérico que contiene exactamente dos variantes, de las cuales una no contiene data y la otra contiene un solo campo, es elegible para la "optimizacion de apuntador nulable". Cuando dicha enum es instanciada con uno de los tipos no-nulables, es representada como un solo apuntador, y la variante sin data es representada como el apuntador null. Entonces `Option<extern "C" fn(c_int) -> c_int>` es como uno representa un apuntador a función nullable usando el ABI de C.

# Llamando coding Rust desde C

Podrías querer compilar código Rust de modo que permita ser llamado desde C. Esto es fácil, pero requiere ciertas cosas:

```rust
#[no_mangle]
pub extern fn hola_rust() -> *const u8 {
    "Hola, mundo!\0".as_ptr()
}
# fn main() {}
```

El `extern` hace que esta función se adhiera a la convención de llamadas de C, tal y como se discute en "[Convenciones de llamadas foráneas](ffi.html#convenciones-de-llamadas-foráneas)". El atributo `no_mangle` desactiva el estropeo (mangling) de nombres de Rust, de manera tal que sea mas facil de enlazar.


# FFI y pánicos

Es importante estar consciente de los `panic!`os cuando se trabaja con FFI. Un `panic!` en al limite de FFI es comportamiento indefinido. Si estas escribiendo código que pueda hacer pánico, debes ejecutarlo en otro hilo, de esta forma el pánico no se propagara hasta C:

```rust
use std::thread;

#[no_mangle]
pub extern fn oh_no() -> i32 {
    let h = thread::spawn(|| {
        panic!("Oops!");
    });

    match h.join() {
        Ok(_) => 1,
        Err(_) => 0,
    }
}
# fn main() {}
```
