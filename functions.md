% Funciones

Todo programa Rust posee al menos una función, la función `main`:

```rust
fn main() {
}
```

Esta es la declaración de función mas simple posible. Como hemos mencionado anteriormente, `fn` dice ‘esto es una función’, seguido del nombre, paréntesis debido a que esta función no recibe ningún argumento, y luego llaves para indicar el cuerpo de la función. He aquí una función llamada `foo`:

```rust
fn foo() {
}
```

Entonces, que hay acerca de recibir argumentos? A continuación una función que imprime un numero:

```rust
fn imprimir_numero(x: i32) {
    println!("x es: {}", x);
}
```

Acá un programa completo que hace uso de `imprimir_numero`:

```rust
fn main() {
    imprimir_numero(5);
}

fn imprimir_numero(x: i32) {
    println!("x es: {}", x);
}
```

Como podrás ver, los argumentos de función trabajan de manera muy similar a las declaraciones `let`: agregas un tipo a el nombre del argumento, después de dos puntos.

He aquí un programa completo que suma dos números y luego los imprime:

```rust
fn main() {
    imprimir_suma(5, 6);
}

fn imprimir_suma(x: i32, y: i32) {
    println!("la suma es: {}", x + y);
}
```

Los argumentos son separados por una coma, en ambos casos, cuando llamas a la función así como cuando la declaras.

A diferencia de `let`, _debes_ declarar los tipos de los argumentos. Lo siguiente, no compilara:

```rust,ignore
fn imprimir_suma(x, y) {
    println!("suma es: {}", x + y);
}
```

Obtendrías el siguiente error:

```text
expected one of `!`, `:`, or `@`, found `)`
fn imprimir_suma(x, y) {
```

Esto es una deliberada decisión de diseño. Aunque la inferencia del programa completo es possible, los lenguajes que la poseen, como Haskell, a menudo sugieren que documentar tus tipos de manera explicita es una buena practica. Nosotros coincidimos en que obligar a las funciones a declarar los tipos al mismo tiempo permitiendo inferencia dentro de la función es un maravilloso punto medio entre inferencia completa y la ausencia de inferencia.

Que hay acerca de retornar un valor? A continuación una función que suma uno a un entero:

```rust
fn suma_uno(x: i32) -> i32 {
    x + 1
}
```

La funciones en Rust retornan exactamente un valor, y el tipo es declarado después de una ‘flecha’, que es un guión (`-`) seguido por un signo mayor-que (`>`). La ultima linea de una función determina lo que esta retorna. Notaras la ausencia de un punto y coma aquí. De agregarlo:

```rust,ignore
fn suma_uno(x: i32) -> i32 {
    x + 1;
}
```

Obtendríamos un error:

```text
error: not all control paths return a value
fn suma_uno(x: i32) -> i32 {
     x + 1;
}

help: consider removing this semicolon:
     x + 1;
          ^
```

Lo anterior revela dos cosas interesantes acerca de Rust: es un lenguaje basado en expresiones, y los puntos y coma son diferentes a los de otros lenguajes basados en ‘llaves y puntos y coma’. Estas dos cosas están relacionadas.

## Expresiones vs. Sentencias

Rust en un lenguaje principalmente basado en expresiones. Existen solo dos tipos de sentencias, todo lo demás es una expresión.

Entonces, cual es la diferencia? Las expresiones retornan un valor, y las sentencias no. Es por ello que terminamos con el error ‘not all control paths return a value’ aquí: la sentencia `x + 1;` no retorna ningún valor.Hay dos tipos de sentencias en Rust: ‘sentencias de declaración’ and ‘sentencias de expresión’.  Todo lo demás es una expresión. Hablemos primero de las sentencias de declaración.

En algunos lenguajes, los enlaces a variable pueden ser escritos como expresiones, no solo sentencias. Como en Ruby:

```ruby
x = y = 5
```

En Rust, sin embargo, el uso de `let` para introducir un enlace a variable _no es_ una expresión. Lo siguiente producirá un error en tiempo de compilación:

```ignore
let x = (let y = 5); // expected identifier, found keyword `let`
```

El compilador nos dice que estaba esperando el comienzo de una expresión, y un `let` puede ser solo una sentencia, no una expresión.

Nota que la asignación a una variable que ha sido previamente asignada  (e.j. `y = 5`) es una expresión, aun así su valor no es particularmente util. A diferencia de otros lenguajes en los que una asignación es evaluada a el valor asignado  (e.j. `5` en ejemplo anterior), en Rust el valor de una asignación es una tupla vacía `()` debido a que el valor asignado puede tener [un solo dueño](ownership.html) y cualquier otro valor de retorno seria sorpresivo:

```rust
let mut y = 5;

let x = (y = 6);  // x posee el valor `()`, no `6`
```

El segundo tipo de sentencia en Rust es la *sentencia de expresión*. Su propósito es convertir cualquier expresión en una sentencia. En términos prácticos, la gramática de Rust espera que una sentencia sea seguida por mas sentencias. Esto significa que puedes usar puntos y coma para separar una expresión de otra. Esto causa que Rust luzca muy similar a esos lenguajes en los cuales se requiere un punto y coma al final de cada linea, de hecho, observaras puntos y coma al final de casi todas las lineas de Rust que veras.

Entonces, Cual es la excepción que nos hace decir "casi"? Ya lo has visto en el código anterior:

```rust
fn suma_uno(x: i32) -> i32 {
    x + 1
}
```

Nuestra función clama retornar un `i32`, pero con un punto y coma, retornaría `()`. Rust determina que esto no es probablemente lo que queremos, y sugiere la remoción del punto y coma en el error que vimos anteriormente.

## Retornos tempranos

Pero que hay acerca de los retornos tempranos? Rust proporciona una palabra reservada para ello, `return`:

```rust
fn foo(x: i32) -> i32 {
    return x;

    // nunca alcanzaremos este código!
    x + 1
}
```

Usar un `return` como la ultima linea de una función es valido, pero es considerado pobre estilo:

```rust
fn foo(x: i32) -> i32 {
    return x + 1;
}
```

La definición previa sin el `return` puede lucir un poco extraña si nunca has trabajado con un lenguaje basado en expresiones, pero esta se vuelve intuitiva con el tiempo.

## Funciones divergentes

Rust tiene una sintaxis especial para las ‘funciones divergentes’, que son funciones que no retornan:

```rust
fn diverge() -> ! {
    panic!("Esta función nunca retorna!");
}
```

`panic!` es una macro, similar a la `println!()` que ya hemos visto. A diferencia de  `println!()`, `panic!()` causa que el hilo de ejecución actual termine de manera abrupta mostrando el mensaje proporcionado.

Debido a que esta función causara una salida abrupta, esta nunca retornara, es por ello que posee el tipo ‘`!`’, que se lee ‘diverge’. Una función divergente puede ser usada como cualquier tipo:

```should_panic
# fn diverge() -> ! {
#    panic!("Esta función nunca retorna!");
# }
let x: i32 = diverge();
let x: String = diverge();
```

## Apuntadores a función

También podemos crear enlaces a variables que apunten a funciones:

```rust
let f: fn(i32) -> i32;
```

`f` es un enlace a variable que apunta a una función que toma un `i32` como argumento y retorna un `i32`. Por ejemplo:

```rust
fn mas_uno(i: i32) -> i32 {
    i + 1
}

// sin inferencia de tipos
let f: fn(i32) -> i32 = mas_uno;

// inferencia de tipos
let f = mas_uno;
```

Podemos entonces hacer uso de `f` para llamar a la función:

```rust
# fn mas_uno(i: i32) -> i32 { i + 1 }
# let f = mas_uno;
let seis = f(5);
```
