% Enlace Avanzado

Los casos comunes de enlace con Rust han sido cubiertos anteriormente en este libro, pero soportar el rango de posibilidades hechas disponibles por otros lenguajes es importante para Rust para lograr una buena interaccion con bibliotecas nativas.

# Argumentos de enlace

Hay otra forma de decirle a `rustc` como personalizar el enlazado, y es via el atributo `link_args`. Este atributo es aplicado a los bloques `extern` y especifica flags planos que necesitan ser pasados a el enlazador cuando se produce un artefacto. Un ejemplo podria ser:

``` no_run
#![feature(link_args)]

#[link_args = "-foo -bar -baz"]
extern {}
# fn main() {}
```

Nota que actualmente esta facilidad esta escondida detras la puerta `feature(link_args)` debido a que no es una forma sancionada de efectuar enlazado. Actualmente `rutsc` envuelve a el enlazador del sistema (`gcc` en la mayoria de los sitemas, `link.exe` en MSVC), es por ello que tiene sentido proveer argumentos de linea de comando extra, pero no siempre sera este el caso. En el futuro `rustc` pudiera usar LLVM directamente para enlazar con bibliotecas nativas, y en donde `link_args` no tendria sentido. Puedes lograr el mismo efecto del atributo `link_args` con el argumento `-C link-args` de `rustc`.

Es latamente recomendado *no* usar este atributo, y en su lugar usar el mas formal `#[link(...)]` en bloques `extern`.

# Enlazado estatico

El enlace estatico se refiere a el proceso de crear una salida que contenga todas las bibliotecas requeridas y por lo tanto no requiera que las bibliotecas esten presentes en cada sistema que desees usar tu proyecto compilado. Dependencias puras Rust son enlazadas estaticamente por defecto de manera que puedas usar las bibliotecas y ejecutables creados sin instalar Rust en todos lados. En contraste, las bibliotecas nativas (e.j  `libc` y `libm`) son usualmente enlazadas de manera dinamica, pero es posible cambiar esto y enlazarlas tambien de manera estatica.

El enlace es un topico muy dependiente de plataforma, y el enlazado estatico puede no ser posible en todas las plataformas. Esta seccion asume alguna familiaridad con el enlace en tu plataforma.

## Linux

Por defecto, todos los programas en Linux se enlazaran con la biblioeca `libc` del sistema en conjunto con otro grupo de bibliotecas. Veamos un ejemplo en una maquina linux de 64 bits con GCC y `glibc` (por lejos la `libc` mas comun en Linux):

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

Enlace dinamico en linux puede ser indeseable si deseas usar las facilidades de la nueva biblioteca en sistemas viejos o sistemas destino que no poseen las dependencias requeridas para ejecutar el programa.

El enlace estatico es soportado a traves de una `libc` alternativa, [`musl`](http://www.musl-libc.org). Puedes compilar tu version de Rust con `musl` habilitado e instalarlo en un directorio personalizado con las siguientes instrucciones:

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

Ahora posees un Rust con `musl` habilitado! Y debido a que lo hemos instalado en un prefijo personalizado necesitamos asegurarnos de que nuestro sistema poueda encontarr los ejecutables y bibliotecas apropiadas cuando tratamos de ejecutarlo:

```text
$ export PATH=$PREFIX/bin:$PATH
$ export LD_LIBRARY_PATH=$PREFIX/lib:$LD_LIBRARY_PATH
```

Probemoslo!

```text
$ echo 'fn main() { println!("Hola!"); panic!("falla"); }' > example.rs
$ rustc --target=x86_64-unknown-linux-musl example.rs
$ ldd example
        not a dynamic executable
$ ./example
hi!
thread '<main>' panicked at 'failed', example.rs:1
```

Exito! Este binario puede ser copiado a casi cualquier maquina Linux con la misma arquitectura y ser ejecutado sin ningun problema.

`cargo build` tambien permite la opcion `--target` por medio de la cual deberias puedes construir tus crates normalmente. Sin embargo,  puedes necesitar recompilar tus bibliotecas nativas con `musl` antes de poder enlazar con ellas.
