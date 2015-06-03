# Juego de Adivinanzas

Para nuestro primer proyecto, implementaremos un problema clásico de programación para principiantes: el juego de las adivinanzas. Como funciona este juego: Nuestro programa generarara un numero entero aleatorio entre uno y cien. Nos pedira que introduzcamos un valor. Despues de haber proporcionado nuestro numero, este nos dira si estuvimos muy por debajo y muy por encima. Una vez que adivinemos el numero correcto, nos felicitara. Te suena bien

# Configuracion Inicial

Creemos un nuevo proyecto. Anda a tu directorio de proyectos. Recuerdas como creamos nuestra estructura de directorios y un `Cargo.toml` para `hola_mundo`? Cargo posse un comando que hace eso por nosotros. Intentemos probarlo:

```bash
$ cd ~/proyectos
$ cargo new adivinanzas --bin
$ cd adivinanzas
```

Pasamos el nombre de nuestro proyecto a `cargo new`, junto con el flag `--bin`, debido a que estamos creando un binario, en vez de una biblioteca.

Echa un vistazo al Cargo.toml` generado

```toml
[package]

name = "adivinanzas"
version = "0.1.0"
authors = ["Tu Nombre <tu@ejemplo.com>"]
```

Cargo obtiene esta información  de tue entorno. Si no esta correcta, anda y corrigela.

Finalmente, Cargo ha generado un ‘Hola, mundo!’ para nosotros. Echa un vistazo a `src/main.rs`:

```rust
fn main() {
    println!("Hello, world!");
}
```

Tratemos de compilar lo que Cargo nos ha proporcionado:

```{bash}
$ cargo build
   Compiling adivinanzas v0.1.0 (file:///home/tu/proyectos/adivinanzas)
```

Excelente! Abre tu `src/main.rs` otra vez. Estaremos escribiendo todo nuestro codigo en este archivo.

Antes de que continuemos, dejame mostrarte un comoando mas de Cargo: `run`. `cargo run` es una especie de `cargo build`, pero con la diferencia de que tambien ejecuta el binario producido. Pruebalo: 

```bash
$ cargo run
   Compiling adivinanzas v0.1.0 (file:///home/tu/proyectos/adivinanzas)
     Running `target/debug/adivinanzas`
Hola, mundo!
```

Grandioso! El comando `run` es muy util cuando necesitas iterar rapido en un proyecto. Nuestro juego es uno de esos prpyectos, necesitaremos probar rapidamente cada iteracion antes de movernos a la siguiente.

# Procesando un Intento de Adivinanza

Probemoslo! La primera cosa que necesitamos hacer para nuestro juego de adivinanzas es permitir a nuestro jugador entrar un intento de adivinanza. Coloca esto en tu `src/main.rs`:

```rust,no_run
use std::io;

fn main() {
    println!("Adivina el numero!");

    println!("Por favor introduzca su adivinanza.");

    let mut adivinanza = String::new();

    io::stdin().read_line(&mut adivinanza)
        .ok()
        .expect("Fallo lectura de linea");

    println!("Haz adivinado: {}", adivinanza);
}
```

Hay un monton aqui! Tratemos de ir a traves de ello, pieza por pieza.

```rust,ignore
use std::io;
```

Necesitaremos recibir entrada del usuario, y luego imprimir el resultado como salida. Debido a esto necesitamos la biblioteca `io` de la biblioteca estandar. Rust solo importa pocas para en todos los programas, este conjunto de cosas se denomina [‘preludio’][prelude]. Si no esta en el preludio tendras que llamarlo directamente a traves de `use`.

[prelude]: ../std/prelude/index.html

```rust,ignore
fn main() {
```

Como has visto con anterioridad, la función `main()` es el punto de entrada a tu programa. La sintaxis `fn` declara una nueva función, los `()`s indican que no hay argumentos y  `{` comienza el cuerpo de la función. Debido a que no incluimos un tipo de retorno, se asume ser `()` una [tupla][tuples] vacía.

[tuples]: primitive-types.html#tuples

```rust,ignore
    println!("Adivina el numero!");

    println!("Por favor introduzca su adivinanza.");
```

Anteriormente aprendimos que `println!()` es un [macro][macros] que imprime una [cadena de caracteres][strings] a la pantalla.

[macros]: macros.html
[strings]: strings.html

```rust,ignore
    let mut adivinanza = String::new();
```

Ahora nos estamos poniendo interesantes! Hay un monton de cosas pasando en esta pequeña linea. La primera cosas a notar es que es una [sentencia let][let], que es usada para crear variables. Tiene la forma:

```rust,ignore
let foo = bar;
```

[let]: variable-bindings.html

Esto creara una nueva variable llamada `foo`, y la enlazara al valor `bar`. En muchos lenguajes, esto es llamado una ‘variable’ pero las variables de Rust tienen un par de trucos bajo la manga.

Por ejemplo, estas son [immutables][immutable] por defecto. Es por ello que nuestro ejemplo usa `mut`: esto hace un binding mutable, en vez de inmutable. `let` no toma un nombre del lado izquierdo, `let` acepta un ‘[patrón][patterns]’. Usaremos los patrones un poco mas tarde. Es suficiente usar po ahora: 

```rust
let foo = 5; // immutable.
let mut bar = 5; // mutable
```

[immutable]: mutability.html
[patterns]: patterns.html

An, y `//` inicia un comentario, hasta el final de la linea. Rust ignora todo en [comentarios][comments]

[comments]: comments.html

Entonces sabemos que `let mut adivinanza` introducirá un binding mutable llamado `adivinanza`, pero tenemos que ver al otro lado del `=` para saber a que esta asociado: `String::new()`.

`String` es un tipo de cadena de caracter, proporcionado por la biblioteca estandar. Un  [`String`][string] es un segmento de texto codificado en UTF-8 capaz de crecer.

[string]: ../std/string/struct.String.html

La sintaxis `::new()` usa `::` porque es una ‘función asociada’ de un tipo particular. Es decir esta asociada con `String` en si mismo, en vez de con una isntacia en particular de `String`. Algunos lenguajes llaman a esto un ‘metodo estatico

Esta funcion es llamada `new()`, porque crea un nuevo `String` vacio. Encontraras una funcion `new()` en muchos tipos, debido a que es un nombre común para la creacion de un nuevo valor de algun tipo. 

Continuemos:

```rust,ignore
    io::stdin().read_line(&mut adivinanza)
        .ok()
        .expect("Fallo lectura de linea");
```

Otro monton! Vallamos pieza por pieza. La primera linea tiene dos partes. He aqui la primera:

```rust,ignore
io::stdin()
```

Recuerdas como usamos `use` en `std::io` en la primera linea de nuestro programa? Ahora estamos llamando una funcion asociada en `std::io`. De no haber usado `use std::io`, pudimos haber escrito esta linea como `std::io::stdin()`.

Esta función en particular retorna un habdle a la entrada estandar de tu terminal. Mas especificamente, un [std::io::Stdin][iostdin].

[iostdin]: ../std/io/struct.Stdin.html

La siguiente parte usara dicho hadle para obtener entrada del usuario:

```rust,ignore
.read_line(&mut adivinanza)
```

Aqui, llamamos el metodo [`read_line()`][read_line] en nuestro handle. Los [metodos][method] son como las funciones asociadas, pero solo estan disponibles en una instancia en particular de un tipo, en vez de en el tipo en si. Tambien estamos pasandoun argumento a `read_line()`: `&mut adivinanza`.

[read_line]: ../std/io/struct.Stdin.html#method.read_line
[method]: method-syntax.html

Recuerdas cuando creamos `adivinanza`? Dijimos que era mutable. Sin embargo `read_line` no acepta un `String` como argumento: acepta un `&mut String`. Rust tiene una caracteristica llamada ‘[referencias][references]’, que te permite tener multiples referencias a una pieza de data, lo cual puede reducir el copiado. Estas son una caracteristica compleja, debido a que uno de los puntos de venta mas fuertes de Rust es acerca de cuan fácil y seguro es usar referencias. Por ahora no necesitamos saber mucho de esos detalles para finalizar nuestro programa. Todo lo que necesitamos saber por el momento es que al igual que los bindings `let` las referencias son inmutables por defecto. Como consecuencia necesitamos escribir `&mut adivinanza` en vez de `&adivinanza`.

Porque `read_line()` acepta una referencia mutable a una cadena de caracteres. Su trabajo es tomar lo que el usuario ingresa en la entrada estandar, y colocarlo en una cadena de caracteres. Debido a ello toma dicha cadena como argumento, y debidoa que debe de agregar la entrada del usuario, este necesita ser mutable.

[references]: references-and-borrowing.html

Todavia no hemos terminado con esta linea. Si bien es una sola linea de texto, es solo la primera parte de una linea logica de codigo completa:

```rust,ignore
        .ok()
        .expect("Fallo lectura de linea");
```

Cuando llamas a un metodo con la sintaxis `.foo()` puedes introducir un salto de linea y otro espacio. Esto te ayuda a dividir lineas largas. Pudimos haber escrito:

```rust,ignore
    io::stdin().read_line(&mut adivinanza).ok().expect("Fallo lectura de linea");
```

Pero eso se hace dificil de leer. Asi que lo hemos dividido en tres lineas para tres llamadas a metodo. Yaa hemos hablado de `read_line()`, pero que acerca de `ok()` y `expect()`? Bueno, ya mencionamos que `read_line()` coloca la entrada del usuario en el `&mut String` que le proprocionamos. Pero tambien retorna un valor: en este caso un [`io::Result`][ioresult]. Rust posse un numero de tipos llamados `Result` en su biblioteca estandar: un [`Result`][result] generico, y versiones especificas para sub-librerias, como `io::Result`.

[ioresult]: ../std/io/type.Result.html
[result]: ../std/result/enum.Result.html


El proposito de esos `Result` es codificar informacion de manejo de errores. Valores del tipo `Result` tienen metodos definidos en ellos. En este caso `io::Result` posee un metodo `ok()`, que se traduce en ‘queremos asumir que este valor es un valor exitoso. Sino, descarta la información acerca del error’. Porque descartar la información acerca del error?, para un programa basico, queremos simplemente imprimir un error generico, cualquier problema que signifique que no podamos continuar. El [metodo `ok()`][ok] retorna un valor que tiene otro metodo definito en el: `expect()`. El [`metodo expect()`][expect] toma el valor en el cual es llamado y si no es un valor exitoso, hace panico [`panic!`][panic] con un mensaje que le hemos proporcionado. Un `panic!` como este causara que el programa tenga una salida abrupta (crash), mostrando dicho mensaje.

[ok]: ../std/result/enum.Result.html#method.ok
[expect]: ../std/option/enum.Option.html#method.expect
[panic]: error-handling.html

Si quitamos las llamadas a esos  dos metodos, nuestro programa compilara, pero obtendremos una advertencia:

```bash
$ cargo build
   Compiling adivinanzas v0.1.0 (file:///home/tu/proyectos/adivinanzas)
src/main.rs:10:5: 10:39 warning: unused result which must be used,
#[warn(unused_must_use)] on by default
src/main.rs:10     io::stdin().read_line(&mut adivinanza);
                   ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

Rust nos advierte que no hemos usado el valor `Result`. Esta advertencia viene de una anotacion especial que tiene `io::Result`. Rust esta tratando de decirte que no has manejado un posible error. La manera correcta de suprimir el error es, en efecto escribir el codigo para el manejo de erroes. Por suerte, si solo queremos terminar la ejecucion del programa de haber un problema, podemos usador estos dos pequeños metodos. Si podemos recuperarnos del error de alguna manera, hariamos algo diferente,  pero dejemos eso para un proyecto futuro.

Solo nos queda una linea de este primer ejemplo:

```rust,ignore
    println!("Haz adivinado: {}", adivinanza);
}
```

Esta linea imprime la cadena de caracteres en la que guardamos neustra entrada. Los `{}`s son marcadores de posicion, es por ello que pasamos `adivinanza` como argumento.  De haber habido multiples `{}`s, debiamos haber pasado multiples argumentos:

```rust
let x = 5;
let y = 10;

println!("x y y: {} y {}", x, y);
```

Fácil.

De cualquier modo, ese es el tour. Podemos ejecutarlo con `cargo run`:

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///home/tu/proyectos/adivinanzas)
     Running `target/debug/adivinanzas`
Adivina el numero!
Por favor introduzca su adivinanza.
6
Haz adivinado: 6
```

En hora buena! Nuestra primera parte ha terminado: podemos obtener entrada del teclado e imprimirla de vuelta.

# Generando un numero secreto

A continuación necesitamos generar un numero secreto. Rust todavía no incluye una funcionalidad de numeros aleatorios en la biblioteca estándar. Sin embargo, el quipo de Rust provee un [`rand` crate][randcrate]. Un ‘crate’ es un paquete de código Rust. Nosotros hemos estado construyendo un ‘crate binaro’, el cual es un ejecutable. `rand` es un ‘crate biblioteca’, que contiene codigo a ser usado por otros programas.

[randcrate]: https://crates.io/crates/rand

Usar crates externos es donde Cargo realmente brilla. Antes que podamos escribir código que haga uso de `rand`, debemos modificar nuestro archivo `Cargo.toml`. Abrelo, y agrega estas lineas al final:

```toml
[dependencies]

rand="0.3.0"
```

La sección `[dependencies]` de `Cargo.toml` es como la sección `[package]`: todo lo que le sigue es parte de ella, hasta que la siguiente sección comience. Cargo usa la sección de dependencias para saber en cuales crates externos dependemos, asi como las versiones requeridas. En este caso hemos usado la version `0.3.0`. Cargo entiende [Versionado Semantico][semver], que es un estandar para las escritura de numeros de version. Si quisieramos usar la ultima version podriamos haber usado `*` o un rango de versiones. La [documentación de Cargo][cargodoc]  contiene mas detalles.

[semver]: http://semver.org
[cargodoc]: http://doc.crates.io/crates-io.html

Ahora, sin cambiar nada en nuestro código, construyamos nuestro proyecto:

```bash
$ cargo build
    Updating registry `https://github.com/rust-lang/crates.io-index`
 Downloading rand v0.3.8
 Downloading libc v0.1.6
   Compiling libc v0.1.6
   Compiling rand v0.3.8
   Compiling adivinanzas v0.1.0 (file:///home/tu/proyectos/adivinanzas)
```

(Podrias ver versiones diferentes, por supuesto.)

Un montón de salida mas! Ahora que tenemos una dependencia externa, Cargo descarga del registro las ultimas versiones de todo, lo cual puede copiar datos desde [Crates.io][cratesio]. Crates.io es donde la gente del ecosistema Rust publica sus proyectos open source para que otros los usen.


[cratesio]: https://crates.io

Despues de actualizar el registro, Cargo chequea nuestras dependencias (en `[dependencies]`) y las descarga de no tenerlas todavía. En este caso solo dijimos que queriamos depender en `rand`, y tambien obtuvimos una copia de `libc`. Esto es debido a que `rand` depende de `libc` para funcionar. Despues de descargar las dependencias, Cargo las compila, para despues compilar nuestro código.

Si ejecutamos  `cargo build`, obtendremos una salida diferente:


```bash
$ cargo build
```

Asi es, no hay salida! Cargo sabe que nuestro proyecto ha sido construido, asi como todas sus dependencias, asi que no nay razon para hacer todo el proceso otra vez. Sin nada que hacer, simplemente termina la ejecucion. Si abrimos `src/main.rs` otra vez, hacemos un cambio trivial, salvamos los cambios, solamente veriamos una linea:


```bash
$ cargo build
    Compiling adivinanzas v0.1.0 (file:///home/tu/proyectos/adivinanzas)
```

Entonces, le hemos dicho a Cargo `0.3.x` que queriamos cualquier versión `0.3.x` de `rand`, y este descargo la ultima version para el momento de la escritura de este tutorial, `v0.3.8`. Pero que pasa cuando la siguiente versión `v0.3.9` sea publicada con un importante bugfix? Si bien recibir bugfixes es importante, que pasa si `0.3.9` contiene una regresion que rompe nuestro codigo?

La respuesta a este problema es el archivo `Cargo.lock`, archivo que encontraras en tu directorio de proyecto. Cuando construyes tu proyecto por primera vez, cargo determina todas las versiones que coinciden con tus criterios y las escribe en el archivo `Cargo.lock`. Cuando construyes tu proyecto en el futuro, Cargo notara que que un archivo `Cargo.lock` existe, y usara las versiones especificadas en el mismo, en vez de hacer todo el trabajo de determinar las versiones otra vez. Esto te permite tener una construcción reproducible de manera automatica. En otras palabras, nos quedaremos en `0.3.8` hasta que subamos de version de manera explicita, de igual manera lo hará la gente con la que hemos compartido nuestro código, gracias al archivo `Cargo.lock`.

Pero que pasa cuando _queremos_ usar `v0.3.9`? Cargo posee otro comando, `update`, que se traduce en ‘ignora el bloqueo y determina todas las ultimas versiones que coincidan con lo que hemos especficado. De funcionar esto, escribe esas versiones al archivo de bloqueo `Cargo.lock`’. Pero, por defecto, Cargo solo buscara versiones mayores a `0.3.0`
y menores a `0.4.0`. Si queremos movernos a `0.4.x`, necesitariamos actualizar el archivo `Cargo.toml` directamente. Cuando lo hacemos, la siguente vez que ejecutemos `cargo build`, Cargo actualizara el indice y re-evaluara nuestros requerimentos acerca de `rand`.

Hay mucho mas que decir acerca de [Cargo][doccargo] y [su ecosistema][doccratesio], pero por ahora, eso es todo lo que necesitamos saber. Cargo hace realmente fácil reusar bibliotecas, y los Rusteros tienden a escribir proyectos pequenos los cuales estan construidos de un numero de paquetes mas pequeños.

[doccargo]: http://doc.crates.io
[doccratesio]: http://doc.crates.io/crates-io.html

Hagamos _uso_ ahora de `rand`. He aqui nuestro siguiente paso:

```rust,ignore
extern crate rand;

use std::io;
use rand::Rng;

fn main() {
    println!("Adivina el numero!");

    let numero_secreto = rand::thread_rng().gen_range(1, 101);

    println!("El numero secreto es: {}", numero_secreto);

    println!("Por favor introduce tu adivinanza.");

    let mut adivinanza = String::new();

    io::stdin().read_line(&mut adivinanza)
        .ok()
        .expect("Fallo al leer linea"));

    println!("Haz adivinado: {}", adivinanza);
}
```

La primera cosa que hemos hecho es cambiar la primera linea. Ahora dice `extern crate rand`. Debido a que declaramos `rand` en nuestra sección `[dependencies]`, podemos usar `extern crate` para hacerle saber a Rust que estaremos haciendo uso de `rand`. Esto es equivalente a un `use rand;`, de manera que podamos hacer uso de lo que sea dentro del crate `rand` a traves del prefijo `rand::`.

Después, hemos agregado otra linea `use`: `use rand::Rng`. En unos momentos estaremos haciendo uso de un metodo, y esto requiere que `Rng` este disponible para que funcione. La idea basica es la siguiente: los metodos estan dentro de algo llamado ‘traits’ (Rasgos), y para que el metodo funcione necesita que el trait este disponible. Para mayores detalles dirigete a la sección [Rasgos][traits].

[traits]: traits.html

Hay dos lineas mas en el medio: 


```rust,ignore
    let numero_secreto = rand::thread_rng().gen_range(1, 101);

    println!("El numero secreto es: {}", numero_secreto);
```

Hacemos uso de la función `rand::thread_rng()` para obtener una copia del generador de numeros aleatorios, el cual es local al [hilo][concurrency] de ejecucion en el cual estamos. Debido a que hemos hecho disponible a `rand::Rng` a traves de `use rand::Rng`, este tiene un metodo `gen_range()` disponible. Este metodo acepta dos argumentos, y genera un numero aleatorio entre estos. Es inclusivo en el limite inferior, pero es exclusivo en el limite superior, por eso necesitamos `1` y `101` para obtener un numero entre uno y cien.

[concurrency]: concurrency.html

La segunda lina solo imprime el numero secreto. Esto es util mietras desarrollamos nuestro programa , de manera tal que podamos probarlo. Estaremos elminando esta linea para la version final. No es un juego si imprime la respuesta cuando lo inicias!

Trata de ejecutar el programa unas pocas veces:

```bash
$ cargo run
   Compiling adivinanzas v0.1.0 (file:///home/tu/proyectos/adivinanzas)
     Running `target/debug/adivinanzas`
Adivina el numero!
El numero secreto es: 7
Por favor introduce tu adivinanza.
4
Haz adivinado: 4
$ cargo run
     Running `target/debug/adivinanzas`
Adivina el numero!
El numero secreto es: 83
Por favor introduce tu adivinanza.
5
Haz adivinado: 5
```

Gradioso! A continuacion: comparemos nuestra adivinanza con el numero secreto.

# Comparando adivinanzas

Ahora que tenemos entrada del usuario, comparemos la adivinanza con nuestro numero secreto. He aqui nuestro siguiente paso, aunque todavia no funciona completamente:

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Adivina el numero!");

    let numero_secreto = rand::thread_rng().gen_range(1, 101);

    println!("El numero secreto es: {}", numero_secreto);

    println!("Por favor introduce tu adivinanza.");

    let mut adivinanza = String::new();

    io::stdin().read_line(&mut adivinanza)
        .ok()
        .expect("Fallo al leer linea");

    println!("Haz adivinado: {}", adivinanza);

    match adivinanza.cmp(&numero_secreto) {
        Ordering::Less    => println!("Muy pequeño!"),
        Ordering::Greater => println!("Muy grande!"),
        Ordering::Equal   => println!("Haz ganado!"),
    }
}
```

Algunas piezas aca. La primera es otro `use`.  Hemos hecho disponible un tipo llamado `std::cmp::Ordering`. Despues, cinco nuevas lineas al fondo que lo usan:


```rust,ignore
match adivinanza.cmp(&numero_secreto) {
    Ordering::Less    => println!("Muy pequeño!"),
    Ordering::Greater => println!("Muy grande!"),
    Ordering::Equal   => println!("Haz ganado!"),
}
```

El metodo `cmp()` puede ser llamado an cualquier cosa que pueda ser comparada, este toma una referencia a la cosa con la cual quieras comparar. Retorna el tipo `Ordering` que hicimos disponible anteriormente. hemos usado una sentencia  [`match`][match] para determinar exactamente que tipo de `Ordering` es. `Ordering` es un [`enum`][enum], abreviacion para  ‘enumeration’, las cuales lucen de la siguiente manera: 

The `cmp()` method can be called on anything that can be compared, and it
takes a reference to the thing you want to compare it to. It returns the
`Ordering` type we `use`d earlier. We use a [`match`][match] statement to
determine exactly what kind of `Ordering` it is. `Ordering` is an
[`enum`][enum], short for ‘enumeration’, which looks like this:

```rust
enum Foo {
    Bar,
    Baz,
}
```

[match]: match.html
[enum]: enums.html

With this definition, anything of type `Foo` can be either a
`Foo::Bar` or a `Foo::Baz`. We use the `::` to indicate the
namespace for a particular `enum` variant.

The [`Ordering`][ordering] enum has three possible variants: `Less`, `Equal`,
and `Greater`. The `match` statement takes a value of a type, and lets you
create an ‘arm’ for each possible value. Since we have three types of
`Ordering`, we have three arms:

```rust,ignore
match guess.cmp(&secret_number) {
    Ordering::Less    => println!("Too small!"),
    Ordering::Greater => println!("Too big!"),
    Ordering::Equal   => println!("You win!"),
}
```

[ordering]: ../std/cmp/enum.Ordering.html

If it’s `Less`, we print `Too small!`, if it’s `Greater`, `Too big!`, and if
`Equal`, `You win!`. `match` is really useful, and is used often in Rust.

I did mention that this won’t quite work yet, though. Let’s try it:

```bash
$ cargo build
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
src/main.rs:28:21: 28:35 error: mismatched types:
 expected `&collections::string::String`,
    found `&_`
(expected struct `collections::string::String`,
    found integral variable) [E0308]
src/main.rs:28     match guess.cmp(&secret_number) {
                                   ^~~~~~~~~~~~~~
error: aborting due to previous error
Could not compile `guessing_game`.
```

Whew! This is a big error. The core of it is that we have ‘mismatched types’.
Rust has a strong, static type system. However, it also has type inference.
When we wrote `let guess = String::new()`, Rust was able to infer that `guess`
should be a `String`, and so it doesn’t make us write out the type. And with
our `secret_number`, there are a number of types which can have a value
between one and a hundred: `i32`, a thirty-two-bit number, or `u32`, an
unsigned thirty-two-bit number, or `i64`, a sixty-four-bit number or others.
So far, that hasn’t mattered, and so Rust defaults to an `i32`. However, here,
Rust doesn’t know how to compare the `guess` and the `secret_number`. They
need to be the same type. Ultimately, we want to convert the `String` we
read as input into a real number type, for comparison. We can do that
with three more lines. Here’s our new program:

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .ok()
        .expect("failed to read line");

    let guess: u32 = guess.trim().parse()
        .ok()
        .expect("Please type a number!");

    println!("You guessed: {}", guess);

    match guess.cmp(&secret_number) {
        Ordering::Less    => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal   => println!("You win!"),
    }
}
```

The new three lines:

```rust,ignore
    let guess: u32 = guess.trim().parse()
        .ok()
        .expect("Please type a number!");
```

Wait a minute, I thought we already had a `guess`? We do, but Rust allows us
to ‘shadow’ the previous `guess` with a new one. This is often used in this
exact situation, where `guess` starts as a `String`, but we want to convert it
to an `u32`. Shadowing lets us re-use the `guess` name, rather than forcing us
to come up with two unique names like `guess_str` and `guess`, or something
else.

We bind `guess` to an expression that looks like something we wrote earlier:

```rust,ignore
guess.trim().parse()
```

Followed by an `ok().expect()` invocation. Here, `guess` refers to the old
`guess`, the one that was a `String` with our input in it. The `trim()`
method on `String`s will eliminate any white space at the beginning and end of
our string. This is important, as we had to press the ‘return’ key to satisfy
`read_line()`. This means that if we type `5` and hit return, `guess` looks
like this: `5\n`. The `\n` represents ‘newline’, the enter key. `trim()` gets
rid of this, leaving our string with just the `5`. The [`parse()` method on
strings][parse] parses a string into some kind of number. Since it can parse a
variety of numbers, we need to give Rust a hint as to the exact type of number
we want. Hence, `let guess: u32`. The colon (`:`) after `guess` tells Rust
we’re going to annotate its type. `u32` is an unsigned, thirty-two bit
integer. Rust has [a number of built-in number types][number], but we’ve
chosen `u32`. It’s a good default choice for a small positive number.

[parse]: ../std/primitive.str.html#method.parse
[number]: primitive-types.html#numeric-types

Just like `read_line()`, our call to `parse()` could cause an error. What if
our string contained `Aߑ���? There’d be no way to convert that to a number. As
such, we’ll do the same thing we did with `read_line()`: use the `ok()` and
`expect()` methods to crash if there’s an error.

Let’s try our program out!

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
     Running `target/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

Nice! You can see I even added spaces before my guess, and it still figured
out that I guessed 76. Run the program a few times, and verify that guessing
the number works, as well as guessing a number too small.

Now we’ve got most of the game working, but we can only make one guess. Let’s
change that by adding loops!

# Looping

The `loop` keyword gives us an infinite loop. Let’s add that in:

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .ok()
            .expect("failed to read line");

        let guess: u32 = guess.trim().parse()
            .ok()
            .expect("Please type a number!");

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less    => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal   => println!("You win!"),
        }
    }
}
```

And try it out. But wait, didn’t we just add an infinite loop? Yup. Remember
our discussion about `parse()`? If we give a non-number answer, we’ll `return`
and quit. Observe:

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
     Running `target/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit
thread '<main>' panicked at 'Please type a number!'
```

Ha! `quit` actually quits. As does any other non-number input. Well, this is
suboptimal to say the least. First, let’s actually quit when you win the game:

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .ok()
            .expect("failed to read line");

        let guess: u32 = guess.trim().parse()
            .ok()
            .expect("Please type a number!");

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less    => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal   => {
                println!("You win!");
                break;
            }
        }
    }
}
```

By adding the `break` line after the `You win!`, we’ll exit the loop when we
win. Exiting the loop also means exiting the program, since it’s the last
thing in `main()`. We have just one more tweak to make: when someone inputs a
non-number, we don’t want to quit, we just want to ignore it. We can do that
like this:

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .ok()
            .expect("failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less    => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal   => {
                println!("You win!");
                break;
            }
        }
    }
}
```

These are the lines that changed:

```rust,ignore
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue,
};
```

This is how you generally move from ‘crash on error’ to ‘actually handle the
error’, by switching from `ok().expect()` to a `match` statement. The `Result`
returned by `parse()` is an enum just like `Ordering`, but in this case, each
variant has some data associated with it: `Ok` is a success, and `Err` is a
failure. Each contains more information: the successful parsed integer, or an
error type. In this case, we `match` on `Ok(num)`, which sets the inner value
of the `Ok` to the name `num`, and then we just return it on the right-hand
side. In the `Err` case, we don’t care what kind of error it is, so we just
use `_` instead of a name. This ignores the error, and `continue` causes us
to go to the next iteration of the `loop`.

Now we should be good! Let’s try:

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
     Running `target/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

Awesome! With one tiny last tweak, we have finished the guessing game. Can you
think of what it is? That’s right, we don’t want to print out the secret
number. It was good for testing, but it kind of ruins the game. Here’s our
final source:

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .ok()
            .expect("failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less    => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal   => {
                println!("You win!");
                break;
            }
        }
    }
}
```

# Complete!

At this point, you have successfully built the Guessing Game! Congratulations!

This first project showed you a lot: `let`, `match`, methods, associated
functions, using external crates, and more. Our next project will show off
even more.

