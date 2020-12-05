% Juego de Adivinanzas

Para nuestro primer proyecto, implementaremos un problema clásico de programación para principiantes: un juego de 
adivinanzas. ¿Cómo funciona el juego? Nuestro programa generará un número entero aleatorio entre uno y cien. Nos pedirá 
que introduzcamos una corazonada. Después de haber proporcionado nuestro número, éste nos dirá si estuvimos muy por 
debajo y muy por encima. Una vez que adivinemos el número correcto, nos felicitará. ¿Suena bien?

# Configuración Inicial

Creemos un nuevo proyecto. Anda a tu directorio de proyectos. ¿Recuerdas cómo creamos nuestra estructura de directorios 
y un `Cargo.toml` para `hola_mundo`? Cargo posse un comando que hace eso por nosotros. Probémoslo:

```bash
$ cd ~/proyectos
$ cargo new adivinanzas --bin
$ cd adivinanzas
```

Pasamos el nombre de nuestro proyecto a `cargo new`, junto con el flag `--bin`, debido a que estamos creando un binario, 
en vez de una biblioteca.

Echa un vistazo al `Cargo.toml` generado:

```toml
[package]

name = "adivinanzas"
version = "0.1.0"
authors = ["Tu Nombre <tu@ejemplo.com>"]
```

Cargo obtiene esta información  de tu entorno. Si no está correcta, corrígela.

Finalmente, Cargo ha generado un ‘¡Hola, mundo!’ para nosotros. Echa un vistazo a `src/main.rs`:

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

¡Excelente! Abre tu `src/main.rs` otra vez. Estaremos escribiendo todo nuestro código en este archivo.

Antes de continuar, dejame mostrarte un comando más de Cargo: `run`. `cargo run` es una especie de `cargo build`, pero 
con la diferencia de que también ejecuta el binario producido. Probemoslo:

```bash
$ cargo run
   Compiling adivinanzas v0.1.0 (file:///home/tu/proyectos/adivinanzas)
     Running `target/debug/adivinanzas`
Hola, mundo!
```

¡Grandioso! El comando `run` es muy útil cuando necesitas iterar rápido en un proyecto. Nuestro juego es uno de esos 
proyectos, necesitaremos probar rapidamente cada iteración antes de movernos a la siguiente.

# Procesando un Intento de Adivinanza

¡Probemoslo! La primera cosa que necesitamos hacer para nuestro juego de adivinanzas es permitir a nuestro jugador 
ingresar un intento de adivinanza. Coloca esto en tu `src/main.rs`:

```rust,no_run
use std::io;

fn main() {
    println!("¡Adivina el número!");

    println!("Por favor introduce tu corazonada:");

    let mut corazonada = String::new();

    io::stdin().read_line(&mut corazonada)
        .ok()
        .expect("Fallo al leer línea");

    println!("Tu corazonada fue: {}", corazonada);
}
```

¡Hay un monton aquí! Tratemos de ir a través de ello, pieza por pieza.

```rust,ignore
use std::io;
```

Necesitaremos recibir entrada del usuario, para luego imprimir el resultado como salida. Debido a esto necesitamos 
la biblioteca `io` de la biblioteca estándar. Rust sólo importa unas pocas cosas para todos los programas, este conjunto 
de cosas se denomina [‘preludio’][prelude]. Si no esta en el preludio tendrás que llamarlo directamente a través de `use`.

[prelude]: ../std/prelude/index.html

```rust,ignore
fn main() {
```

Como has visto con anterioridad, la función `main()` es el punto de entrada a tu programa. La sintaxis `fn` declara una 
nueva función, los `()`s indican que no hay argumentos y  `{` comienza el cuerpo de la función. Debido a que no incluimos 
un tipo de retorno, se asume ser `()` una [tupla][tuples] vacía.

[tuples]: primitive-types.html#tuples

```rust,ignore
    println!("¡Adivina el número!");

    println!("Por favor introduce tu corazonada:");
```

Anteriormente aprendimos que `println!()` es una [macro][macros] que imprime una [cadena de caracteres][strings] a la pantalla.

[macros]: macros.html
[strings]: strings.html

```rust,ignore
    let mut corazonada = String::new();
```

¡Ahora nos estamos poniendo interesantes! Hay un montón de cosas pasando en esta pequeña línea. La primera cosas a notar 
es que es una [sentencia let][let], usada para crear variables. Tiene la forma:

```rust,ignore
let foo = bar;
```

[let]: variable-bindings.html

Esto creará una nueva variable llamada `foo` y la enlazará al valor `bar`. En muchos lenguajes, esto es llamado una 
‘variable’ pero las variables de Rust tienen un par de trucos bajo la manga.

Por ejemplo, son [immutables][immutable] por defecto. Es por ello que nuestro ejemplo usa `mut`: esto hace un binding 
mutable, en vez de inmutable. `let` no sólo toma un nombre del lado izquierdo, `let` acepta un ‘[patrón][patterns]’. 
Usaremos los patrones un poco más tarde. Es suficiente por ahora usar:

```rust
let foo = 5; // inmutable
let mut bar = 5; // mutable
```

[immutable]: mutability.html
[patterns]: patterns.html

Ah, `//` inicia un comentario, hasta el final de la línea. Rust ignora todo en [comentarios][comments]

[comments]: comments.html

Entonces sabemos que `let mut corazonada` introducirá un binding mutable llamado `corazonada`, pero tenemos que ver al 
otro lado del `=` para saber a que esta siendo asociado: `String::new()`.

`String` es un tipo de cadena de caracter, proporcionado por la biblioteca estándar. Un [`String`][string] es un 
segmento de texto codificado en UTF-8 capaz de crecer.

[string]: ../std/string/struct.String.html

La sintaxis `::new()` usa `::` porque es una ‘función asociada’ de un tipo particular. Es decir esta asociada con `String` 
en sí mismo, en vez de con una instacia en particular de `String`. Algunos lenguajes llaman a esto un ‘método estático’.

Esta función es llamada `new()`, porque crea un nuevo `String` vacío. Encontrarás una función `new()` en muchos tipos, 
debido a que es un nombre común para la creación de un nuevo valor de algún tipo.

Continuemos:

```rust,ignore
    io::stdin().read_line(&mut corazonada)
        .ok()
        .expect("Fallo lectura de línea");
```

¡Otro montón! Vayamos pieza por pieza. La primera línea tiene dos partes. He aquí la primera:

```rust,ignore
io::stdin()
```

¿Recuerdas cómo usamos `use` en `std::io` en la primera línea de nuestro programa? Ahora estamos llamando una función 
asociada en `std::io`. De no haber usado `use std::io`, pudimos haber escrito esta línea como `std::io::stdin()`.

Esta función en particular retorna un handle a la entrada estándar de tu terminal. Mas especificamente, un [std::io::Stdin][iostdin].

[iostdin]: ../std/io/struct.Stdin.html

La siguiente parte usará dicho handle para obtener entrada del usuario:

```rust,ignore
.read_line(&mut corazonada)
```

Aquí, llamamos al método [`read_line()`][read_line] en nuestro handle. Los [métodos][method] son similares a las funciones 
asociadas, pero sólo estan disponibles en una instancia en particular de un tipo, en vez de en el tipo en sí. También 
estamos pasando un argumento a `read_line()`: `&mut corazonada`.

[read_line]: ../std/io/struct.Stdin.html#method.read_line
[method]: method-syntax.html

¿Recuerdas cuando creamos `corazonada`? Dijimos que era mutable. Sin embargo `read_line` no acepta un `String` como 
argumento: acepta un `&mut String`. Rust posee una característica llamada ‘[referencias][references]’ (‘references’), 
la cual permite tener múltiples referencias a una pieza de data, de esta manera se reduce la necesidad de copiado. Las 
referencias son una característica compleja, debido a que uno de los puntos de venta más fuertes de Rust es acerca de 
cuán fácil y seguro es usar referencias. Por ahora no necesitamos saber mucho de esos detalles para finalizar nuestro 
programa. Todo lo que necesitamos saber por el momento es que al igual que los bindings `let`, las referencias son 
inmutables por defecto. Como consecuencia necesitamos escribir `&mut corazonada` en vez de `&corazonada`.

Porque `read_line()` acepta una referencia mutable a una cadena de caracteres, su trabajo es tomar lo que el usuario 
ingresa en la entrada estándar y colocarlo en una cadena de caracteres. Por esta razón toma dicha cadena como argumento y 
debido a que debe de agregar la entrada del usuario, éste necesita ser mutable.

[references]: references-and-borrowing.html

Todavía no hemos terminado con esta línea. Si bien es una sola línea de texto, es sólo la primera parte de una línea 
lógica de código completa:

```rust,ignore
        .ok()
        .expect("Fallo lectura de línea");
```

Cuando llamas a un método con la sintaxis `.foo()` puedes introducir un salto de línea y otro espacio. Esto te ayuda a 
dividir líneas largas. Pudimos haber escrito:

```rust,ignore
    io::stdin().read_line(&mut corazonada).ok().expect("Fallo lectura de línea");
```

Pero eso es más difícil de leer. Así que lo hemos dividido en tres líneas para tres llamadas a métodos. Ya hemos 
hablado de `read_line()`, ¿pero qué acerca de `ok()` y `expect()`? Bueno, ya mencionamos que `read_line()` coloca la 
entrada del usuario en el `&mut String` que le proprocionamos. Pero también retorna un valor: en este caso un [`io::Result`][ioresult]. 
Rust posee un número de tipos llamados `Result` en su biblioteca estándar: un [`Result`][result] genérico y versiones 
específicas para sub-bibliotecas, como `io::Result`.

[ioresult]: ../std/io/type.Result.html
[result]: ../std/result/enum.Result.html


El proposito de esos `Result` es codificar información de manejo de errores. Valores del tipo `Result` tienen métodos 
definidos en ellos. En este caso `io::Result` posee un método `ok()`, que se traduce en ‘queremos asumir que este valor 
es un valor exitoso. Si no, descarta la información acerca del error’. ¿Por qué descartar la información acerca del error? 
Para un programa básico, queremos simplemente imprimir un error genérico, cualquier problema que signifique que no 
podemos continuar. El [método `ok()`][ok] retorna un valor que tiene otro método definido en él: `expect()`. El 
[`método expect()`][expect] toma el valor en el cual es llamado y si no es un valor exitoso, hace pánico [`panic!`][panic] 
con un mensaje que le hemos proporcionado. Un `panic!` como este causará que el programa tenga una salida abrupta (crash), 
mostrando dicho mensaje.

[ok]: ../std/result/enum.Result.html#method.ok
[expect]: ../std/option/enum.Option.html#method.expect
[panic]: error-handling.html

Si quitamos las llamadas a esos dos métodos, nuestro programa compilará pero obtendremos una advertencia:

```bash
$ cargo build
  Compiling adivinanzas v0.1.0 (file:///home/tu/proyectos/adivinanzas)
src/main.rs:10:5: 10:44 warning: unused result which must be used, #[warn(unused_must_use)] on by default
src/main.rs:10     io::stdin().read_line(&mut corazonada);
                   ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

Rust nos advierte que no hemos usado el valor `Result`. Esta advertencia viene de una anotación especial que tiene 
`io::Result`. Rust esta tratando de decirte que no has manejado un posible error. La manera correcta de suprimir el 
error es, en efecto, escribir el código para el manejo de erroes. Por suerte, si sólo queremos terminar la ejecución 
del programa en caso de haber un problema, podemos usar estos dos pequeños métodos. Si pudiéramos recuperarnos del 
error de alguna manera, haríamos algo diferente, pero dejemos eso para un proyecto futuro.

Sólo nos queda una línea de este primer ejemplo:

```rust,ignore
    println!("Tu corazonada fue: {}", corazonada);
}
```

Esta línea imprime la cadena de caracteres en la que guardamos nuestra entrada. Los `{}`s son marcadores de posición, 
es por ello que pasamos `corazonada` como argumento.  De haber habido múltiples `{}`s, debiamos haber pasado múltiples argumentos:

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
¡Adivina el número!
Por favor introduce tu corazonada:
6
Tu corazonada fue: 6
```

¡Enhorabuena! Nuestra primera parte ha terminado: podemos obtener entrada del teclado e imprimirla de vuelta.

# Generando un número secreto

A continuación necesitamos generar un número secreto. Rust todavía no incluye una funcionalidad de números aleatorios en 
la biblioteca estándar. Sin embargo, el equipo de Rust provee un [crate `rand`][randcrate]. Un ‘crate’ es un paquete de 
código Rust. Nosotros hemos estado construyendo un ‘crate binaro’, el cual es un ejecutable. `rand` es un ‘crate biblioteca’, 
que contiene código a ser usado por otros programas.

[randcrate]: https://crates.io/crates/rand

Usando crates externos es donde Cargo realmente brilla. Antes de que podamos escribir código que haga uso de `rand`, 
debemos modificar nuestro archivo `Cargo.toml`. Ábrelo y agrega estas líneas al final:

```toml
[dependencies]

rand="0.3.0"
```

La sección `[dependencies]` de `Cargo.toml` es como la sección `[package]`: todo lo que le sigue es parte de ella, hasta 
que la siguiente sección comience. Cargo usa la sección de dependencias para saber de cuáles crates externos dependemos, 
así como las versiones requeridas. En este caso hemos usado la versión `0.3.0`. Cargo entiende [Versionado Semantico][semver], 
que es un estándar para las escritura de números de versión. Si quisieramos usar la ultima versión podríamos haber 
usado `*` o un rango de versiones. La [documentación de Cargo][cargodoc] contiene más detalles.

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

(Podrías ver versiones diferentes, por supuesto.)

¡Un montón de salida más! Ahora que tenemos una dependencia externa, Cargo descarga del registro las ultimas versiones 
de todo, lo cual puede copiar datos desde [Crates.io][cratesio]. Crates.io es donde las personas del ecosistema Rust 
publican sus proyectos open source para que otros los usen.


[cratesio]: https://crates.io

Después de actualizar el registro, Cargo chequea nuestras dependencias (en `[dependencies]`) y las descarga de no 
tenerlas todavía. En este caso sólo dijímos que queríamos depender de `rand` y también obtuvimos una copia de `libc`. 
Esto es debido a que `rand` depende a su vez de `libc` para funcionar. Después de descargar las dependencias, Cargo las 
compila, para después compilar nuestro código.

Si ejecutamos  `cargo build`, obtendremos una salida diferente:


```bash
$ cargo build
```

Así es, ¡no hay salida! Cargo sabe que nuestro proyecto ha sido construido, así como todas sus dependencias, por lo que 
no nay razón para hacer todo el proceso otra vez. Sin nada que hacer, simplemente termina la ejecución. Si abrimos 
`src/main.rs` otra vez, hacemos un cambio trivial y guardamos los cambios, solamente veríamos una línea:


```bash
$ cargo build
    Compiling adivinanzas v0.1.0 (file:///home/tu/proyectos/adivinanzas)
```

Entonces, le hemos dicho a Cargo que queríamos cualquier versión `0.3.x` de `rand` y éste descargo la última versión 
para el momento de la escritura de este tutorial, `v0.3.8`. Pero ¿qué pasa cuando la versión `v0.3.9` sea publicada con 
un importante bugfix? Si bien recibir bugfixes es importante, ¿qué pasa si `0.3.9` contiene una regresión que rompe nuestro 
código?

La respuesta a este problema es el archivo `Cargo.lock`, archivo que encontrarás en tu directorio de proyecto. Cuando 
construyes tu proyecto por primera vez, cargo determina todas las versiones que coinciden con tus criterios y las escribe 
en el archivo `Cargo.lock`. Cuando construyes tu proyecto en el futuro, Cargo notará que un archivo `Cargo.lock` existe
y usará las versiones especificadas en el mismo, en vez de hacer todo el trabajo de determinar las versiones otra vez. 
Esto te permite tener una construcción reproducible de manera automática. En otras palabras, nos quedaremos en `0.3.8` 
hasta que subamos de versión de manera explicita, de igual manera lo hará la gente con la que hemos compartido nuestro 
código, gracias al archivo `Cargo.lock`.

Pero ¿qué pasa cuando _queremos_ usar `v0.3.9`? Cargo posee otro comando, `update`, que se traduce en ‘ignora el bloqueo 
y determina todas las últimas versiones que coincidan con lo que hemos especficado. De funcionar esto, escribe esas 
versiones al archivo de bloqueo `Cargo.lock`’. Pero, por defecto, Cargo sólo buscara versiones mayores a `0.3.0`
y menores a `0.4.0`. Si queremos movernos a `0.4.x`, necesitaríamos actualizar el archivo `Cargo.toml` directamente. 
Cuando lo hacemos, la siguente vez que ejecutemos `cargo build`, Cargo actualizará el índice y re-evaluará nuestros 
requerimentos acerca de `rand`.

Hay mucho más que decir acerca de [Cargo][doccargo] y [su ecosistema][doccratesio], pero por ahora, eso es todo lo que 
necesitamos saber. Cargo hace realmente fácil reusar bibliotecas y los Rusteros tienden a escribir proyectos pequeños,
los cuales están construidos por un conjunto de paquetes más pequeños.

[doccargo]: http://doc.crates.io
[doccratesio]: http://doc.crates.io/crates-io.html

Hagamos _uso_ ahora de `rand`. He aquí nuestro siguiente paso:

```rust,ignore
extern crate rand;

use std::io;
use rand::Rng;

fn main() {
    println!("¡Adivina el número!");

    let numero_secreto = rand::thread_rng().gen_range(1, 101);

    println!("El número secreto es: {}", numero_secreto);

    println!("Por favor introduce tu corazonada:");

    let mut corazonada = String::new();

    io::stdin().read_line(&mut corazonada)
        .ok()
        .expect("Fallo al leer línea");

    println!("Tu corazonada fue: {}", corazonada);
}
```

La primera cosa que hemos hecho es cambiar la primera línea. Ahora dice `extern crate rand`. Debido a que declaramos 
`rand` en nuestra sección `[dependencies]`, podemos usar `extern crate` para hacerle saber a Rust que estaremos haciendo 
uso de `rand`. Esto es equivalente a un `use rand;`, de manera que podamos hacer uso de lo que sea dentro del crate 
`rand` a través del prefijo `rand::`.

Después, hemos agregado otra línea `use`: `use rand::Rng`. En unos momentos estaremos haciendo uso de un método y esto 
requiere que `Rng` este disponible para que funcione. La idea básica es la siguiente: los métodos estan dentro de algo 
llamado ‘traits’ (Rasgos) y para que el método funcione, necesita que el trait esté disponible. Para mayores detalles 
dirígete a la sección [Rasgos][traits] (Traits).

[traits]: traits.html

Hay dos líneas más en el medio:


```rust,ignore
    let numero_secreto = rand::thread_rng().gen_range(1, 101);

    println!("El número secreto es: {}", numero_secreto);
```

Hacemos uso de la función `rand::thread_rng()` para obtener una copia del generador de números aleatorios, el cual es 
local al [hilo][concurrency] de ejecución en el cual estamos. Debido a que hemos hecho disponible a `rand::Rng` a través 
de `use rand::Rng`, este tiene un método `gen_range()` disponible. Este método acepta dos argumentos y genera un número 
aleatorio entre estos. Es inclusivo en el limite inferior pero es exclusivo en el limite superior, por eso necesitamos 
`1` y `101` para obtener un número entre uno y cien.

[concurrency]: concurrency.html

La segunda línea sólo imprime el número secreto. Esto es útil mietras desarrollamos nuestro programa, de manera tal que 
podamos probarlo. Estaremos eliminando esta línea para la versión final. ¡No es un juego si imprime la respuesta justo 
cuando lo inicias!

Trata de ejecutar el programa unas pocas veces:

```bash
$ cargo run
   Compiling adivinanzas v0.1.0 (file:///home/tu/proyectos/adivinanzas)
     Running `target/debug/adivinanzas`
¡Adivina el número!
El número secreto es: 7
Por favor introduce tu corazonada:
4
Tu corazonada fue: 4
$ cargo run
     Running `target/debug/adivinanzas`
¡Adivina el número!
El número secreto es: 83
Por favor introduce tu corazonada:
5
Tu corazonada fue: 5
```

¡Gradioso! A continuación: comparemos nuestra adivinanza con el número secreto.

# Comparando adivinanzas

Ahora que tenemos entrada del usuario, comparemos la adivinanza con nuestro número secreto. He aquí nuestro siguiente 
paso, aunque todavia no funciona completamente:

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("¡Adivina el número!");

    let numero_secreto = rand::thread_rng().gen_range(1, 101);

    println!("El número secreto es: {}", numero_secreto);

    println!("Por favor introduce tu corazonada:");

    let mut corazonada = String::new();

    io::stdin().read_line(&mut corazonada)
        .ok()
        .expect("Fallo al leer línea");

    println!("Tu corazonada fue: {}", corazonada);

    match corazonada.cmp(&numero_secreto) {
        Ordering::Less    => println!("¡Muy pequeño!"),
        Ordering::Greater => println!("¡Muy grande!"),
        Ordering::Equal   => println!("¡Haz ganado!"),
    }
}
```

Algunas piezas acá. La primera es otro `use`.  Hemos hecho disponible un tipo llamado `std::cmp::Ordering`. Después, 
cinco nuevas líneas al fondo que lo usan:


```rust,ignore
match corazonada.cmp(&numero_secreto) {
    Ordering::Less    => println!("¡Muy pequeño!"),
    Ordering::Greater => println!("¡Muy grande!"),
    Ordering::Equal   => println!("¡Haz ganado!"),
}
```

El método `cmp()` puede ser llamado en cualquier cosa que pueda ser comparada, este toma una referencia a la cosa con 
la cual quieras comparar. Retorna el tipo `Ordering` que hicimos disponible anteriormente. Hemos usado una sentencia 
[`match`][match] para determinar exactamente qué tipo de `Ordering` es. `Ordering` es un [`enum`][enum], abreviación para 
‘enumeration’, las cuales lucen de la siguiente manera:


```rust
enum Foo {
    Bar,
    Baz,
}
```

[match]: match.html
[enum]: enums.html


Con esta definición, cualquier cosa de tipo `Foo` puede ser bien sea un `Foo::Bar` o un `Foo::Baz`. Usamos el `::` para 
indicar el espacio de nombres para una variante `enum` en particular.

La enum [`Ordering`][ordering] tiene tres posibles variantes:  `Less`, `Equal`,
and `Greater` (menor, igual y mayor respectivamente). La sentencia `match` toma un valor de un tipo y te permite crear 
un ‘brazo’ para cada valor posible. Debido a que tenemos tres posibles tipos de `Ordering`, tenemos tres brazos:


```rust,ignore
match guess.cmp(&secret_number) {
    Ordering::Less    => println!("¡Muy pequeño!"),
    Ordering::Greater => println!("¡Muy grande!"),
    Ordering::Equal   => println!("¡Haz ganado!"),
}
```

[ordering]: ../std/cmp/enum.Ordering.html

Si es `Less`, imprimimos `¡Muy pequeño!`, si es `Greater`, `¡Muy grande!` y si es
`Equal`, `¡Haz ganado!`. `match` es realmente útil y es usado con frecuencia en Rust.

Anteriormente mencione que todavia no funciona. Pongamoslo a prueba:


```bash
$ cargo build
   Compiling adivinanzas v0.1.0 (file:///home/tu/proyectos/adivinanzas)
src/main.rs:28:21: 28:35 error: mismatched types:
 expected `&collections::string::String`,
    found `&_`
(expected struct `collections::string::String`,
    found integral variable) [E0308]
src/main.rs:28     match corazonada.cmp(&numero_secreto) {
                                   ^~~~~~~~~~~~~~
error: aborting due to previous error
Could not compile `adivinanzas`.
```

¡Oops! Un gran error. Lo principal en él es que tenemos ‘tipos incompatibles’ (‘mismatched types’). Rust posee un fuerte 
sistema de tipos estático. Sin embargo, también tiene inferencia de tipos. Cuando escribimos `let corazonada = String::new()`, 
Rust fue capaz de inferir que `corazonada` debia ser un `String` y por ello no nos hizo escribir el tipo. Con nuestro 
`numero_secreto` hay un número de tipos que pueden tener un valor entre uno y cien: `i32`, un número de treinta y dos bits, 
`u32`, un número sin signo de treinta y dos bits o `i64` un número de sesenta y cuatro bits u otros. Hasta ahora, eso no 
ha importado debido a que Rust por defecto usa `i32`. Sin embargo, en este caso, Rust no sabe cómo comparar `corazonada` 
con `numero_secreto`. Ambos necesitan ser del mismo tipo. Al final, queremos convertir el `String` que leímos como entrada 
en un tipo real de número, para efectos de la comparación. Podemos hacer eso con tres líneas más. He aquí nuestro nuevo programa:

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("¡Adivina el número!");

    let numero_secreto = rand::thread_rng().gen_range(1, 101);

    println!("El número secreto es: {}", numero_secreto);

    println!("Por favor introduce tu corazonada:");

    let mut corazonada = String::new();

    io::stdin().read_line(&mut corazonada)
        .ok()
        .expect("Fallo al leer línea");

    let corazonada: u32 = corazonada.trim().parse()
        .ok()
        .expect("¡Por favor introduce un número!");

    println!("Tu corazonada fue: {}", corazonada);

    match corazonada.cmp(&numero_secreto) {
        Ordering::Less    => println!("¡Muy pequeño!"),
        Ordering::Greater => println!("¡Muy grande!"),
        Ordering::Equal   => println!("¡Haz ganado!"),
    }
}
```

Las tres nuevas líneas:

```rust,ignore
    let corazonada: u32 = corazonada.trim().parse()
        .ok()
        .expect("Por favor introduce un número!");
```

Espera un momento, pensé que ya teniamos una `corazonada`. Y la tenemos, pero Rust nos permite sobreescribir
la `corazonada` previa con una nueva (‘shadowing’). Esto es usado con frecuencia en esta misma situación, en donde 
`corazonada` es un `String` pero queremos convertirlo a un `u32`. Este shadowing nos permite reusar el nombre 
`corazonada` en vez de forzarnos a idear dos nombres únicos como `corazonada_str` y `corazonada`, u otros.

Estamos asociando `corazonada` a una expresión que luce como algo que escribimos anteriormente:

```rust,ignore
guess.trim().parse()
```

Seguido por una invocación a `ok().expect()`. Aquí `corazonada` hace referencia a la vieja versión, la que era un `String` 
que contenía nuestra entrada de usuario en ella. El método `trim()` en los `String`s elimina cualquier espacio en blanco 
al principio y al final de nuestras cadenas de caracteres. Esto es importante debido a que tuvimos que presionar la tecla 
‘retorno’ para satisfacer a `read_line()`. Esto significa que si escribimos `5` y presionamos ‘retorno’ `corazonada` 
luce así: `5\n`. El `\n` representa ‘nueva línea’ (‘newline’), la tecla enter. `trim()` se deshace de esto, dejando 
nuestra cadena de caracteres sólo con el `5`. El [método `parse()` en las cadenas caracteres][parse] parsea una cadena 
de caracteres en algún tipo de número. Debido a que puede parsear una variedad de números, debemos darle a Rust una 
pista del tipo exacto de número que deseamos. De ahí la parte `let corazonada: u32`. Los dos puntos (`:`)  después de 
`corazonada` le dicen a Rust que vamos a anotar el tipo. `u32` es un entero sin signo de treinta y dos bits. Rust posee 
[una variedad de tipos número integrados][number], pero nosotros hemos escojido `u32`. Es una buena opción por defecto 
para un número positivo pequeño.

[parse]: ../std/primitive.str.html#method.parse
[number]: primitive-types.html#numeric-types

Al igual que `read_line()`, nuestra llamada a `parse()` podría causar un error. ¿Qué tal si nuestra cadena de caracteres 
contiene `Aߑ���?` No habría  forma de convertir eso en un número. Es por ello que haremos lo mismo que hicimos con `read_line()`: 
usar los métodos `ok()` y `expect()` para terminar abruptamente si hay algún error.


¡Probemos nuestro programa!

```bash
$ cargo run
   Compiling adivinanzas v0.1.0 (file:///home/tu/proyectos/adivinanzas)
     Running `target/adivinanzas`
¡Adivina el número!
El número secreto es: 58
Por favor introduce tu corazonada:
  76
Tu corazonada fue: 76
¡Muy grande!
```

¡Excelente! Puedes ver que incluso he agregado espacios antes de mi intento y aún así el programa determinó que intente 76. 
Ejecuta el programa unas pocas veces y verifica que adivinar el número funciona, así como intentar un número muy pequeño.

Ahora tenemos la mayoria del juego funcionando, pero sólo podemos intentar adivinar una vez. ¡Tratemos de cambiar eso 
agregando ciclos!

# Iteración

La palabra clave `loop` nos proporciona un ciclo infinito. Agreguémosla:

¡Adivina el número!
El número secreto es: 58
Por favor introduce tu adivinanza.
  76
Tu corazonada fue: 76
¡Muy grande!

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("¡Adivina el número!");

    let numero_secreto = rand::thread_rng().gen_range(1, 101);

    println!("El número secreto es: {}", numero_secreto);

    loop {
        println!("Por favor introduce tu corazonada:");

        let mut corazonada = String::new();

        io::stdin().read_line(&mut corazonada)
            .ok()
            .expect("Fallo al leer línea");

        let corazonada: u32 = corazonada.trim().parse()
            .ok()
            .expect("Por favor introduce un número!");

        println!("Haz corazonada: {}", corazonada);

        match corazonada.cmp(&numero_secreto) {
            Ordering::Less    => println!("¡Muy pequeño!"),
            Ordering::Greater => println!("¡Muy grande!"),
            Ordering::Equal   => println!("¡Haz ganado!"),
        }
    }
}
```

Pruébalo. Pero espera, ¿no acabamos de agregar un ciclo infinito? Sip. ¿Recuerdas nuestra discusión acerca de `parse()`? 
Si damos una respuesta no numérica, retornaremos (`return`) y finalizaremos la ejecución. Observa:

```bash
$ cargo run
   Compiling adivinanzas v0.1.0 (file:///home/tu/proyectos/adivinanzas)
     Running `target/adivinanzas`
¡Adivina el número!
El número secreto es: 59
Por favor introduce tu corazonada:
45
Tu corazonada fue: 45
¡Muy pequeño!
Por favor introduce tu corazonada:
60
Tu corazonada fue: 60
¡Muy grande!
Por favor introduce tu corazonada:
59
Tu corazonada fue: 59
¡Haz ganado!
Por favor introduce tu corazonada:
quit
thread '<main>' panicked at 'Please type a number!'
```

¡Ja! `quit` en efecto termina la ejecución. Así como cualquier otra entrada que no sea un número. Bueno, esto es subóptimo 
por decir lo menos. Primero salgamos cuando ganemos:

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("¡Adivina el número!");

    let numero_secreto = rand::thread_rng().gen_range(1, 101);

    println!("El número secreto es: {}", numero_secreto);

    loop {
        println!("Por favor introduce tu corazonada:");

        let mut corazonada = String::new();

        io::stdin().read_line(&mut corazonada)
            .ok()
            .expect("Fallo al leer línea");

        let corazonada: u32 = corazonada.trim().parse()
            .ok()
            .expect("Por favor introduce un número!");

        println!("Tu corazonada fue: {}", corazonada);

        match corazonada.cmp(&numero_secreto) {
            Ordering::Less    => println!("¡Muy pequeño!"),
            Ordering::Greater => println!("¡Muy grande!"),
            Ordering::Equal   => {
                println!("¡Haz ganado!");
                break;
            }
        }
    }
}
```

Al agregar la línea `break` después del "¡Haz ganado!", romperemos el ciclo cuando ganemos. Salir del ciclo también significa 
salir del programa, debido a que es la ultima cosa en `main()`. Sólo nos queda una sola mejora por hacer: cuando alguien 
introduzca un valor no numérico, no queremos terminar la ejecución, queremos simplemente ignorarlo. Podemos hacerlo de la 
siguiente manera:


```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("¡Adivina el número!");

    let numero_secreto = rand::thread_rng().gen_range(1, 101);

    println!("El número secreto es: {}", numero_secreto);

    loop {
        println!("Por favor introduce tu corazonada:");

        let mut corazonada = String::new();

        io::stdin().read_line(&mut corazonada)
            .ok()
            .expect("Fallo al leer línea");

        let corazonada: u32 = match corazonada.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("Tu corazonada fue: {}", corazonada);

        match corazonada.cmp(&numero_secreto) {
            Ordering::Less    => println!("¡Muy pequeño!"),
            Ordering::Greater => println!("¡Muy grande!"),
            Ordering::Equal   => {
                println!("¡Haz ganado!");
                break;
            }
        }
    }
}
```

Estas son las líneas que han cambiado:

```rust,ignore
let corazonada: u32 = match corazonada.trim().parse() {
    Ok(num) => num,
    Err(_) => continue,
};
```

Es así como pasamos de ‘terminar abruptamente en un error’ a ‘efectivamente manejar el error’, a través del cambio de 
`ok().expect()` a una sentencia `match`. El `Result` retornado por `parse()` es un enum justo como `Ordering`, pero en 
éste caso cada variante tiene datos asociados: `Ok` es éxito y `Err` es una falla. Cada uno contiene más información: 
el entero parseado en el caso exitoso o un tipo de error en el caso de fallo. En este caso hacemos `match` en`Ok(num)`, 
el cual asigna el valor interno del `Ok` al nombre `num` y seguidamente retorna en el lado derecho. En el caso de `Err`, 
no nos importa qué tipo de error es, por ello usamos `_` en lugar de un nombre. Esto ignora el error y `continue` nos 
mueve a la siguiente iteración del ciclo (`loop`).

¡Ahora deberíamos estar bien! Probemos:

```bash
$ cargo run
   Compiling adivinanzas v0.1.0 (file:///home/tu/proyectos/adivinanzas)
     Running `target/adivinanzas`
¡Adivina el número!
El número secreto es: 61
Por favor introduce tu corazonada:
10
Tu corazonada fue: 10
¡Muy pequeño!
Por favor introduce tu corazonada:
99
Tu corazonada fue: 99
¡Muy pequeño!
Por favor introduce tu corazonada:
foo
Por favor introduce tu corazonada:
61
Tu corazonada fue: 61
¡Haz ganado!
```

¡Genial! Con una última mejora finalizamos el juego de las advinanzas. ¿Te imaginas cuál es? Es correcto, no queremos 
imprimir el número secreto. Era bueno para las pruebas, pero arruina nuestro juego. He aquí nuestro código fuente final:

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("¡Adivina el número!");

    let numero_secreto = rand::thread_rng().gen_range(1, 101);

    loop {
        println!("Por favor introduce tu corazonada:");

        let mut corazonada = String::new();

        io::stdin().read_line(&mut corazonada)
            .ok()
            .expect("Fallo al leer línea");

        let corazonada: u32 = match corazonada.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("Tu corazonada fue: {}", corazonada);

        match corazonada.cmp(&numero_secreto) {
            Ordering::Less    => println!("¡Muy pequeño!"),
            Ordering::Greater => println!("¡Muy grande!"),
            Ordering::Equal   => {
                println!("¡Haz ganado!");
                break;
            }
        }
    }
}
```

# ¡Completado!

En éste punto has terminado satisfactoriamente el juego de las adivinanza, ¡Felicitaciones!

Éste primer proyecto te enseñó un montón: `let`, `match`, métodos, funciones asociadas, usar crates externos y más. 
Nuestro siguiente proyecto demostrara aún más.
