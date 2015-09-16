% Tiempos de Vida

Esta guía es una de las tres presentando el sistema de pertenencia de Rust. Esta es una de las características mas únicas y atractivas de Rust, con la que los desarrolladores Rust deben estar bien familiarizados. La pertenencia es como Rust logra su objetivo mayor, seguridad en el manejo de memoria. Existen unos pocos conceptos distintos, cada uno con su propio capitulo:

* [pertenencia][ownership], el concepto principal
* [prestamo][borrowing], y sus caracteristica asociada ‘referencias’
* [tiempos de vida][lifetimes], la que lees ahora

Estos tres capítulos están relacionados, y en orden. Necesitaras leer los tres para entender completamente el sistema de pertenencia.

[ownership]: ownership.html
[lifetimes]: lifetimes.html

# Meta

Antes de entrar en detalle, dos notas importantes acerca del sistema de pertenencia.

Rust tiene foco en seguridad y velocidad. Rust logra esos objetivos a travez de muchas ‘abstracciones de cero costo’, lo que significa que en Rust, las abstracciones cuestan tan poco como sea posible para hacerlas funcionar. El sistema de pertenencia es un ejemplo primordial de una abstracción de cero costo. Todo el análisis del que estaremos hablando en la presente guía es _llevado a cabo en tiempo de compilación_. No pagas ningún costo en tiempo de ejecución por ninguna de estas facilidades.

Sin embargo, este sistema tiene cierto costo: la curva de aprendizaje. Muchos usuarios nuevos Rust experimentan algo que nosotros denominamos ‘pelear con el comprobador de préstamo’ (‘fighting with the borrow checker’), situación en la cual el compilador de Rust se rehusa a compilar un programa el cual el autor piensa valido. Esto ocurre con frecuencia debido a que el modelo mental del programador acerca de como funciona la pertenencia no concuerda con las reglas actuales implementadas en Rust. Probablemente tu experimentes cosas similares al comienzo. Sin embargo, hay buenas noticias: otros desarrolladores Rust experimentados reportan que una vez que trabajan con las reglas del sistema de pertenencia por un periodo de tiempo, pelean cada vez menos con el comprobador de préstamo.

Con eso en mente, aprendamos acerca de los tiempos de vida.

# Tiempos de vida

Prestar una referencia a otro recurso del que alguien mas es dueno puede ser complicado. Por ejemplo, imagina este conjunto de operaciones:

- Yo Obtengo un handle a algun tipo de recurso.
- Te presto una referencia a el recurso.
- Decido que he terminado con el recurso, y lo libero, mientras todavia tienes la referencia a el.
- Tu decides usar el recurso.

Oh no! Tu referencia es ta apuntandon a un recurso invalido. Esto es llamado un puntero colgante o `uso despues de liberacion`, cuando el recurso es memoria.

Para arreglar esto, tenemos que asegurranos que el paso cuatro nunca ocurra despues del paso tres. El sistema de pertenencia de Rust lleva acabo esto a traves de un concepto denominado tiempos de vida, los cuales describen el ambito en el cual una referencia es valida.

Cuando tenemos una funcion que toma una referencia como argumento, podemos ser implicitos o explicitos acrca del tiempo de vida de la referencia:

```rust
// implicito
fn foo(x: &i32) {
}

// explicito
fn bar<'a>(x: &'a i32) {
}
```

El `'a` se lee ‘el tiempo de vida a’. Tecnicamente, toda referencia posee un tiempo de vida asociado a ella, pero el compilador te permite omitirlas en casos comunes. Antes que llegemos a eso, analicemos el pedazo explicito:


```rust,ignore
fn bar<'a>(...)
```

Anteriormente hablamos un poco acerca de la [sintaxis de funciones][functions], pero no discutimos los `<>`s despues de un nombre de funcion. Una funcion puede tener ‘parametros genericos’ entre los `<>`s, y los tiempos de vida son un tipo de parametros genericos. Discutiremos otros tipos de dgenericos [mas tarde en el libro][generics], pero por ahora, enfoquemonos solo en el aspecto de los tiempos de vida.

[functions]: functions.html
[generics]: generics.html

Usamos `<>` para declarar nuestros tiempos de vida. Esto dice que `bar` posee un tiempo de vida, `'a`. De haber tenido referencias como parametros, hubiese lucido de esta manera:

```rust,ignore
fn bar<'a, 'b>(...)
```

Entonces en nuestra lista de parametros, usamos los tiempos de vida que hemos nombrado:

```rust,ignore
...(x: &'a i32)
```

De haber querido una referencia `&mut`, pudimos haber hecho lo siguiente:

```rust,ignore
...(x: &'a mut i32)
```

Si comparas `&mut i32` con `&'a mut i32`, estas son lo mismo, es solo que el tiempo de vida `'a` se ha metido entre el `&` y el `mut i32`.  Leemos `&mut i32` como ‘una referencia mutable a un `i32`’ y `&'a mut i32` como ‘una referencia mutable a un `i32` con el tiempo de vida `'a`’.

# En `struct`s

Tambien necesitaras tiempos de vida explicitos cuando trabajes con [`struct`][structs]s:

```rust
struct Foo<'a> {
    x: &'a i32,
}

fn main() {
    let y = &5; // esto es lo mismo que `let _y = 5; let y = &_y;`
    let f = Foo { x: y };

    println!("{}", f.x);
}
```

[structs]: structs.html

Como puedes ver, los `struct`s pueden tener tiempos de vida tambien. En una forma similar a las funciones,


```rust
struct Foo<'a> {
# x: &'a i32,
# }
```

declara un tiempo de vida, y

```rust
# struct Foo<'a> {
x: &'a i32,
# }
```

hace uso de el. Entoces, porque necesitamos un tiempo de vida aqui? Necesitamos asegurarnos que cualquier referencia a un `Foo` no pueda vivir mas que la referencia a un `i32` que este contiene.

uses it. So why do we need a lifetime here? We need to ensure that any reference
to a `Foo` cannot outlive the reference to an `i32` it contains.

## bloques `impl`

Implementemos un metodo en `Foo`:


```rust
struct Foo<'a> {
    x: &'a i32,
}

impl<'a> Foo<'a> {
    fn x(&self) -> &'a i32 { self.x }
}

fn main() {
    let y = &5; // esto es lo mismo que `let _y = 5; let y = &_y;`
    let f = Foo { x: y };

    println!("x es: {}", f.x());
}
```

Como puedes ver, neceitamos declarar un tiempo de vida para `Foo` en la linea `impl`. Repetimos `'a` dos veces, justo como en funciones: `impl<'a>` define un tiempo de vida `'a`, y `Foo<'a>` hace uso de el.

## Multiples tiempo de vida

Si posees multiples referencias, puedes hacer uso de el mismo tiemnpo de vida multiples veces:


```rust
fn x_o_y<'a>(x: &'a str, y: &'a str) -> &'a str {
#    x
# }
```

Lo anterior dice que ambos `x` y `y` viven por el mismo ambito, y que el valor de retorno tambien esta vivio para ese ambito. Si hubieses querido que `x` y `y` tuviesen diferentes tiempos de vida, puedes hacer uso de multiples parametros de tiempos de vida:


```rust
fn x_o_y<'a, 'b>(x: &'a str, y: &'b str) -> &'a str {
#    x
# }
```

En este ejemplo, `x` y `y` tienen diferentes ambitos validos, pero el valor de retorno tiene el mismo tiempo de vida que `x`.


## Pensando en ambitos

Una forma de pensar acerca de los tiempod de vida es visualizar el ambito en el cual es valida una referencia. Por ejemplo:


```rust
fn main() {
    let y = &5;     // -+ y entra en ambito
                    //  |
    // stuff        //  |
                    //  |
}                   // -+ y sale de ambito
```

Agregando nuestro `Foo`:

```rust
struct Foo<'a> {
    x: &'a i32,
}

fn main() {
    let y = &5;           // -+ y entra en ambito
    let f = Foo { x: y }; // -+ f entra en ambito
    // stuff              //  |
                          //  |
}                         // -+ f y y salen de ambito
```

Nuestro `f` vive dentro de el ambito de `y`, es por ello que todo funciona. Que pasaria de lo contrario? Este codigo no funcionaria:


```rust,ignore
struct Foo<'a> {
    x: &'a i32,
}

fn main() {
    let x;                    // -+ x entra en ambito
                              //  |
    {                         //  |
        let y = &5;           // ---+ y entra en ambito
        let f = Foo { x: y }; // ---+ f entra en ambito
        x = &f.x;             //  | | error aqui
    }                         // ---+ f y y salen de ambito
                              //  |
    println!("{}", x);        //  |
}                             // -+ x sale de ambito
```

Uff! Como puedes ver aqui, los ambitos de `f` y `y` son menores que el ambito de `x`. Pero cuando hacemos `x = &f.x`, hacemos a `x` una referencia a algo que estar por salir de ambito.

Los tiempos de vida con nombre son una forma de darles a dichos ambitos un nombre. Darle un nombre a algo es el primer paso hacia el poder hablar acerca de ello.

## 'static

El tiempo de vida denominado ‘static’ es u tiempo de vida especial. Este senala que algo posee el tiempo de vida de el programa completo. La mayoria de los desarrolladores Rust conocen a `'static` cuando lidian con cadenas de caracteres:

```rust
let x: &'static str = "Hola, mundo.";
```

Los literales de cadenas de caracteres posee el tipo `&'static str` puesto que la referencia esta siempre viva: estos son colocados en el segmento de datos del binario final. Otro ejemplo son las globales:

```rust
static FOO: i32 = 5;
let x: &'static i32 = &FOO;
```

Lo anterior agrega un `i32` a el segmento de datos de el binario, y `x` es una referencia a el.

## Elision de tiempos de vida

Rust soporta una inferencia de tipos poderosa en cuerpos de funcion, pero esta prohibido en las firmas de elementos permitir razonamiento basado unicamente en la firma. Sin embargo, por razones ergonomicas, una inferencia secundaria muy restringida llamada
“elision de tiempos de vida” se aplica en las firmas de funcion. Esta infiere basandose solo en los componentes de la firma sin basarse en el cuerpo de la funcion, unicamnete infiere parametros de tiempos de vida, y hace esto con solo tres reglas facilmente memorizables e inambiguas. Todo esto hace a la elision de tiempos de vida un atajo para escribir una firma, sin necesidad de ocultar los tipos involucrados puesto a que inferencia local completa sera aplicada a ellos.

Cuando se habla de elision de tiempos de vida, usamos el termino *tiempo de vida de entrada* y *tiempo de vida de salida*. Un *tiempo de vida de entrada* es un tiempo de vida asociado con un parametro de una funcion, y un *tiempo de vida de salida* es un tiempo de vida asociado con el valor de retorno de una funcion. Por ejemplo, la siguiente funcion tiene un tiempo de vida de entrada:

```rust,ignore
fn foo<'a>(bar: &'a str)
```

Esta posee un tiempo de vida de salida:

```rust,ignore
fn foo<'a>() -> &'a str
```

La siguiente tiene un tiempo de vida en ambas posisiones:

```rust,ignore
fn foo<'a>(bar: &'a str) -> &'a str
```

He aqui las tres reglas:

* Cada tiempo de vida elidido en los argumentos de una funcion se convierte en un parametro de tiempo de vida distinto.

* Si existe exactamente un solo tiempo de vida de entrada, elidido o no, ese tiempo de vida es asigando a todos los tiempos de vida elididos en los valores de retorno de esa funcion.

* Si existen multiples tiempos de vida de entrada, pero una de ellos es `&self` o `&mut self`, el tiempo de vida de `self` es asignado a todos los tiempos de vida de salida elididos.

De lo contrario, es un error elidir un tiempo de vida de salida.

### Ejemplos

He aqui algunos ejemplos de funciones con tiempos de vida elididos. Hemos pareado cada ejemplo de un tiempo de vida elidido con su forma expandida.

```rust,ignore
fn print(s: &str); // elidido
fn print<'a>(s: &'a str); // expandido

fn debug(lvl: u32, s: &str); // elidido
fn debug<'a>(lvl: u32, s: &'a str); // expandido

// En el ejemplo anterior, `lvl` no necesita un tiempo de vida debido a que no es una referencia (`&`). Solo las cosas relacionadas con referencias (como un `struct` que contiene una referencia) necesitan tiempos de vida.

fn substr(s: &str, until: u32) -> &str; // elidido
fn substr<'a>(s: &'a str, until: u32) -> &'a str; // expandido

fn get_str() -> &str; // ILLEGAL, no inputs

fn frob(s: &str, t: &str) -> &str; // ILEGAL, dos entradas
fn frob<'a, 'b>(s: &'a str, t: &'b str) -> &str; // Expandido: Tiempo de vida de salida es ambiguo

fn get_mut(&mut self) -> &mut T; // elidido
fn get_mut<'a>(&'a mut self) -> &'a mut T; // expanded

fn args<T:ToCStr>(&mut self, args: &[T]) -> &mut Command // elidido
fn args<'a, 'b, T:ToCStr>(&'a mut self, args: &'b [T]) -> &'a mut Command // expanded

fn new(buf: &mut [u8]) -> BufWriter; // elidido
fn new<'a>(buf: &'a mut [u8]) -> BufWriter<'a> // expanded
```
