% Crates y Modulos

Cuando un proyecto comienza a crecer, se considera buena practica de ingeniería de software dividirlo en un grupo de componentes mas pequeños que posteriormente serán unidos. También es importante tener una interfaz bien definida, de manera que una parte de la funcionalidad sea privada, y otra publica. Para facilitar esto, Rust posee un sistema de módulos.

# Terminologia basica: Crates y Modulos

Rust posee dos términos distintos relacionados con el sistema de módulos: ‘crate’ y ‘modulo’. Un crate es un sinónimo para ‘biblioteca’ or ‘paquete’ en otros lenguajes. De allí el nombre “Cargo” del manejador de paquetes de Rust: haces disponibles tus crates a los demás con Cargo. Los crates pueden producir un ejecutables o una biblioteca, dependiendo del proyecto.

Cada crate posee un *modulo raiz* implícito que contiene el código para dicho crate. Puedes definir un árbol de sub-modulos bajo el módulo raíz. Los módulos te permiten particionar tu codigo dentro del crate.

Como ejemplo, creemos un crate *frases*, que nos proveerá varias frases en diferentes idiomas. Para mantener las cosas simples, nos apegaremos solo a ‘saludos’ y ‘despedidas’ como los dos tipos de frases, así como Ingles y Japonés (日本語) para los dos idiomas en los que las frases estarán representadas. Tendremos la siguiente distribución de módulos:


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
               +---|  japonés  |--+
                   +-----------+  |
                                  |   +------------+
                                  +---| despedidas |
                                      +------------+
```

En este ejemplo, `frases` es el nombre de nuestro crate. El resto son módulos. Puedes observar que los módulos forman un árbol, partiendo desde la *raíz*, que a su vez es la raíz del árbol `frases` en si.

Ahora que tenemos un plan, definamos estos módulos en código. Para comenzar, generemos un proyecto con Cargo:

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

`src/lib.rs` es la raíz de nuestro crate, correspondiendo con `frases` en nuestro diagrama anterior.
above.

# Definiendo Módulos

Para definir cada uno de nuestros módulos, usamos la palabra reservada `mod`. Hagamos que nuestro `src/lib.rs` se vea así:

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

Después de la palabra reservada `mod` debes proporcionar el nombre del modulo. Los nombres de de modulo siguen las mismas convenciones que los demás identificadores en Rust: `snake_case_en_minusculas`. El contenido de cada modulo esta delimitado por llaves (`{}`).

Dentro de un determinado `mod`, puedes declarar sub-`mod`s. Podemos referirnos a los sub-modulos con una notación dos puntos dobles (`::`): nuestros cuatro módulos anidados son `ingles::saludos`, `ingles::despedidas`, `japones::saludos`, y `japones::despedidas`. Debido a que estos sub-modulos están dentro del espacio de nombres del modulo padre, los nombres no entran en conflicto: `ingles::saludos` y `japones::saludos` son distintos incluso cuando sus nombres son ambos `saludos`.

Debido a que este crate no posee una función `main()`, y se llama `lib.rs`, Cargo construirá este crate como una biblioteca:

```bash
$ cargo build
   Compiling frases v0.0.1 (file:///home/tu/proyectos/frases)
$ ls target/debug
build  deps  examples  libfrases-a7448e02a0468eaa.rlib  native
```

`libfrases-hash.rlib` es el crate compilado. Antes de ver como usar este crate desde otro, dividámoslo en multiples archivos.

# Crates con multiple archivos

Si cada crate fuera un solo archivo, dichos archivos serian bastante grandes. Es mas fácil dividir los crates en multiples archivos. Rust soporta esto de dos maneras.

En lugar de declarar un modulo así:

```rust,ignore
mod ingles {
    // el contenido del modulo va aquí
}
```

Podemos declararlo de esta forma:

```rust,ignore
mod ingles;
```

Si hacemos eso, Rust esperara encontrar bien un archivo `ingles.rs` o un archivo `ingles/mod.rs` con el contenido de nuestro modulo.

Nota que todos en estos archivos, no necesitas re-declarar el modulo: con la declaración inicial `mod` es suficiente.

Usando estas dos técnicas, podemos partir nuestro crate en dos directorios y siete archivos:

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
│   ├── japones
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

`src/lib.rs` es la raíz de nuestro crate, y luce así:

```rust,ignore
mod ingles;
mod japones;
```

Estas dos declaraciones le dicen a Rust que busque bien sea `src/ingles.rs` y `src/japones.rs`, o `src/ingles/mod.rs` y `src/japones/mod.rs`, dependiendo de nuestra preferencia. En este caso, debido a que nuestros módulos poseen sub-modulos hemos seleccionado la segunda. Ambos `src/ingles/mod.rs` y `src/japones/mod.rs`se ven lucen de esta manera:

```rust,ignore
mod saludos;
mod despedidas;
```

De nuevo, estas declaraciones hacen que Rust busque bien sea por `src/ingles/saludos.rs` y `src/japones/saludos.rs` o `src/ingles/despedidas/mod.rs` and `src/japones/despedidas/mod.rs`. Debido a que los sub-modulos no poseen sus propios sub-modulos, hemos optado por usar el enfoque `src/ingles/saludos.rs` and `src/japones/despedidas.rs`. Uff!

Ambos `src/ingles/saludos.rs` y `src/japones/despedidas.rs` están vacíos por el momento. Agreguemos algunas funciones.

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

Por supuesto, puedes copiar y pegar el Japones desde esta pagina web, o simplemente escribe alguna otra cosa. No es importante que en efecto coloques ‘konnichiwa’ para aprender acerca del sistema de módulos de Rust.

Coloca lo siguiente en `src/japones/despedidas.rs`:

```rust
fn adios() -> String {
    "さようなら".to_string()
}
```

(‘Sayōnara’, por si eres curioso.)

Ahora que tenemos algo de funcionalidad en nuestro crate, intentemos usarlos desde otro.

# Importing External Crates

Ya tenemos un crate biblioteca. Creemos un crate ejecutable que importe y use nuestra biblioteca.

Crea un archivo `src/main.rs` y coloca esto en el (no compilara todavía):

```rust,ignore
extern crate frases;

fn main() {
    println!("Hola en Ingles: {}", frases::ingles::saludos::hola());
    println!("Adios en Ingles: {}", frases::ingles::despedidas::adios());

    println!("Hola en Japones: {}", frases::japones::saludos::hola());
    println!("Adios en Japones: {}", frases::japones::despedidas::adios());
}
```

La declaración `extern crate` le informa a Rust que necesitamos compilar y enlazar al crate `frases`. Podemos entonces usar el modulo `frases` aqui. Como mencionamos anteriormente, puedes hacer uso de dos puntos dobles para hacer referencia a los sub-modulos y las funciones dentro de ellos.

Nota: cuando se importa un crate que tiene guiones en su nombre "como-este", lo cual no es un identificador valido en Rust, sera convertido cambiándole los guiones a guiones bajos, así que escribirías algo como `extern crate como_este;`

También, Cargo assume que `src/main.rs` es la raíz de un crate binario, en lugar de un crate biblioteca. Nuestro paquete ahora tiene dos crate: `src/lib.rs` y `src/main.rs`. Este patron es muy común en crates ejecutables: la mayoría de la funcionalidad esta en un crate biblioteca, y el crate ejecutable usa dicha biblioteca. De esta forma, otros programas pueden también hacer uso de la biblioteca, aunado a que ofrece un buena separación de responsabilidades.

Lo anterior todavía no funciona. Obtenemos cuatro errores que lucen similares a estos:

```bash
$ cargo build
   Compiling frases v0.0.1 (file:///home/tu/proyectos/frases)
src/main.rs:4:38: 4:72 error: function `hola` is private
src/main.rs:4     println!("Hola en Ingles: {}", frases::ingles::saludos::hola());
                                                 ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
note: in expansion of format_args!
<std macros>:2:25: 2:58 note: expansion site
<std macros>:1:1: 2:62 note: in expansion of print!
<std macros>:3:1: 3:54 note: expansion site
<std macros>:1:1: 3:58 note: in expansion of println!
frases/src/main.rs:4:5: 4:76 note: expansion site
```

Por defecto todo es privado en Rust. Hablemos de esto con mayor detalle.

# Exportando una Interfaz Publica

Rust te permite controlar de manera precisa cuales aspectos de tu interfaz son públicos, y private es el defacto. Para hacer a algo publico, debes hacer uso de la palabra reservada `pub`. Enfoquemonos primero en el modulo `ingles`, para ello, reduzcamos nuestro `src/main.rs` a:

```rust,ignore
extern crate frases;

fn main() {
    println!("Hola en Ingles: {}", frases::ingles::saludos::hola());
    println!("Adios en Ingles: {}", frases::ingles::despedidas::adios());
}
```

En nuestro `src/lib.rs`, agreguemos `pub` a la declaración del modulo `ingles`:

```rust,ignore
pub mod ingles;
mod japones;
```

Y en nuestro `src/ingles/mod.rs`, hagamos a ambos `pub`:

```rust,ignore
pub mod saludos;
pub mod despedidas;
```

En nuestro `src/ingles/saludos.rs`, agreguemos `pub` a nuestra declaración `fn`:

```rust,ignore
pub fn hola() -> String {
    "Hola!".to_string()
}
```

Y también en `src/ingles/despedidas.rs`:

```rust,ignore
pub fn adios() -> String {
    "Adios.".to_string()
}
```

Ahora, nuestro crate compila, con unas pocas advertencias acerca de no haber usado las funciones en `japones`:

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

`pub` también aplica a las `struct`s y sus campos miembro. Y Rust teniendo siempre su tendencia hacia la seguridad, el hacer una `struct` public no hará a sus miembros public automáticamente: debes marcarlos individualmente como `pub`.

Ahora que nuestras funciones son public, podemos hacer uso de ellas. Grandioso! Sin embargo, escribir `frases::ingles::saludos::hola()` es largo y repetitivo. Rust posee otra palabra reservada para importar nombres en el ámbito actual, de manera que puedas hacer referencia a ellos con nombres mas cortos, Hablemos acerca de `use`.

# Importando Modulos con `use`

Rust posee una palabra reservada, `use`, que te permite importar nombres en tu ámbito local. Cambiemos nuestro `src/main.rs` para que luzca de la siguiente manera:

```rust,ignore
extern crate frases;

use frases::ingles::saludos;
use frases::ingles::despedidas;

fn main() {
    println!("Hola en Ingles: {}", saludos::hola());
    println!("Adios en Ingles: {}", despedidas::adios());
}
```

Las dos lineas `use` importan cada modulo en el ámbito local, de manera tal que podamos referirnos a las funciones con nombres mucho mas cortos. Por convención, cuando se importan funciones, se considera una buena practica importar el modulo, en lugar de la función directamente. En otras palabras, puedes _hacer_ esto:

```rust,ignore
extern crate frases;

use frases::ingles::saludos::hola;
use frases::ingles::despedidas::adios;

fn main() {
  println!("Hola en Ingles: {}", hola());
  println!("Adios en Ingles: {}", adios());
}
```

Pero no es idiomático. Hacerlo de esta forma tiene altas probabilidades de introducir un conflicto de nombres. En nuestro pequeño programa, no es gran cosa, pero a medida que crece, se va convirtiendo en un problema. Si tenemos nombres conflictivos, Rust generara un error de compilación. Por ejemplo, de haber hecho publicas las funciones de `japones` y haber intentado:

```rust,ignore
extern crate frases;

use frases::ingles::saludos::hola;
use frases::japones::saludos::hola;

fn main() {
    println!("Hola en Ingles: {}", hola());
    println!("Hola en Japones: {}", hola());
}
```

Rust proporcionaría un error en tiempo de compilación:

```text
   Compiling frases v0.0.1 (file:///home/tu/frases)
src/main.rs:4:5: 4:40 error: a value named `hola` has already been imported in this module [E0252]
src/main.rs:4 use frases::japones::saludos::hola;
                  ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
error: aborting due to previous error
Could not compile `frases`.
```

Si estuviéramos importando multiples nombres del mismo modulo, no necesitamos escribirlo dos veces. En lugar de:

```rust,ignore
use phrases::ingles::saludos;
use phrases::ingles::despedidas;
```

Podemos usar esta version mas corta:

```rust,ignore
use phrases::ingles::{saludos, despedidas};
```

## Re-exportando con `pub use`

No solo usamos `use` para acortar identificadores. Puedes también usarlos dentro de tu crate para re-exportar una función dentro de otro modulo. Esto te permite presentar una interfaz externa que necesariamente no mapee de directamente a la organización interna de tu código.

Veamos un ejemplo. modifica tu `src/main.rs` para que se lea así:

```rust,ignore
extern crate frases;

use frases::ingles::{saludos, despedidas};
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

A continuación, haz las dos funciones publicas, primero en `src/japones/saludos.rs`:

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

Finalmente, modifica tu `src/japones/mod.rs` de esta forma:

```rust,ignore
pub use self::saludos::hola;
pub use self::despedidas::adios;

mod saludos;
mod despedidas;
```

La declaración `pub use` trae la función a el ámbito en esta parte de nuestra jerarquía de modulos. Debido que hemos hecho `pub use` dentro de nuestro modulo `japones`, ahora tenemos una función `frases::japones::hola()` y una función `frases::japones::adios()`, aun cuando el código para ellas vive en `frases::japones::saludos::hola()` y frases::japones::despedidas::adios()`. Nuestra organización interna no define nuestra interfaz externa.

Acá tenemos un `pub use` para cada función que deseamos traer en el ámbito de `japones`. Alternativamente pudimos haber usado la sintaxis alternativa de comodín para incluir todo desde `saludos` en el ámbito actual: `pub use self::saludos::*`

Que hay acerca de el `self`? Bueno, por defecto, las declaraciones `use` son rutas absolutas, partiendo desde la raíz de tu crate. `self`, a diferencia, hace esa ruta relativa a tu lugar actual dentro de la jerarquía. Hay una ultima forma especial de usar `use`: puedes usar `use super::` para alcanzar un nivel superior en la jerarquía desde tu posición actual. Algunas personas gustan ver a `self` como `.` y `super` como `..` similarmente los usados por los shells para mostrar los directorios actual y padre respectivamente.

Fuera de `use`, las rutas son relativas: `foo::bar()` se refiere a una función dentro de `foo` en relación a en donde estamos. Si posee un prefijo `::`, como en `::foo::bar()`, entonces se refiere a un `foo` diferente, una ruta absoluta desde la raíz de tu crate.

El ultimo código que escribimos, compilara y se ejecutara sin problemas:

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

Rust ofrece un numero de opciones avanzadas que pueden hacer tus sentencias `extern crate` mas compactas y convenientes. He aquí un ejemplo:

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

Que esta pasando?

Primero, ambos `extern crate` y `use` permiten renombrar lo que esta siendo importado. Entonces el crate todavía se llama "frases", pero aquí nos referiremos a el como "dichos". Similarmente, el primer `use` trae el modulo `japones::saludos` desde el crate, pero lo hace disponible a través del nombre `saludos_ja` en lugar de simplemente `saludos`. Lo anterior puede ayudar a evitar ambigüedad cuando se importan nombres similares desde distintos lugares.

El segundo `use` posee un asterisco para traer _todos_ los símbolos desde el modulo `dichos::japones::despedidas`. Como podrás ver mas tarde podemos referirnos al `adios` Japones sin calificadores de modulo. Este tipo de glob debe ser usando con cautela.

El tercer `use` requiere un poco mas de explicación. Esta usando "expansion de llaves" para comprimir tres sentencias `use` en una (este tipo de sintaxis puede serte familiar si has escrito scripts del shell de Linux). La forma descomprimida de esta sentencia seria:

```rust,ignore
use dichos::ingles;
use dichos::ingles::saludos as saludos_en;
use dichos::ingles::despedidas as despedidas_en;
```

Como puedes ver, las llaves comprimen las sentencias `use` para varios items bajo la misma ruta, y en este contexto `self` hace referencia a dicha ruta. Nota: Las llaves no pueden ser anidadas o mezcladas con globbing de asteriscos.
