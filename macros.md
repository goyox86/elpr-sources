% Macros

Por ahora, has aprendido sobre muchas las herramientas en Rust para abstraer y
reutilizar el código. Estas unidades de reutilización tienen una rica
estructura semántica. Por ejemplo, las funciones tienen un tipo, los parámetros
de tipo tienen límites de traits, y functiones sobrecargadas deben pertenecer a
un trait particular.

Debido a esta estructura, abstracciones fundamentales de Rust tienen potente
comprobación de exactitud. Pero, el precio es flexibilidad reducida. Si se
identifica visualmente un patrón de código repetido, podría ser dificil o
tedioso para expresar ese patrón como una función genérica, un trait, o
cualquier otra parte de la semántica de Rust.

Macros nos permiten abstraer a un nivel *sintáctic*. Una invocación de macro es
la abreviatura de una forma sintáctica "expandido". Esta expansión ocurre
durante la compilación, antes de comprobación estática. Como resultado, las
macros pueden capturar muchos patrones de reutilización de código que las
abstracciones fundamentales de Rust no puede.

El inconveniente es que el código basado en la macro puede ser más difícil de
entender, porque menos de las normas incorporadas aplican. Al igual que una
función ordinaria, una macro de buen comportamiento se puede utilizar sin
entender detalles de implementación. Sin embargo, puede ser difícil diseñar una
macro de buen comportamiento! Además, los errores de compilación en código de
macro son más difíciles de entender, porque describen problemas en el código
expandido, no el formulario de origen que los desarrolladores utilizan.

Estos inconvenientes hacen macros una "herramienta de último recurso". Eso no
quiere decir que las macros son malos; son parte de Rust porque a veces se
necesitan las macros para código concisa y abstraído. Sólo tenga en cuenta este
equilibrio.

# Definición de una macro

Puede que haya visto la macro `vec!`, utilizado para inicializar un [vector][]
con cualquier número de elementos.

[vector]: arrays-vectors-and-slices.html

```rust
let x: Vec<u32> = vec![1, 2, 3];
# assert_eq!(&[1,2,3], &x);
```

Esto no puede ser una función ordinaria, porque acepta cualquier número de
argumentos. Pero podemos imaginarlo como abreviatura sintáctica para

```rust
let x: Vec<u32> = {
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
};
# assert_eq!(&[1,2,3], &x);
```

Podemos implementar esta abreviatura, utilizando una macro: [^actual]

[^actual]: La propia definición de `vec!` en libcollections difiere de la
           presentada aquí, por razones de eficiencia y reutilización.

```rust
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
# fn main() {
#     assert_eq!(vec![1,2,3], [1, 2, 3]);
# }
```

¡Whoa!, eso es mucha nueva sintaxis. Examinemos pieza por pieza.

```ignore
macro_rules! vec { ... }
```

Este dice que estamos definiendo una macro llamada `vec`, tanto
como `fn vec` definiría una función llamada `vec`. En prosa,
informalmente escribimos el nombre de una macro con un signo de
exclamación, por ejemplo, `vec!`. El signo de exclamación es
parte de la sintaxis de invocación y sirve para distinguir una
macro desde una función ordinaria.

## Emparejamiento de patrones

La macro se define a través de una serie de reglas de emparejamiento de patrones.
Por encima, tuvimos

```ignore
( $( $x:expr ),* ) => { ... };
```

Esto es como un brazo de expresión `match`, pero coincide con árboles de
sintaxis Rust en tiempo de compilación. El punto y coma es opcional en el caso
final (aquí, el único caso). El "patrón" en el lado izquierdo de `=>` es
conocido como un 'matcher'. Estos tienen [su propio pequeño gramática][] dentro
del language.

[su propio pequeño gramática]: ../reference.html#macros

El matcher `$x:expr` coincidirá con cualquier expresión Rust, vinculante esta
árbol sintáctico a la 'metavariable' `$x`. El identificador `expr` es un
'especificador fragmento'; todas las posibilidades se enumeran más adelante en
este capitulo. Al rodear el matcher con `$(...),*`, que coincidirá con cero o
más expresiones separadas por comas.

Aparte de la sintaxis especial de matcher, las tokens Rust que aparecen en un
matcher deben coincidir exactamente. Por ejemplo,

```rust
macro_rules! foo {
    (x => $e:expr) => (println!("mode X: {}", $e));
    (y => $e:expr) => (println!("mode Y: {}", $e));
}

fn main() {
    foo!(y => 3);
}
```

imprimirá

```text
mode Y: 3
```

Con

```rust,ignore
foo!(z => 3);
```

obtenemos el error del compilador

```text
error: no rules expected the token `z`
```

## Expansión

El lado derecho de una regla macro es la sintaxis Rust ordinaria, en su mayor
parte. Pero podemos insertar partes de sintaxis capturados por el matcher. Del
ejemplo original:

```ignore
$(
    temp_vec.push($x);
)*
```

Cada expresión emparejado `$x` producirá una sola expresión `push` en la
expansión de macros. La repetición se desarrolla en "lockstep" con la
repetición en el matcher (más sobre esto en un momento).

Porque `$x` ya fue declarado como emparejar una expresión, no repetimos `:expr`
en el lado derecho. Además, no incluimos una coma separando como parte del
operador de repetición. En cambio, tenemos un punto y coma que termina dentro
del bloque repetido.

Otro detalle: la macro `vec` tiene *dos* pares de paréntesis rizado en la mano
derecha. A menudo se combinan de este modo:

```ignore
macro_rules! foo {
    () => {{
        ...
    }}
}
```

Los paréntesis exteriores son parte de la sintaxis de `macro_rules!`. De hecho,
puede utilizar `()` o `[]` en su lugar. Ellos simplemente delimitan el lado
derecho como un todo.

Los paréntesis interiores son parte de la sintaxis expandida. Recuerde, la
macro `vec!` se utiliza en un contexto de expresión. Para escribir una
expresión con varias statements, entre ellas `let`-bindings, utilizamos un
bloque. Si la macro se expande a una sola expresión, no necesitas esta
capa extra de paréntesis.

Tenga en cuenta que nunca *declaramos* que la macro produce una expresión. De
hecho, esto no se determina hasta que usamos la macro como una expresión. Con
cuidado, se puede escribir una macro cuya expansión trabaja en varios
contextos. Por ejemplo, la abreviatura de un tipo de datos podría ser válido
como una expresión o un patrón.

## Repetición

El operador de repetición sigue dos reglas principales:

1. `$(...)*` camina a través de una "capa" de repeticiones, para todos los `$nombre`s
    que contiene, al mismo paso, y
2. cada `$nombre` debe estar bajo al menos tantos `$(...)*` como se compara con.
   Si es bajo más, que va a ser duplicado, según el caso.

Esta macro barroca ilustra la duplicación de las variables de los niveles de
repetición exteriores.

```rust
macro_rules! o_O {
    (
        $(
            $x:expr; [ $( $y:expr ),* ]
        );*
    ) => {
        &[ $($( $x + $y ),*),* ]
    }
}

fn main() {
    let a: &[i32]
        = o_O!(10; [1, 2, 3];
               20; [4, 5, 6]);

    assert_eq!(a, [11, 12, 13, 24, 25, 26]);
}
```


Esa es la mayor parte de la sintaxis de matcher. Estos ejemplos utilizan
`$(...)*`, que coincide con "cero o más" elementos sintácticos.
Alternativamente, puedes escribir `$(...)+` para coincide con "uno o
más".  Ambas formas incluyen, opcionalmente, un separador, que puede ser
cualquier token excepto `+` o `*`.

Este sistema se basa en
"[Macro-by-Example](http://www.cs.indiana.edu/ftp/techreports/TR206.pdf)"
(PDF).

# Higiene

Algunos lenguajes implementan macros con sustitución de texto simple, lo que
conduce a diversos problemas. Por ejemplo, este programa C imprime `13` en
lugar de la esperada `25`.

```text
#define CINCO_VECES(x) 5 * x

int main() {
    printf("%d\n", CINCO_VECES(2 + 3));
    return 0;
}
```

Después de la expansión tenemos `5 * 2 + 3`, y la multiplicación tiene mayor
precedencia que la suma. Si has utilizado macros C mucho, probablemente
sabes los idiomas estándar para evitar este problema, así como cinco o seis
otros. En Rust, no nos preocupamos por ello.

```rust
macro_rules! cinco_veces {
    ($x:expr) => (5 * $x);
}

fn main() {
    assert_eq!(25, cinco_veces!(2 + 3));
}
```

El metavariable `$x` se analiza como un nodo de expresión individual, y
mantiene su lugar en el árbol de sintaxis, incluso después de la sustitución.

Otro problema común en los sistemas de macro es "captura variable". Aquí hay
una macro C, utilizando [una extensión de GNU C][] para emular bloques de
expresión de Rust.

[una extensión de GNU C]: https://gcc.gnu.org/onlinedocs/gcc/Statement-Exprs.html

```text
#define LOG(msg) ({ \
    int state = get_log_state(); \
    if (state > 0) { \
        printf("log(%d): %s\n", state, msg); \
    } \
})
```

Aquí es un caso simple de usar que va terriblemente mal:

```text
const char *state = "reticulante acanaladuras";
LOG(state)
```

Esto expande a

```text
const char *state = "reticulante acanaladuras";
int state = get_log_state();
if (state > 0) {
    printf("log(%d): %s\n", state, state);
}
```

La segunda variable llamada `state` ensombrece de la primera.
Esto es un problema porque la expresión de impresión (`printf`)
debe hacer referencia a los dos.

La macro Rust equivalente tiene el comportamiento deseado.

```rust
# fn get_log_state() -> i32 { 3 }
macro_rules! log {
    ($msg:expr) => {{
        let state: i32 = get_log_state();
        if state > 0 {
            println!("log({}): {}", state, $msg);
        }
    }};
}

fn main() {
    let state: &str = "reticulante acanaladuras";
    log!(state);
}
```

Esto funciona porque Rust tiene una [sistema de macros
higiénica][]. Cada expansión macro ocurre en un "contexto de
sintaxis" distinto, y cada variable está asociada con el
contexto de sintaxis donde fue introducida. Es como si el
variable `state` dentro `main` está pintado de un "color"
diferente de la variable `state` dentro de la macro, y por lo
tanto no contradigan.

[sistema de macros higiénica]: http://en.wikipedia.org/wiki/Hygienic_macro


Esto también restringe la capacidad de macros para introducir nuevos enlaces en
el sitio de invocación. Código como el siguiente no funcionará:

```rust,ignore
macro_rules! foo {
    () => (let x = 3);
}

fn main() {
    foo!();
    println!("{}", x);
}
```

En lugar necesitas para pasar el nombre de la variable en la invocación, por lo
que tiene el contexto de sintaxis correcta.

```rust
macro_rules! foo {
    ($v:ident) => (let $v = 3);
}

fn main() {
    foo!(x);
    println!("{}", x);
}
```

Esto es válido para enlaces `let` y etiquetas de bucle, pero no para [items][items]. Así que el siguiente código hace compilar:

```rust
macro_rules! foo {
    () => (fn x() { });
}

fn main() {
    foo!();
    x();
}
```

[items]: ../reference.html#items

# Macros recursivas

La expansión de una macro puede incluir más invocaciones macro, incluyendo
invocaciones de la misma macro ser expandido. Estas macros recursivas son
útiles para el procesamiento de entrada con estructura de árbol, como se
ilustra por esta (simplista) taquigrafía HTML:

```rust
# #![allow(unused_must_use)]
macro_rules! write_html {
    ($w:expr, ) => (());

    ($w:expr, $e:tt) => (write!($w, "{}", $e));

    ($w:expr, $tag:ident [ $($inner:tt)* ] $($rest:tt)*) => {{
        write!($w, "<{}>", stringify!($tag));
        write_html!($w, $($inner)*);
        write!($w, "</{}>", stringify!($tag));
        write_html!($w, $($rest)*);
    }};
}

fn main() {
#   // FIXME(#21826)
    use std::fmt::Write;
    let mut out = String::new();

    write_html!(&mut out,
        html[
            head[title["Macros guide"]]
            body[h1["Macros are the best!"]]
        ]);

    assert_eq!(out,
        "<html><head><title>Macros guide</title></head>\
         <body><h1>Macros are the best!</h1></body></html>");
}
```

# Depuración de código de macro

Para ver los resultados de las macros en expansión, ejecutar
`rustc --pretty expanded`. La salida representa toda un crate, por lo que también
puede alimentar de nuevo en a `rustc`, que a veces producir mejores mensajes de
error de la compilación inicial. Tenga en cuenta que la salida de `--pretty
expanded` puede tener un significado diferente si varias variables del mismo
nombre (pero diferentes contextos sintácticos) están en juego en el mismo
ámbito. En este caso `--pretty expanded,hygiene` le dirá acerca de los
contextos de sintaxis.

`rustc` ofrece dos extensiones de sintaxis que ayudan con el depuración de
macros. Por ahora, son inestables y requieren puertas de características.

* `log_syntax!(...)` imprimirá sus argumentos en la salida estándar, en tiempo
  de compilación, y "expandir" para nada.

* `trace_macros!(true)` permitirá un mensaje compilador cada vez que una macro es
  expandida. Use `trace_macros!(false)` adelante en la expansión de apagarlo.

# Requisitos sintácticos

Incluso cuando el código Rust contiene macros un-expandida, se puede
analizar como un completo [árbol sintáctico][ast]. Esta propiedad puede ser
muy útil para los editores y otras herramientas que procesan código. También
tiene algunas consecuencias para el diseño del sistema de macro de Rust.

[ast]: glossary.html#abstract-syntax-tree

Una consecuencia es que Rust debe determinar, cuando se analiza una invocación
macro, si la macro se destaca por

* cero o más items,
* cero o más methods,
* una expresión,
* un statement, o
* un patrón.

Una invocación macro dentro de un bloque podía expandir para algunos items, o
para un expresión. Rust utiliza una regla simple para resolver esta ambigüedad.
Una invocación macro que significa items debe ser

* delimitada por paréntesis rizado, como `foo! { ... }`, o
* terminada por un punto y coma, como `foo!(...);`.

Otra consecuencia del análisis de pre-expansión es que la invocación macro debe
constar de tokens válidos de Rust. Además, paréntesis (ronda, cuadrado, y
rizado) deben equilibrarse dentro de un invocación macro. Por ejemplo,
`foo!([)` está prohibido. Esto permite Rust saber dónde termina la invocación
macro.

Más formalmente, el cuerpo de invocación macro debe ser una secuencia de
"árboles de token". Un árbol de token se define de forma recursiva, ya sea como

* una secuencia de árboles token rodeadas de `()`, `[]`, o `{}`, o
* cualquier otra token única.

Dentro de un matcher, cada metavariable tiene un 'especificador fragmento',
identificando qué forma sintáctica coincide.

* `ident`: un identificador. Por ejemplo: `x`; `foo`.
* `path`: un nombre calificado. Por ejemplo: `T::SpecialA`.
* `expr`: una expresión. Por ejemplo: `2 + 2`; `if true then { 1 } else { 2 }`; `f(42)`.
* `ty`: un tipo. Por ejemplo: `i32`; `Vec<(char, String)>`; `&T`.
* `pat`: un patrón. Por ejemplo: `Some(t)`; `(17, 'a')`; `_`.
* `stmt`: una sola statement. Por ejemplo: `let x = 3`.
* `block`: una secuencia de statements. Por ejemplo:
  `{ log(error, "hi"); return 12; }`.
* `item`: un [item][item]. Por ejemplo: `fn foo() { }`; `struct Bar;`.
* `meta`: un "meta item", como se encuentra en atributos. Por ejemplo: `cfg(target_os = "windows")`.
* `tt`: un árbol de token única.

Existen reglas adicionales con respecto a la siguiente token después de un metavariable:

* `expr` solamente puede ser seguido por uno de: `=> , ;`
* `ty` y `path` solamente puede ser seguido por uno de: `=> , : = > as`
* `pat` solamente puede ser seguido por uno de: `=> , = if in`
* Otros variables puede ser seguido de cualquier token.

Estas reglas proporcionan cierta flexibilidad para evolucionar la sintaxis de
Rust sin romper macros existentes.

El sistema de macro no se ocupa analizar la ambigüedad. Por ejemplo, la
gramática `$($t:ty)* $e:expr` siempre dejará de analizar, porque el analizador
se ve obligada a elegir entre analizar `$t` y analizar `$e`. Cambio de la
sintaxis de invocación para poner un token distintiva delante puede resolver
el problema. En este caso, se puede escribir `$(T $t:ty)* E $e:exp`.

[item]: ../reference.html#items

# Alcance y importación/exportación de macros

Las macros se expanden en una etapa temprana en la compilación, antes de la
resolución de nombres. Una desventaja es que alcance funciona de forma
diferente para las macros, en comparación con otras construcciones en el
lenguaje.

Definición y expansión de macros ambos suceden en un solo recorrido de un
crate. Así que una macro se define en el alcance módulo es visible para
cualquier código posterior en el mismo módulo, que incluye el cuerpo de un niño
`mod` items subsiguientes.

Una macro definida en el cuerpo de una `fn`, o en cualquier otro lugar, no en
el ámbito de módulo, sólo es visible dentro de ese parte.

Si un módulo tiene el atributo `macro_use`, sus macros también son visibles en
su módulo padre después del item `mod` del niño. Si el padre también tiene
`macro_use`, entonces las macros serán visibles en el abuelo después del item
`mod` del padre, y así sucesivamente.

El atributo `macro_use` también puede aparecer en `extern crate`. En este
contexto, controla qué macros se cargan desde el crate externo, por ejemplo,

```rust,ignore
#[macro_use(foo, bar)]
extern crate baz;
```

Si el atributo se da simplemente como `#[macro_use]`, todas las macros se cargan. Si no hay atributo `#[macro_use]` entonces no hay macros se cargan. Sólo macros definidas con el atributo `#[macro_export]` pueden cargar.

Para cargar las macros de un cajón sin vincularla a la salida, utilice `#[no_link]` también.

Un ejemplo:

```rust
macro_rules! m1 { () => (()) }

// visibles aquí: m1

mod foo {
    // visibles aquí: m1

    #[macro_export]
    macro_rules! m2 { () => (()) }

    // visibles aquí: m1, m2
}

// visibles aquí: m1

macro_rules! m3 { () => (()) }

// visibles aquí: m1, m3

#[macro_use]
mod bar {
    // visibles aquí: m1, m3

    macro_rules! m4 { () => (()) }

    // visibles aquí: m1, m3, m4
}

// visibles aquí: m1, m3, m4
# fn main() { }
```

Cuando esta biblioteca se carga con `#[macro_use] extern crate`, se importará solamente `m2`.

El Referencia de Rust tiene una [lista de atributos relacionados a macros](../reference.html#macro-related-attributes).

# La variable `$crate`

Una dificultad adicional se produce cuando se utiliza una macro en múltiples crates. Decir que define `mibibl`

```rust
pub fn increment(x: u32) -> u32 {
    x + 1
}

#[macro_export]
macro_rules! inc_a {
    ($x:expr) => ( ::increment($x) )
}

#[macro_export]
macro_rules! inc_b {
    ($x:expr) => ( ::mibibl::increment($x) )
}
# fn main() { }
```

`inc_a` sólo funciona dentro de `mibibl`, mientras que `inc_b` sólo funciona
fuera de la biblioteca. Además, `inc_b` se romperá si importas `mibibl` con
otro nombre.

Rust no (todavía) tiene un sistema de higiene para las referencias de crates,
pero proporciona una solución sencilla para este problema. Dentro de una macro
importado de un crate llamado `foo`, la variable macro especial `$crate`
expandirá a `::foo`. Por el contrario, cuando una macro se define y luego se
usa en el mismo crate, `$crate` se expandirá para nada. Esto significa que
podemos escribir

```rust
#[macro_export]
macro_rules! inc {
    ($x:expr) => ( $crate::increment($x) )
}
# fn main() { }
```

para definir una sola macro que funciona tanto dentro como fuera de nuestra
biblioteca. El nombre de la función expandirá a cualquiera `::increment` o
`::mibibl::increment`.

Para mantener este sistema simple y correcta, `#[macro_use] extern crate ...`
solamente puede aparecer en la raíz de su crate, no dentro de `mod`. Esto
asegura que `$crate` es un único identificador.

# El extremo profundo

Las secciones antes mencionadas macros recursivas, pero no dio la historia
completa. Macros recursivas son útiles por otra razón: Cada invocación
recursiva es una otra oportunidad para emparejar los patrónes con los
argumentos de la macro.

Como un ejemplo extremo, es posible, aunque poco recomendable, para implementar la autómata [Bitwise Cyclic Tag](http://esolangs.org/wiki/Bitwise_Cyclic_Tag) dentro del sistema macro de Rust.

```rust
macro_rules! bct {
    // cmd 0:  d ... => ...
    (0, $($ps:tt),* ; $_d:tt)
        => (bct!($($ps),*, 0 ; ));
    (0, $($ps:tt),* ; $_d:tt, $($ds:tt),*)
        => (bct!($($ps),*, 0 ; $($ds),*));

    // cmd 1p:  1 ... => 1 ... p
    (1, $p:tt, $($ps:tt),* ; 1)
        => (bct!($($ps),*, 1, $p ; 1, $p));
    (1, $p:tt, $($ps:tt),* ; 1, $($ds:tt),*)
        => (bct!($($ps),*, 1, $p ; 1, $($ds),*, $p));

    // cmd 1p:  0 ... => 0 ...
    (1, $p:tt, $($ps:tt),* ; $($ds:tt),*)
        => (bct!($($ps),*, 1, $p ; $($ds),*));

    // alto en cadena vacía
    ( $($ps:tt),* ; )
        => (());
}
```

Ejercicio: usar macros para reducir la duplicación en la definición anterior de la macro `bct!`.

# Macros comunes

Estas son algunas de las macros comunes que verás en el código de Rust.

## panic!

Esta macro hace que el thread actual de pánico. Puede darle un mensaje de pánico.

```rust,no_run
panic!("qué vergüenza");
```

## vec!

La macro `vec!` se utiliza en todo el libro, por lo que probablemente ha visto
ya. Crea los `Vec<T>` con facilidad:

```rust
let v = vec![1, 2, 3, 4, 5];
```

También le permite realizar los vectores con valores de repetir. Por ejemplo, un centenar de ceros:

```rust
let v = vec![0; 100];
```

## assert! y assert_eq!

Estas dos macros se utilizan en las pruebas. `assert!` toma un boolean.
`assert_eq!` toma dos valores y los comprueba por la igualdad. `true` pasa,
`false` entra en pánico. Como este:

```rust,no_run
// A-ok!

assert!(true);
assert_eq!(5, 3 + 2);

// nope :(

assert!(5 < 3);
assert_eq!(5, 3);
```

## try!

`try!` se utiliza para el tratamiento de errores. Toma algo que puede devolver un
`Result<T, E>`, y da `T` si es un `Ok(T)` y se `return` con la
`Err(E)` si se trata de eso. Como este:

```rust,no_run
use std::fs::File;

fn foo() -> std::io::Result<()> {
    let f = try!(File::create("foo.txt"));

    Ok(())
}
```

Esto es más limpio que hacer esto:

```rust,no_run
use std::fs::File;

fn foo() -> std::io::Result<()> {
    let f = File::create("foo.txt");

    let f = match f {
        Ok(t) => t,
        Err(e) => return Err(e),
    };

    Ok(())
}
```

## unreachable!

Esta macro se utiliza cuando se piensa en algo de código nunca debe ejecutar:

```rust
if false {
    unreachable!();
}
```

A veces, el compilador puede hacer que tienes una rama que sabe que nunca,
nunca correr. En estos casos, utilizar esta macro, por lo que si terminas
mal, obtendrá un `panic!`.

```rust
let x: Option<i32> = None;

match x {
    Some(_) => unreachable!(),
    None => println!("Sé que x es None!"),
}
```

## unimplemented!

Esta macro se puede utilizar cuando estás tratando de typecheck sus
funciones, y no quieren que preocuparse por escribir el cuerpo de la función.
Un ejemplo de esta situación está implementando un trait con métodos múltiples
requeridos, en los que desea hacer frente a uno a la vez. Definir los demás
como `unimplemented!` hasta que esté listo para escribirlas.

# Macros de procedimiento

Si el sistema de macro de Rust no puede hacer lo que necesita, es posible que
desee escribir una [plugin compilador](compiler-plugins.html) en su lugar. En
comparación con macros `macro_rules!`, esto es significativamente más trabajo,
las interfaces son mucho menos estables, y los errores pueden ser mucho más
difíciles de localizar. A cambio se obtiene la flexibilidad de la ejecución de
código arbitrario en Rust en tiempo de compilación. Plugins de extensión de
sintaxis a veces se llaman "procedural macros" (macros de procedimiento) por
este motivo.
