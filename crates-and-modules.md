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

De nuevo, estas declaraciones hacen que Rust busque bien sea por `src/ingles/saludos.rs` y `src/japones/saludos.rs` o `src/ingles/despedidas/mod.rs` and `src/japones/despedidas/mod.rs`. Debido a que los submodulos no poseen sus propios submodulos, hemos optado por usar el enfoque `src/ingles/saludos.rs` and `src/japones/despedidas.rs`. Uff!

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

Rust posee una palabra reservada, `use`, que te permite importar nombres en tu ambito locoal. Cambiemos nuestro `src/main.rs` para que luzca de la siguiente manera:

```rust,ignore
extern crate frases;

use frases::ingles::saludos;
use frases::ingles::despedidas;

fn main() {
    println!("Hola en Ingles: {}", saludos::hola());
    println!("Adios en Ingles: {}", despedidas::adios());
}
```

Las dos lineas `use` importan cada modulo en el ambito local, de manera tal que podamos referirnos a las funciones con nombres mucho mas cortos. Por convencion, cuando se importan funciones, se considera una buena practica importar el modulo, en lugar de la funcion directamente. En otras palabras, puedes _hacer_ esto:

```rust,ignore
extern crate frases;

use frases::ingles::saludos::hola;
use frases::ingles::despedidas::adios;

fn main() {
  println!("Hola en Ingles: {}", hello());
  println!("Adios en Ingles: {}", adios());
}
```

Pero no es idomatico. Lo anterior tiene significativas probabilidaes de introducir un conflicto de nombres. En nuestro pequeno programa, no es gran cosa, pero a medida que crece, se va convirtiendo en un problema. Si tensmo nombres conflictivos, Rust generara un error de compilacion. Por ejemplo, de haber hecho publicas las funciones de `japones` y haber intentado:

```rust,ignore
extern crate frases;

use frases::ingles::saludos::hola;
use frases::japones::saludos::hola;

fn main() {
    println!("Hola en Ingles: {}", hello());
    println!("Hola en Japones: {}", hello());
}
```

Rust propircionaria un error en tiempo de compilacion:

```text
   Compiling frases v0.0.1 (file:///home/tu/frases)
src/main.rs:4:5: 4:40 error: a value named `hola` has already been imported in this module [E0252]
src/main.rs:4 use frases::japones::saludos::hola;
                  ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
error: aborting due to previous error
Could not compile `frases`.
```

Si estuvieramos importando multiples nombres del mismo modulo, no necesitamos escribirlo dos veces. En lugar de:

```rust,ignore
use phrases::ingles::saludos;
use phrases::ingles::despedidas;
```

Podemos usar esta version mas corta:

```rust,ignore
use phrases::ingles::{saludos, despedidas};
```

## Re-exportando con `pub use`

No solo usamos `use` para acortar identificadores. Puedes tambien usarlos dentro de tu crate para re-exportar una funcion dentro de optro modulo. Esto te permite presentar una interfaz extrerna que pueda no mapear de directamente a la organizacion interna de tu codigo.

Veamos un ejemplo. modifica tu `src/main.rs`  para que se lea asi:

```rust,ignore
extern crate frases;

use frases::ingles::{saludos,despedidas};
use frases::japones;

fn main() {
    println!("Hola en Ingles: {}", saludos::hola());
    println!("Adios en Ingles: {}", despedidas::adios());

    println!("Hola en Japones: {}", japones::hola());
    println!("Adios en Japones: {}", japones::adios());
}
```

Entonces, modifica tu `src/lib.rs` para hacer el modulo `japones` publico:

```rust,ignore
pub mod ingles;
pub mod japones;
```

A continuacion, haz las dos funciones publicas, primero en `src/japones/saludos.rs`:

```rust,ignore
pub fn hola() -> String {
    "こんにちは".to_string()
}
```

Y luego en `src/japones/despedidas.rs`:

```rust,ignore
pub fn adios() -> String {
    "さようなら".to_string()
}
```

Finalmente, modifica tu `src/japanese/mod.rs` para que se vea asi:

```rust,ignore
pub use self::saludos::hola;
pub use self::despedidas::adios;

mod saludos;
mod despedidas;
```

La declaracion `pub use` trae la funcion a el ambito en esta parte de nuestra jerarquia de modulos. Debido que hemos hecho `pub use` dentro de nuestro modulo `japones`, ahora tenemos una funcion `frases::japones::hola()` y una funcion `frases::japones::adios()`, aun cuando el codigo para ellas vive en `frases::japones::saludos::hola()` y frases::japones::despedidas::adios()`.  Nuestra organizacion interna no define nuestra interfaz extrerna.

Aca tenemos un `pub use` para cada funcion que deseamos traer en el ambito de `japones`. Alternativamente pudimos haber usado la sintaxis alternativa de comodin para incluir todo desde `saludos` en el ambito actual: `pub use self::saludos::*`

Que hay acerca de el `self`? Bueno, por defecto, las declaraciones `use` son rutas absolutas, partendo desde la raiz de tu crate. `self`, a diferencia, hace esa ruta relativa a tu lugar actual dentro de la jerarquia. Hay una ultima frma especial de usar `use`: puedes usar `use super::` para alcanzar un nivel superior en la jeraquia desde tu posicion actual. Algunas personas gustan ver a `self` como `.` y `super` como `..` como los usados por los shells para mostrar los directorios actual y padre respectivamente.

Fuera de `use`, las rutas son relativas: `foo::bar()` se refiere a una funcion dentro de `foo` en relacion a en donde estamos. Si posee un prefijo `::`, como en `::foo::bar()`, entonces se refiere a un `foo` diferente, una ruta absoluta desdela raiz de tu crate.

El ultimo codigo que escribimos, compilara y se ejecutara sin problemas:

```bash
$ cargo run
   Compiling frases v0.0.1 (file:///home/tu/proyectos/frases)
     Running `target/debug/frases`
Hola in Ingles: Hello!
Adios in Ingles: Goodbye.
Hola in Japones: こんにちは
Adios in Japones: さようなら
```

## Importes complejos

Rust ofrece un numero de opciones avanzadas que pueden hacer tus sentencias `extern crate` mas compactas y convenientes. He aqui un ejemplo:

```rust,ignore
extern crate frases as dichos;

use dichos::japones::saludos as saludos_ja;
use dichos::japones::despedidas::*;
use dichos::ingles::{self, saludos as saludos_en, despedidas as despedidas_en};

fn main() {
    println!("Hola en Ingles; {}", saludos_en::hola());
    println!("Y en Japones: {}", saludos_ja::hola());
    println!("Goodbye in Ingles: {}", ingles::despedidas::adios());
    println!("Otra vez: {}", despedidas_en::adios());
    println!("Y en Japones: {}", adios());
}
```

Que esta pasando aca?

Primero, ambos `extern crate` y `use` permiten renombrar lo que esta siendo importado. Entonces el crate todavia se llama "frases", pero aqui nos referiremos a el como "dichos". Similarmente, el primer `use` trae el modulo `japones::saludos` desde el crate, pero lo hace disponible a traves del nombre `saludos_ja` en lugar de simplemente `saludos`. Lo anterior puede ayudar a evitar ambiguedad cuando se importan nmbres similares desde distintos lugares.

El segundo `use` posee un asterisco para traer _todos_ los simbolos desde el modulo `dichos::japones::despedidas`. Como podras ver mas tarde podemos referirnos al `adios` Japones sin calificadores de modulo. Este tipo de glob debe ser usando con cautela.

The second `use` statement uses a star glob to bring in _all_ symbols from the
`sayings::japanese::farewells` module. As you can see we can later refer to
the Japanese `goodbye` function with no module qualifiers. This kind of glob
should be used sparingly.

El tercer `use` requiere un poco mas de explicacion. Esta usando "expansion de llaves" para comprimir tres sentencias `use` en una (este tipo de sintaxis puede ser familiar si has excrito scripts del shell de Linux con anterioridad). La forma descomprimida de esta sentencia seria:

```rust,ignore
use dichos::ingles;
use dichos::ingles::saludos as saludos_en;
use dichos::ingles::despedidas as despedidas_en;
```

Como puedes ver, las llaves comprimen las sentencias `use` para varios items bajo la misma ruta, y en este contexto `self` hace referencia a dicha ruta. Nota: Las llaves no pueden ser anidadas o mezcladas con globbing de asteriscos.
