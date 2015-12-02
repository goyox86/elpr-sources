% Crates y Modulos

Cuando un proyecto comienza a crecer, se considera una buena practica de ingenieria de software dividirlo en un grupo de componentes mas pequenos, uniendolos luego. Tamnieb es importante tener una interfaz bien definida, de manera que una parte de la funcionalidad sea privada, y otra publica. Para facilitar esto, Rust posee un sitema de modulos.

# Terminologia basica: Crates y Modulos

Rust posee dos terminos distintos que se relacionan con el sistema de modulos: ‘crate’ y ‘modulo’. Un crate es un sinonimo para ‘bliblioteca’ or ‘paquete’ en otros lenguajes. De ali el nombre “Cargo” del manejador de paquetes de Rust: haces disponibles tus crates a los demas con Cargo.  Los crates pueden producir un ejecutables o una biblioteca, dependiendo del proyecto.

Cada crate posee un *modulo raiz* implicito que contiene el codigo para dicho crate. Puedes definir un arbol de submodiulos bajo el emodulo raiz. Los modulos te permiten particionar tu codigo dentro del crate en si mismo.

Como ejemplo, creemos un crate *frases*, que nos proveera varias frases en diferentes idiomas. Para mantener las cosas simples, nos apegaremos a ‘saludos’ y ‘despedidas’ como los dos tipos de frases, asi como Ingles y Japones (日本語) como los dos idiomas en los que dichas frases estaran representados. Usaremos la siguiente distribucion de modulos:


```text
                                      +-----------+
                                  +---|  saludos  |
                                  |   +-----------+
                   +----------+   |
               +---|  ingles  |---+
               |   +----------+   |   +-----------+
               |                  +---| despedidas |
+----------+   |                      +-----------+
|  frases  |---+
+----------+   |                      +-----------+
               |                  +---|  saludos  |
               |   +-----------+  |   +-----------+
               +---|  japones  |--+
                   +-----------+  |
                                  |   +------------+
                                  +---| despedidas |
                                      +------------+
```

En este ejemplo, `frases` es el nombre de nuestro crate. El resto son modulos. Puedes observar que los modulos forman un arbol, partiendo desde la *raiz*, la cual es la raiz del arbol `frases` en si.

Ahora que tenemos un plan, definamos estos modulos en codigo. Para comenzar, generemos un proyecto con Cargo:

```bash
$ cargo new frases
$ cd frases
```

Si recuerdas, lo anterior genera un proyecto simple:

```bash
$ tree .
.
├── Cargo.toml
└── src
    └── lib.rs

1 directory, 2 files
```

`src/lib.rs` es la raiz de nuestro crate, correspondiendo con `frases` en nuestro diagrama anterior.
above.

# Definiendo Modulos

Para definir cada uno de nuestros modulos, usamos la palabra reservada `mod`. Hagamos que nuestro `src/lib.rs` se vea asi:

```rust
mod ingles {
    mod saludos {
    }

    mod despedidas {
    }
}

mod japones {
    mod saludos {
    }

    mod despedidas {
    }
}
```

Despues de la palabra reservada `mod` debes proporcionar el nombre de el modulo. Los nombrede de modulo siguen la smismas convenciones que los demas identificadores en Rust: `snake_case_en_minusculas`. El contenido de cada modulos esta delimitado por llaves (`{}`).

Dentro de un determinado `mod`, puedes declarar sub-`mod`s. Podemos referirnos a los submodulos con una notacion de unos dos puntos dobles (`::`): nuestros cuatrio modulos anidados son `ingles::saludos`, `ingles::despedidas`, `japones::saludos`, and `japones::despedidas`. Debido a que estos sub-modulos estan dentro del espacio de nombres del modulo padre, los nombres no entran en conflicto: `ingles::saludos` y `japones::saludos` son distintos incluso cuando sus nombres son ambos `saludos`.

Debido a que este crate no posee una funcion `main()`, y se llama `lib.rs`, Cargo construira este crate como una biblioteca:

```bash
$ cargo build
   Compiling frases v0.0.1 (file:///home/tu/proyectos/frases)
$ ls target/debug
build  deps  examples  libfrases-a7448e02a0468eaa.rlib  native
```

`libfrases-hash.rlib` es el crate compilado. Antes de ver como usar este crate desde otro, dividamoslo en multiples archivos.

# Crates con multiple archivos

Si cada crate fuera un solo archivo, dichos archivos serian bastante grandes. Es mas facil dividir los crates en multiples archivos, y Rust soporta esto de dos maneras.

En lugar de declarar un modulo de la siguiente manera:

```rust,ignore
mod ingles {
    // el contenido del modulo va aqui
}
```

Podemos declarar nuestro modulo asi:

```rust,ignore
mod ingles;
```

Si hacemos eso, Rust esperara encontrar bien un archivo `ingles.rs` o un archivo `ingles/mod.rs` con el contenido de nuestro modulo.

Nota que todos en estos archivos , no necesitas re-declarar el modulo: ya esto ha sido efectuado con la declaracion incial `mod`.

Usando estas dos tecnicas, podemos partir nuestro crte en dos directorios y siete archivos:

```bash
$ tree .
.
├── Cargo.lock
├── Cargo.toml
├── src
│   ├── ingles
│   │   ├── despedidas.rs
│   │   ├── saludos.rs
│   │   └── mod.rs
│   ├── japanese
│   │   ├── despedidas.rs
│   │   ├── saludos.rs
│   │   └── mod.rs
│   └── lib.rs
└── target
    └── debug
        ├── build
        ├── deps
        ├── examples
        ├── libfrases-a7448e02a0468eaa.rlib
        └── native
```

`src/lib.rs` es la raiz de nuestro crate, y luce asi:

```rust,ignore
mod ingles;
mod japones;
```

Estas dos declaraciones le dicen a Rust que busque bien sea `src/ingles.rs` y `src/japones.rs`, o `src/ingles/mod.rs` y `src/japones/mod.rs`, dependiendo de nuestra preferencia. En este caso, debido a que nuestros modulos poseen submodulos hemos seleccionado la segunda. Ambos `src/ingles/mod.rs` y `src/japones/mod.rs`se ven asi:

```rust,ignore
mod saludos;
mod despedidas;
```

De nuevo, estas declaraciones hacen que Rust busque bien sea por `src/ingles/saludos.rs` and `src/japones/saludos.rs` o `src/ingles/despedidas/mod.rs` and `src/japones/despedidas/mod.rs`. Debido a que los submodulos no poseen sus propios submodulos, hemos optado por usar el enfoque `src/ingles/saludos.rs` and `src/japones/despedidas.rs`

Again, these declarations tell Rust to look for either
`src/english/greetings.rs` and `src/japanese/greetings.rs` or
`src/english/farewells/mod.rs` and `src/japanese/farewells/mod.rs`. Because
these sub-modules don’t have their own sub-modules, we’ve chosen to make them
`src/english/greetings.rs` and `src/japanese/farewells.rs`. Whew!

Ambos `src/ingles/saludos.rs` y `src/japones/despedidas.rs` estan vacios por el momento. Agreguemos algunas funciones.

Coloca esto en `src/ingles/saludos.rs`:

```rust
fn hola() -> String {
    "Hello!".to_string()
}
```

Put this in `src/ingles/despedidas.rs`:

```rust
fn adios() -> String {
    "Goodbye.".to_string()
}
```

Put this in `src/japones/saludos.rs`:

```rust
fn hello() -> String {
    "こんにちは".to_string()
}
```

Por supuesto, puedes copiar y pegar esto desde esta pagina web, o simplemente escribe alguna otra cosa. No es importante que en efecto coloques ‘konnichiwa’ para aprender acerca del sistema de modulos.

Coloca lo siguiente en `src/japones/despedidas.rs`:

```rust
fn adios() -> String {
    "さようなら".to_string()
}
```

(‘Sayōnara’, por si eres curioso.)

Ahora que tenemos algo de funcionalidad en nuestro crate, intentemos usarlos desde otro crate.

# Importing External Crates

Ya tenemos un carte biblioteca. Creemos un crate ejecutable que importe y use nuestra biblioteca.

Crea un archivo `src/main.rs` y coloca esto en el (no compilara todavia):

```rust,ignore
extern crate frases;

fn main() {
    println!("Hola en Ingles: {}", frases::ingles::saludos::hola());
    println!("Adios en Ingles: {}", frases::ingles::despedidas::adios());

    println!("Hola en Japones: {}", frases::japones::saludos::hola());
    println!("Adios en Japones: {}", frases::japones::despedidas::adios());
}
```

La declaracion `extern crate` le informa a Rust que necesitamos compilar y enlazar al crate `frases`. Podemos entonces usar el modulo `frases` aca/ Como mensionamos anteriromente, puedes hacer uso de dos puntos dobles para hacer referencia a los sub-modulos y las funciones dentro de ellos.

Nota: cuando se importa un crate que tiene guiones en su nombre "como-este", lo cual no es un identificador valido en Rust, sera convertido cambiandole los guiones a guiones bajos, asi que escribirias algo como `extern crate como_este;`

Tambien, Cargo assume que `src/main.rs` es la raiz de un crate binario, en lugar de un crate biblioteca. Nuestro paquete ahora tiene dos crate: `src/lib.rs` y `src/main.rs`. Este patronh es muy comun en crates ejecutables: la mayoria de la funcionalidad esta en un crate biblioteca, y el crate ejecutable usa dicha biblioteca. De esta forma, otros programas pueden tambien hacer uso de la biblioteca, y tambien es una buena serparacion de responsabilidades.

Lo anterior todavia no funciona. Obtenemos cuatro errores que lucen similares a estos:

```bash
$ cargo build
   Compiling frases v0.0.1 (file:///home/tu/proyectos/frases)
src/main.rs:4:38: 4:72 error: function `hola` is private
src/main.rs:4     println!("Hola en Ingles: {}", frases::ingles::saludos::hola());
                                                 ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
note: in expansion of format_args!
<std macros>:2:25: 2:58 note: expansion site
<std macros>:1:1: 2:62 note: in expansion of print!
<std macros>:3:1: 3:54 note: expansion site
<std macros>:1:1: 3:58 note: in expansion of println!
frases/src/main.rs:4:5: 4:76 note: expansion site
```

Pro defecto todo es privado en Rust. Hablemos de esto con mayor detale.

# Exportando una Interfaz Publica

Rust te permite controlar de maner precisa cuales aspectos de tu imterfaz son publicos, y por defecto private es el defacto. OPara hacer algo publico, debes hacer uso de la palabra reservada `pub`. Enfoquemonos en el moulo `ingles` primero, para ello, reduzcamos nuestro `src/main.rs` a:

```rust,ignore
extern crate frases;

fn main() {
    println!("Hola en Ingles: {}", frases::ingles::saludos::hola());
    println!("Adios en Ingles: {}", frases::ingles::despedidas::adios());
}
```

En nuestro `src/lib.rs`, agreguemos `pub` a la declaracion del mopdulo `ingles`:

```rust,ignore
pub mod ingles;
mod japones;
```

Y en nuestro `src/english/mod.rs`, hagamos a ambos `pub`:

```rust,ignore
pub mod saludos;
pub mod despedidas;
```

En nuestro `src/ingles/saludos.rs`, agreguemos `pub` a nuestra declaracion `fn`:

```rust,ignore
pub fn hola() -> String {
    "Hola!".to_string()
}
```

Y tambien en `src/ingles/despedidas.rs`:

```rust,ignore
pub fn adios() -> String {
    "Adios.".to_string()
}
```

Ahora, nuestro crate compila, con unos pocos warinigs acerca de no haber usado las funciones en `japones`:

```bash
$ cargo run
   Compiling frases v0.0.1 (file:///home/tu/proyectos/frases)
src/japones/saludos.rs:1:1: 3:2 warning: function is never used: `hola`, #[warn(dead_code)] on by default
src/japones/saludos.rs:1 fn hola() -> String {
src/japones/saludos.rs:2     "こんにちは".to_string()
src/japones/saludos.rs:3 }
src/japones/despedidas.rs:1:1: 3:2 warning: function is never used: `adios`, #[warn(dead_code)] on by default
src/japones/despedidas.rs:1 fn adios() -> String {
src/japones/despedidas.rs:2     "さようなら".to_string()
src/japones/despedidas.rs:3 }
     Running `target/debug/frases`
Hola en Ingles: Hello!
Adios en Ingles: Goodbye.
```

`pub` tambien aplica a las `struct`s y sus cmapos miembro. Y Rust teniendo siempre su tendencia hacia la seguridad, el hacer una `struct` public no hara a sus miembros automaticamente public: debes marcarlos individualmente como `pub`.

Ahora que nuestras funciones son public, podemos hacer uso de ellas. Grandioso! Sin embargo, escribir `frases::ingles::saludos::hola()` es largo y repetitivo. Rust posee otra palabra reservada para importar nombres en el ambito actual, de manera que puedes hacer referencia a ellos con nombres mas cortos, Hablemos acerca de `use`.

# Importando Modulos con `use`

Rust has a `use` keyword, which allows us to import names into our local scope.
Let’s change our `src/main.rs` to look like this:

```rust,ignore
extern crate phrases;

use phrases::english::greetings;
use phrases::english::farewells;

fn main() {
    println!("Hello in English: {}", greetings::hello());
    println!("Goodbye in English: {}", farewells::goodbye());
}
```

The two `use` lines import each module into the local scope, so we can refer to
the functions by a much shorter name. By convention, when importing functions, it’s
considered best practice to import the module, rather than the function directly. In
other words, you _can_ do this:

```rust,ignore
extern crate phrases;

use phrases::english::greetings::hello;
use phrases::english::farewells::goodbye;

fn main() {
    println!("Hello in English: {}", hello());
    println!("Goodbye in English: {}", goodbye());
}
```

But it is not idiomatic. This is significantly more likely to introduce a
naming conflict. In our short program, it’s not a big deal, but as it grows, it
becomes a problem. If we have conflicting names, Rust will give a compilation
error. For example, if we made the `japanese` functions public, and tried to do
this:

```rust,ignore
extern crate phrases;

use phrases::english::greetings::hello;
use phrases::japanese::greetings::hello;

fn main() {
    println!("Hello in English: {}", hello());
    println!("Hello in Japanese: {}", hello());
}
```

Rust will give us a compile-time error:

```text
   Compiling phrases v0.0.1 (file:///home/you/projects/phrases)
src/main.rs:4:5: 4:40 error: a value named `hello` has already been imported in this module [E0252]
src/main.rs:4 use phrases::japanese::greetings::hello;
                  ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
error: aborting due to previous error
Could not compile `phrases`.
```

If we’re importing multiple names from the same module, we don’t have to type it out
twice. Instead of this:

```rust,ignore
use phrases::english::greetings;
use phrases::english::farewells;
```

We can use this shortcut:

```rust,ignore
use phrases::english::{greetings, farewells};
```

## Re-exporting with `pub use`

You don’t just use `use` to shorten identifiers. You can also use it inside of your crate
to re-export a function inside another module. This allows you to present an external
interface that may not directly map to your internal code organization.

Let’s look at an example. Modify your `src/main.rs` to read like this:

```rust,ignore
extern crate phrases;

use phrases::english::{greetings,farewells};
use phrases::japanese;

fn main() {
    println!("Hello in English: {}", greetings::hello());
    println!("Goodbye in English: {}", farewells::goodbye());

    println!("Hello in Japanese: {}", japanese::hello());
    println!("Goodbye in Japanese: {}", japanese::goodbye());
}
```

Then, modify your `src/lib.rs` to make the `japanese` mod public:

```rust,ignore
pub mod english;
pub mod japanese;
```

Next, make the two functions public, first in `src/japanese/greetings.rs`:

```rust,ignore
pub fn hello() -> String {
    "こんにちは".to_string()
}
```

And then in `src/japanese/farewells.rs`:

```rust,ignore
pub fn goodbye() -> String {
    "さようなら".to_string()
}
```

Finally, modify your `src/japanese/mod.rs` to read like this:

```rust,ignore
pub use self::greetings::hello;
pub use self::farewells::goodbye;

mod greetings;
mod farewells;
```

The `pub use` declaration brings the function into scope at this part of our
module hierarchy. Because we’ve `pub use`d this inside of our `japanese`
module, we now have a `phrases::japanese::hello()` function and a
`phrases::japanese::goodbye()` function, even though the code for them lives in
`phrases::japanese::greetings::hello()` and
`phrases::japanese::farewells::goodbye()`. Our internal organization doesn’t
define our external interface.

Here we have a `pub use` for each function we want to bring into the
`japanese` scope. We could alternatively use the wildcard syntax to include
everything from `greetings` into the current scope: `pub use self::greetings::*`.

What about the `self`? Well, by default, `use` declarations are absolute paths,
starting from your crate root. `self` makes that path relative to your current
place in the hierarchy instead. There’s one more special form of `use`: you can
`use super::` to reach one level up the tree from your current location. Some
people like to think of `self` as `.` and `super` as `..`, from many shells’
display for the current directory and the parent directory.

Outside of `use`, paths are relative: `foo::bar()` refers to a function inside
of `foo` relative to where we are. If that’s prefixed with `::`, as in
`::foo::bar()`, it refers to a different `foo`, an absolute path from your
crate root.

This will build and run:

```bash
$ cargo run
   Compiling phrases v0.0.1 (file:///home/you/projects/phrases)
     Running `target/debug/phrases`
Hello in English: Hello!
Goodbye in English: Goodbye.
Hello in Japanese: こんにちは
Goodbye in Japanese: さようなら
```

## Complex imports

Rust offers several advanced options that can add compactness and
convenience to your `extern crate` and `use` statements. Here is an example:

```rust,ignore
extern crate phrases as sayings;

use sayings::japanese::greetings as ja_greetings;
use sayings::japanese::farewells::*;
use sayings::english::{self, greetings as en_greetings, farewells as en_farewells};

fn main() {
    println!("Hello in English; {}", en_greetings::hello());
    println!("And in Japanese: {}", ja_greetings::hello());
    println!("Goodbye in English: {}", english::farewells::goodbye());
    println!("Again: {}", en_farewells::goodbye());
    println!("And in Japanese: {}", goodbye());
}
```

What's going on here?

First, both `extern crate` and `use` allow renaming the thing that is being
imported. So the crate is still called "phrases", but here we will refer
to it as "sayings". Similarly, the first `use` statement pulls in the
`japanese::greetings` module from the crate, but makes it available as
`ja_greetings` as opposed to simply `greetings`. This can help to avoid
ambiguity when importing similarly-named items from different places.

The second `use` statement uses a star glob to bring in _all_ symbols from the
`sayings::japanese::farewells` module. As you can see we can later refer to
the Japanese `goodbye` function with no module qualifiers. This kind of glob
should be used sparingly.

The third `use` statement bears more explanation. It's using "brace expansion"
globbing to compress three `use` statements into one (this sort of syntax
may be familiar if you've written Linux shell scripts before). The
uncompressed form of this statement would be:

```rust,ignore
use sayings::english;
use sayings::english::greetings as en_greetings;
use sayings::english::farewells as en_farewells;
```

As you can see, the curly brackets compress `use` statements for several items
under the same path, and in this context `self` just refers back to that path.
Note: The curly brackets cannot be nested or mixed with star globbing.
