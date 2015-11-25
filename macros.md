% Macros

Por ahora, has aprendido mucho sobre las herramientas que Rust ofrece para abstraer y
reutilizar código. Estas unidades de reutilización poseen una rica
estructura semántica. Por ejemplo, las funciones tienen una firma de tipos, los parámetros
de tipo tienen límites de traits y las funciones sobrecargadas deben pertenecer a
un trait particular.

Esta estructura significa que las principales abstracciones en Rust poseen un
poderoso mecanismo de chequeo en tiempo de compilacion. Pero, el precio es una flexibilidad reducida.
Si se identifica visualmente un patrón de código repetido, podría ser dificil o
tedioso expresar ese patrón como una función genérica, un trait, o
cualquier otro elemento de la semántica de Rust.

Las macros nos permiten abstraer a un nivel *sintáctico*. Una invocación de macro es
la abreviatura de una forma sintáctica "expandida". Dicha expansión ocurre
durante la compilación, antes de comprobación estática. Como resultado, las
macros pueden capturar muchos patrones de reutilización de código que las
abstracciones fundamentales de Rust no pueden.

El inconveniente es que el código basado en macros puede ser más difícil de
entender, porque menos de las normas internas de Rust aplican. Al igual que una
función ordinaria, una macro bien hecha se puede utilizar sin
entender detalles de implementación. Sin embargo, puede ser difícil diseñar una
macro con un buen comportamiento! Además, los errores de compilación en código de
macros son más difíciles de entender, porque describen problemas en el código
expandido, no a nivel del codigo funete que usan los desarrolladores.

Estos inconvenientes hacen de las macros una "herramienta de último recurso". Lo anterior no
quiere decir que las macros son malas; forman parte de Rust porque a veces son necesarias
para código conciso y abstracto. Simplemente manten en cuenta este equilibrio.

# Definiendo una macro

Puede que ya hayas visto la macro `vec!`, utilizada para inicializar un [vector][]
con un numero cualquiera de elementos.

[vector]: arrays-vectors-and-slices.html

```rust
let x: Vec<u32> = vec![1, 2, 3];
# assert_eq!(&[1,2,3], &x);
```

Esto no puede ser una función ordinaria, porque acepta cualquier número de
argumentos. Pero podemos imaginarlo como abreviacion sintáctica para

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
           presentada aquí, por razones de eficiencia y reutilización. Algunas
           de ellas son mencionadas en el [capitulo avanzado de macros][].

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

¡Whoa!, un monton de sintaxis nueva. Examinemoslo parte por parte.

```ignore
macro_rules! vec { ... }
```

Lo anterior dice que estamos definiendo una macro llamada `vec`, al igual que
`fn vec` definiría una función llamada `vec`. En prosa,
informalmente escribimos el nombre de una macro con un signo de
exclamación, por ejemplo, `vec!`. Este signo de exclamación es
parte de la sintaxis de invocación y sirve para distinguir una
macro de una función ordinaria.

## Conicidencia de patrones

La macro se define a través de una serie de reglas, las cuales son casos de coincidencia de patrones. Anteriomente vimos

```ignore
( $( $x:expr ),* ) => { ... };
```

Lo anteriror es similar a un brazo de una expresión `match`, pero las pruebas para coincidencia
ocurren sobre los árboles de sintaxis Rust en tiempo de compilación. El punto y coma es opcional en el caso
final (aquí, el único caso). El "patrón" en el lado izquierdo del `=>` es
conocido como un 'matcher'. Los matchers tienen [su propia pequeño gramática] dentro
del language.

[su propia pequeño gramática]: ../reference.html#macros

El matcher `$x:expr` coincidirá con cualquier expresión Rust, asociando ese
árbol sintáctico a la 'metavariable' `$x`. El identificador `expr` es un
'especificador fragmento'; todas las posibilidades se enumeran en el [capitulo avanzado de macros][].
Al rodear el matcher con `$(...),*`, coincidirá con cero o más expresiones separadas por comas.

Ademas de la sintaxis especial de matchers, los tokens Rust que aparecen en un
matcher deben coincidir de manera exacta. Por ejemplo:

```rust
macro_rules! foo {
    (x => $e:expr) => (println!("modo X: {}", $e));
    (y => $e:expr) => (println!("modo Y: {}", $e));
}

fn main() {
    foo!(y => 3);
}
```

imprimirá

```text
modo Y: 3
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

El lado derecho de una regla macro es sintaxis Rust ordinaria, en su mayor
parte. Pero podemos insertar partes de sintaxis capturadas por el matcher. Del
ejemplo original:

```ignore
$(
    temp_vec.push($x);
)*
```

Cada expresión coincidente `$x` producirá una sola expresión `push` en la
expansión de la macro. La repetición se desarrolla en "lockstep" con la
repetición en el matcher (más sobre esto en un momento).

Debido a que `$x` ya fue marcado como una coincidencia con una expresión, no repetimos `:expr`
en el lado derecho. Además, no incluimos una coma separando como parte del
operador de repetición. En cambio, tenemos un punto y coma que termina dentro
del bloque repetido.

Otro detalle: la macro `vec` tiene *dos* pares de llaves en el lado
derecho. A menudo se combinan de este modo:

```ignore
macro_rules! foo {
    () => {{
        ...
    }}
}
```

Las llaves exteriores son parte de la sintaxis de `macro_rules!`. De hecho,
se puede utilizar `()` o `[]` en su lugar. Simplemente delimitan el lado
derecho como un todo.

Las llaves interiores son parte de la sintaxis expandida. Recuerda que la
macro `vec!` se utiliza en un contexto de expresión. Para escribir una
expresión con varias sentencias, entre ellas enlaces a variable, utilizamos un
bloque. Si la macro se expande a una sola expresión, no necesitas dicha
capa extra de llaves.

Hay que tener tambien en cuenta que nunca *declaramos* que la macro produce una expresión. De
hecho, esto no se determina hasta que usamos la macro como una expresión. Con
cuidado, se puede escribir una macro cuya expansión funcione en varios
contextos. Por ejemplo, la abreviatura de un tipo de datos podría ser válida
como una expresión o un patrón.

## Repetición

El operador de repetición sigue dos reglas principales:

1. `$(...)*` camina a través de una "capa" de repeticiones, para todos los `$nombre`s
    que contiene, al mismo paso, y
2. cada `$nombre` debe estar bajo al menos tantos `$(...)*` como los que fue comparado.
   Si esta bajo más, sera duplicado, según el caso.

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

Esa es la mayor parte de la sintaxis de los matcher. Todos los ejemplos utilizan
`$(...)*`, que coincide con "cero o más" elementos sintácticos.
Alternativamente, puedes escribir `$(...)+` que coincide con "uno o
más".  Ambas formas incluyen, opcionalmente, un separador, que puede ser
cualquier token excepto `+` o `*`.

Este sistema se basa en
"[Macro-by-Example](http://www.cs.indiana.edu/ftp/techreports/TR206.pdf)"
(PDF).

# Higiene

Algunos lenguajes implementan macros con sustitución de texto simple, lo que
trae como consecuencia diversos problemas. Por ejemplo, este programa C imprime `13` en
lugar de la esperada `25`.

```text
#define CINCO_VECES(x) 5 * x

int main() {
    printf("%d\n", CINCO_VECES(2 + 3));
    return 0;
}
```

Después de la expansión tenemos `5 * 2 + 3`, en donde la multiplicación tiene mayor
precedencia que la suma. Si has utilizado muchos macros en C, probablemente
conoces los idiomas estándar para evitar este problema, así como cinco o seis
otros. En Rust, no nos preocupamos por ello.

```rust
macro_rules! cinco_veces {
    ($x:expr) => (5 * $x);
}

fn main() {
    assert_eq!(25, cinco_veces!(2 + 3));
}
```

La metavariable `$x` se analiza como un nodo de expresión individual, y
mantiene su lugar en el árbol de sintaxis, incluso después de la sustitución.

Otro problema común en los sistemas de macro es la "captura de variable". Aquí hay
una macro C, utilizando [una extensión de GNU C][] para emular bloques de
expresión de Rust.

[una extensión de GNU C]: https://gcc.gnu.org/onlinedocs/gcc/Statement-Exprs.html

```text
#define LOG(msj) ({ \
    int estado = obtener_estado_log(); \
    if (estado > 0) { \
        printf("log(%d): %s\n", estado, msj); \
    } \
})
```

He aqui un caso de uso que va terriblemente mal:

```text
const char *estado = "reticulante acanaladuras";
LOG(estado)
```

Lo anterior se expande a

```text
const char *estado = "reticulante acanaladuras";
int estado = obtener_estado_log();
if (estado > 0) {
    printf("log(%d): %s\n", estado, estado);
}
```

La segunda variable llamada `estado` sobreescribe a la primera.
Esto es un problema porque la expresión de impresión (`printf`)
debe hacer referencia a ambas.

La macro Rust equivalente tiene el comportamiento deseado.

```rust
# fn obtener_estado_log() -> i32 { 3 }
macro_rules! log {
    ($msj:expr) => {{
        let estado: i32 = obtener_estado_log();
        if estado > 0 {
            println!("log({}): {}", estado, $msj);
        }
    }};
}

fn main() {
    let estado: &str = "reticulante acanaladuras";
    log!(estado);
}
```

Esta version funciona porque Rust tiene un [sistema de macros
higiénico][]. Cada expansión de macro ocurre en un "contexto de
sintaxis" distinto, y cada variable está asociada con el
contexto de sintaxis donde fue introducida. Es como si la
variable `estado` dentro `main` está pintado de un "color"
diferente de la variable `estado` dentro de la macro, y por lo
tanto no entran en conflicto.

[sistema de macros higiénico]: http://en.wikipedia.org/wiki/Hygienic_macro

El sistema de macros de Rust también restringe la capacidad de las macros para introducir nuevos enlaces en
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

En lugar de eso necesitas pasar el nombre de la variable en la invocación, de manera que sea estiquetado
con el contexto de sintaxis correcto.

```rust
macro_rules! foo {
    ($v:ident) => (let $v = 3);
}

fn main() {
    foo!(x);
    println!("{}", x);
}
```

Lo anterior es válido para enlaces `let` y etiquetas de bucle, pero no para [items][items]. Así que el siguiente código compila:

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

La expansión de una macro puede incluir más invocaciones a macro, incluyendo
invocaciones de la misma macro a ser expandida. Estas macros recursivas son
útiles para el procesamiento de entrada con estructura de árbol, como se
ilustra en esta (simplista) taquigrafía HTML:

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

# Depurando código de macro

Para ver los resultados de las macros en expansión, ejecuta
`rustc --pretty expanded`. La salida representa todo un crate, por lo que también
puede alimentar de nuevo a `rustc`, que a su vez producira mejores mensajes de
error que la compilación inicial. Es importante destacar que la salida de `--pretty
expanded` puede tener un significado diferente si varias variables del mismo
nombre (pero diferentes contextos sintácticos) están en juego en el mismo
ámbito. En este caso `--pretty expanded,hygiene` te dirá acerca de los
contextos de sintaxis.

`rustc` ofrece dos extensiones de sintaxis que ayudan con la depuración de
macros. Por ahora, son inestables y requieren puertas de características (feature gates).

* `log_syntax!(...)` imprimirá sus argumentos en la salida estándar, en tiempo
  de compilación, y se "expandira" a nada.

* `trace_macros!(true)` habilitara un mensaje compilador cada vez que una macro es
  expandida. Use `trace_macros!(false)` adelante en la expansión para apagarlo.

# Mas informacion

El [capitulo avanzado de macros][] entra en mas detalles acerca de la sintaxis de macros.
Tambien describe como compartir macros entre diferentes crates y modulos.

[capitulo avanzado de macros]: advanced-macros.html
