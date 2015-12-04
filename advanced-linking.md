% Enlace Avanzado

Los casos comunes de enlace con Rust han sido cubiertos anteriormente en este libro, pero soportar el rango de posibilidades disponibles en lenguajes es importante para Rust para lograr una buena interacción con bibliotecas nativas.

# Argumentos de enlace

Hay otra forma de decirle a `rustc` como personalizar el enlazado, y es via el atributo `link_args`. Este atributo es aplicado a los bloques `extern` y especifica flags planos que necesitan ser pasados a el enlazador cuando se produce un artefacto. Un ejemplo podría ser:

``` no_run
#![feature(link_args)]

#[link_args = "-foo -bar -baz"]
extern {}
# fn main() {}
```

Nota que actualmente esta facilidad esta escondida detrás la puerta `feature(link_args)` debido a que no es una forma sancionada de efectuar enlazado. Actualmente `rutsc` envuelve a el enlazador del sistema (`gcc` en la mayoría de los sistemas, `link.exe` en MSVC), es por ello que tiene sentido proveer argumentos de linea de comando extra, pero no siempre sera este el caso. En el futuro `rustc` pudiera usar LLVM directamente para enlazar con bibliotecas nativas, escenario en donde `link_args` no tendría sentido. Puedes lograr el mismo efecto del atributo `link_args` con el argumento `-C link-args` de `rustc`.

Es altamente recomendado *no* usar este atributo, y en su lugar usar el mas formal `#[link(...)]` en los bloques `extern`.

# Enlazado estatico

El enlace estático se refiere a el proceso de crear una salida que contenga todas las bibliotecas requeridas y por lo tanto no requiera que las bibliotecas estén instaladas en cada sistema en el que desees usar tu proyecto compilado. La dependencias puras Rust son enlazadas estáticamente por defecto de manera que puedas usar las bibliotecas y ejecutables creados sin instalar Rust en todos lados. En contraste, las bibliotecas nativas (e.j  `libc` y `libm`) son usualmente enlazadas de manera dinámica, pero esto se  puede cambiar y enlazarlas también de manera estática.

El enlace es un tópico muy dependiente de plataforma, y el enlace estático puede no ser posible en todas las plataformas. Esta sección asume cierta familiaridad con el enlace en tu plataforma.

## Linux

Por defecto, todos los programas en Linux se enlazaran con la biblioteca `libc` del sistema en conjunto con otro grupo de bibliotecas. Veamos un ejemplo en una maquina linux de 64 bits con GCC y `glibc` (por lejos la `libc` mas común en Linux):

``` text
$ cat example.rs
fn main() {}
$ rustc example.rs
$ ldd example
        linux-vdso.so.1 =>  (0x00007ffd565fd000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007fa81889c000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007fa81867e000)
        librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007fa818475000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007fa81825f000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fa817e9a000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fa818cf9000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007fa817b93000)
```

El enlace dinámico en linux puede ser indeseable si deseas usar las facilidades de la nueva biblioteca en sistemas viejos o sistemas destino que no poseen las dependencias requeridas para ejecutar el programa.

El enlace estático es soportado a través de una `libc` alternativa, [`musl`](http://www.musl-libc.org). Puedes compilar tu version de Rust con `musl` habilitado e instalarlo en un directorio personalizado con las siguientes instrucciones:

```text
$ mkdir musldist
$ PREFIX=$(pwd)/musldist
$
$ # Construir musl
$ curl -O http://www.musl-libc.org/releases/musl-1.1.10.tar.gz
$ tar xf musl-1.1.10.tar.gz
$ cd musl-1.1.10/
musl-1.1.10 $ ./configure --disable-shared --prefix=$PREFIX
musl-1.1.10 $ make
musl-1.1.10 $ make install
musl-1.1.10 $ cd ..
$ du -h musldist/lib/libc.a
2.2M    musldist/lib/libc.a
$
$ # Construir libunwind.a
$ curl -O http://llvm.org/releases/3.7.0/llvm-3.7.0.src.tar.xz
$ tar xf llvm-3.7.0.src.tar.xz
$ cd llvm-3.7.0.src/projects/
llvm-3.7.0.src/projects $ curl http://llvm.org/releases/3.7.0/libunwind-3.7.0.src.tar.xz | tar xJf -
llvm-3.7.0.src/projects $ mv libunwind-3.7.0.src libunwind
llvm-3.7.0.src/projects $ mkdir libunwind/build
llvm-3.7.0.src/projects $ cd libunwind/build
llvm-3.7.0.src/projects/libunwind/build $ cmake -DLLVM_PATH=../../.. -DLIBUNWIND_ENABLE_SHARED=0 ..
llvm-3.7.0.src/projects/libunwind/build $ make
llvm-3.7.0.src/projects/libunwind/build $ cp lib/libunwind.a $PREFIX/lib/
llvm-3.7.0.src/projects/libunwind/build $ cd ../../../../
$ du -h musldist/lib/libunwind.a
164K    musldist/lib/libunwind.a
$
$ # Construir Rust con musl habilitado
$ git clone https://github.com/rust-lang/rust.git muslrust
$ cd muslrust
muslrust $ ./configure --target=x86_64-unknown-linux-musl --musl-root=$PREFIX --prefix=$PREFIX
muslrust $ make
muslrust $ make install
muslrust $ cd ..
$ du -h musldist/bin/rustc
12K     musldist/bin/rustc
```

Ahora posees un Rust con `musl` habilitado! Y debido a que lo hemos instalado en un prefijo personalizado necesitamos asegurarnos de que nuestro sistema pueda encontrar los ejecutables y bibliotecas apropiadas cuando tratemos de ejecutarlo:

```text
$ export PATH=$PREFIX/bin:$PATH
$ export LD_LIBRARY_PATH=$PREFIX/lib:$LD_LIBRARY_PATH
```

Probémoslo!

```text
$ echo 'fn main() { println!("Hola!"); panic!("falla"); }' > example.rs
$ rustc --target=x86_64-unknown-linux-musl example.rs
$ ldd example
        not a dynamic executable
$ ./example
Hola!
thread '<main>' panicked at 'failed', example.rs:1
```

Exito! Este binario puede ser copiado a casi cualquier maquina Linux con la misma arquitectura y ser ejecutado sin ningún problema.

`cargo build` también permite la opción `--target` por medio de la cual puedes construir tus crates normalmente. Sin embargo, podrías necesitar recompilar tus bibliotecas nativas con `musl` antes de poder enlazar con ellas.
