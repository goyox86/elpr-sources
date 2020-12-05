% ¡Hola, Cargo!

[Cargo][cratesio] es una herramienta que los Rusteros usan como ayuda para administrar sus proyectos Rust. Cargo esta 
actualmente en estado pre-1.0, y debido a ello es todavía un trabajo en proceso. Sin embargo ya es lo suficientemente 
bueno para muchos proyectos Rust y se asume que los proyectos Rust usarán Cargo desde el principio.

[cratesio]: http://doc.crates.io

Cargo administra tres cosas: la compilación de tu código, descarga de las dependencias que tu código necesita y la 
compilación de dichas dependencias. En primera instancia tu código no tendrá ninguna dependencia, es por ello que 
estaremos usando solo la primera parte de la funcionalidad de Cargo. Eventualmente, agregaremos más. Debido a que 
comenzamos usando Cargo desde el principio será fácil agregar después.

Si has instalado Rust a través de los instaladores oficiales entonces deberías tener Cargo. Si has instalado Rust de 
alguna otra manera, podrías echar un vistazo al [README de Cargo][cargoreadme] para instrucciones específicas acerca de 
cómo instalarlo.

[cargoreadme]: https://github.com/rust-lang/cargo#installing-cargo-from-nightlies

## Migrando a Cargo

Convirtamos Hola Mundo a Cargo.

Para Carguificar nuestro proyecto, necesitamos dos cosas: Crear un archivo de configuración `Cargo.toml` y colocar 
nuestro archivo de código fuente en el lugar correcto. Hagamos esa parte primero:


```bash
$ mkdir src
$ mv main.rs src/main.rs
```

Nota que debido a que estamos creando un ejecutable, usamos `main.rs`. Si quisiéramos crear una biblioteca, deberíamos 
usar `lib.rs`. Ubicaciones personalizadas para el punto de entrada pueden ser especificadas con una clave [`[[lib]]` or 
`[[bin]]`][crates-custom] en el archivo TOML descrito a continuación.

[crates-custom]: http://doc.crates.io/manifest.html#configuring-a-target

Cargo espera que tus archivos de código fuente residan en el directorio `src`. Esto deja el nivel raíz para otras cosas, 
como READMEs, información de licencias y todo aquello no relacionado con tu código. Cargo nos ayuda a mantener nuestros 
proyectos agradables y ordenados. Un lugar para todo y todo en su lugar.

A continuación, nuestro archivo de configuración:

```bash
$ editor Cargo.toml
```
Asegúrate de tener éste nombre correcto: ¡necesitas la `C` mayúscula!

Coloca esto dentro:

```toml
[package]

name = "hola_mundo"
version = "0.0.1"
authors = [ "Tu nombre <tu@ejemplo.com>" ]
```

Este archivo esta en formato [TOML][toml]. Dejemos que sea él mismo quien se explique:

> El objetivo de TOML es ser un formato de configuración mínimo fácil de leer debido a
> una semántica obvia. TOML está diseñado para mapear de forma in-ambigua a una tabla hash.
> TOML debería ser fácil de convertir en estructuras de datos en una amplia variedad de
> lenguajes.

TOML es muy similar a INI, pero con algunas bondades extra.

[toml]: https://github.com/toml-lang/toml

Una vez que tengas este archivo es su lugar, ¡deberíamos estar listos para compilar! Prueba esto:

```bash
$ cargo build
   Compiling hola_mundo v0.0.1 (file:///home/tunombre/proyectos/hola_mundo)
$ ./target/debug/hola_mundo
¡Hola, mundo!
```

¡Bam! Construimos nuestro proyecto con `cargo build` y lo ejecutamos con
`./target/debug/hola_mundo`. Podemos hacer los dos pasos en uno con `cargo run`:

```bash
$ cargo run
     Running `target/debug/hola_mundo`
¡Hola, mundo!
```

Nótese que no reconstruimos el proyecto esta vez. Cargo determinó que no habíamos cambiado el archivo de código fuente, 
así que simplemente ejecuto el binario. Si hubiéremos realizado una modificación, deberíamos haberlo visto haciendo los dos pasos:


```bash
$ cargo run
   Compiling hola_mundo v0.0.1 (file:///home/tunombre/proyectos/hola_mundo)
     Running `target/debug/hola_mundo`
¡Hola, mundo!
```

Esto no nos ha aportado mucho por encima de nuestro simple uso de `rustc`, pero piensa en el futuro: cuando nuestros 
proyectos se hagan más complicados, necesitaremos hacer más cosas para lograr que todas las partes compilen correctamente. 
Con Cargo, a medida que nuestro proyecto crece, simplemente ejecutamos `cargo build` y todo funcionará de forma correcta.

Cuando nuestro proyecto esta finalmente listo para la publicación, puedes usar `cargo build --release` para compilarlo con 
optimizaciones.

También habrás notado que Cargo ha creado un archivo nuevo: `Cargo.lock`.

```toml
[root]
name = "hola_mundo"
version = "0.0.1"
```

Este archivo es usado por Cargo para llevar el control de las dependencias usadas en tu aplicación. Por ahora, no tenemos 
ninguna y está un poco disperso. Nunca deberías necesitar tocar este archivo por tu cuenta, solo deja a Cargo manejarlo.

¡Eso es todo! Hemos construido satisfactoriamente `hola_mundo` con Cargo. Aunque nuestro programa sea simple, está usando 
gran parte de las herramientas reales que usarás por el resto de tu carrera con Rust. Puedes asumir que para comenzar 
virtualmente con todo proyecto Rust harás lo siguiente:

```bash
$ git clone algunaurl.com/foo
$ cd foo
$ cargo build
```

## Un Proyecto Nuevo

¡No tienes que pasar por todo ese proceso completo cada vez que quieras comenzar un proyecto nuevo! Cargo posee la 
habilidad de crear un directorio plantilla en el cual puedes comenzar a desarrollar inmediatamente.

Para comenzar un proyecto nuevo con Cargo, usamos `cargo new`:

```bash
$ cargo new hola_mundo --bin
```

Estamos pasando `--bin` porque estamos creando un programa binario: si estuviéramos creando una biblioteca, lo omitiríamos.

Echemos un vistazo a lo que Cargo ha generado para nosotros:

```bash
$ cd hola_mundo
$ tree .
.
├── Cargo.toml
└── src
    └── main.rs

1 directory, 2 files
```

Si no tienes el comando `tree` instalado, probablemente podrías obtenerlo mediante el manejador de paquetes de tu 
distribución. No es necesario, pero es ciertamente útil.

Esto es todo lo que necesitas para comenzar. Primero veamos nuestro `Cargo.toml`:

```toml
[package]

name = "hola_mundo"
version = "0.0.1"
authors = ["Tu Nombre <tu@ejemplo.com>"]
```

Cargo ha rellenado este archivo con valores por defecto basado en los argumentos que le proporcionaste y tu configuración 
global `git`. Podrás notar también que Cargo ha inicializado el directorio `hola_mundo` como un repositorio `git`.

Aquí esta el contenido de `src/main.rs`:

```rust
fn main() {
    println!("¡Hola, mundo!");
}
```

Cargo ha generado un "¡Hola, mundo!" para nosotros, ya estas listo para empezar a codear. Cargo posee su propia 
[guia][guide] la cual cubre todas sus características con mucha más profundidad.

[guide]: http://doc.crates.io/guide.html

Ahora que hemos aprendido las herramientas, comencemos en realidad a aprender más de Rust como lenguaje. Esto es la 
báse que te servirá bien por el resto de tu tiempo con Rust.

Tienes dos opciones: Sumergirte en un proyecto con ‘[Aprende Rust][learnrust]’ o comenzar desde el fondo y trabajar 
hacia arriba con ‘[Sintaxis y Semántica][syntax]’. Programadores de sistemas más experimentados probablemente preferirán 
‘Aprende Rust’, mientras que aquellos provenientes de lenguajes dinámicos podrian tambien disfrutarlo. ¡Gente diferente 
aprende diferente! Escoje lo que funcione mejor para ti.

[learnrust]: learn-rust.html
[syntax]: syntax-and-semantics.html
