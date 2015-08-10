% Funciones

Todo programa Rust posee al menos una funcion, la funcion `main`:

```rust
fn main() {
}
```

Esta es la declaracion de funcion mas simple posible. Como hemos mencionado anteriormente, `fn` dice ‘esto es una funcion’, seguido del nombre, algugos parentesis debido a que esta funcion no recibe ningun argumento, y luego llaves para indicar el cuerpo de la funcion. He aqui una funcion llamada `foo`:

```rust
fn foo() {
}
```

Entonces, que acerca de acerca de recibir argumentos? A continuacion una funcion que imprime un numero:

```rust
fn imprimir_numero(x: i32) {
    println!("x es: {}", x);
}
```

Aca un programa completo que hace uso de `imprimir_numero`:

```rust
fn main() {
    imprimir_numero(5);
}

fn imprimir_numero(x: i32) {
    println!("x es: {}", x);
}
```

Como podras ver los argumentos de funcion trabajan de manera muy similar a las declaraciones `let`: agregas un tipo a el nombre del argumento, despues de dos puntos.

He aqui un programa completo que suma dos numeros y luego los imprime

```rust
fn main() {
    imprmir_suma(5, 6);
}

fn imprmir_suma(x: i32, y: i32) {
    println!("suma es: {}", x + y);
}
```

Separas los argumentos con una coma, en ambos casos, cuando llamas a la funcion as como cuando la declaras.

A diferencia de `let`, _debes_ declarar los tipos de los argumentos. Lo siguiente no funciona:

```rust,ignore
fn imprmir_suma(x, y) {
    println!("suma es: {}", x + y);
}
```

Obtienes el siguiente error:

```text
expected one of `!`, `:`, or `@`, found `)`
fn imprmir_suma(x, y) {
```

Esto es una deliberada desicion de diseno. Aunque la inferencia del programa completo es possible, los lenguajes que la poseeen, como Haskell, a menudo sugieren que documentar tus tipos de manera explicita es un best-practice. Nosotros coincidimos en que obligar a las funciones a declarar los tipos al mismo tiempo perimitiendo inferencia dentro de la funcion es un maravilloso punto medio entre inferencia completa y la ausencia de inferencia.

Que hay acerca de retornar un valor? Acontinuacion una funcion que agrega uno a un entero:

```rust
fn suma_uno(x: i32) -> i32 {
    x + 1
}
```

La funciones en Rust exactamente un valor, y se declara el tipo despues de una ‘flecha’, que es un guion (`-`) seguido por un signo mayor-que (`>`). La ultima linea de una funcion determina lo que esta retorna. Notaras la ausencia de un punto y coma aqui. De agregarlo:

```rust,ignore
fn suma_uno(x: i32) -> i32 {
    x + 1;
}
```

Obtrendriamos un error:

```text
error: not all control paths return a value
fn suma_uno(x: i32) -> i32 {
     x + 1;
}

help: consider removing this semicolon:
     x + 1;
          ^
```

Lo amterior revela dos cosas interesantes acerca de Rust: es un lenguaje basado en expresiones, y los puntos y coma son diferentes a los de otros lenguages basados en ‘llaves y puntos y coma’. Estas dos cosas estan relacionandas.

## Expresiones vs. Sentencias

Rust en un lenguaje principalmente basado en expresiones. Existen solo dos tipos de sentencias, todo lo demas es una expresion.

Entonces, cual es la diferencia? Las expresiones retornan un valor, y las sentencias no. Es por ello que terminamos con el error ‘not all control paths return a value’ aqui: la sentencia `x + 1;` no retorna ningun valor.Hay dos tipos de sentencias en Rust: ‘sentencias de declaracion’ and ‘sentencias de expression’.  Todo lo demas es una expresion. Hablemos primero de las sentencias de declaracion.

En algunos lenguajes, los enlaces a variable pueden ser escritos como epxresiones, no solo sentencias. Como en Ruby:

```ruby
x = y = 5
```

En Rust, sin embargo, el uso de `let` para introducir un enlace a variable  _no es_ una expresion. Lo siguiente producira un error en timepo de compilacion:

```ignore
let x = (let y = 5); // expected identifier, found keyword `let`
```

El compilador nos dice que estaba esperando el comienzo de una expresion, y un `let` puede ser solo una sentencia, no una expresion.

Nota que la asignacion a una variable que ha sido previamente asignada  (e.j. `y = 5`) es aun una expresion, aun asi su valor no es particularmente util. A diferencia de otros lenguajes en los que una asignacion es evaluada a el valor asignado  (e.j. `5` en ejemplo anterior), en Rust el valor de una asignacion es una tupla vacia `()` debido a que el velor asignado puede tener [un solo dueno](ownership.html) y cualquier otro valor de retorno seria muy sorpresivo:

```rust
let mut y = 5;

let x = (y = 6);  // x posee el valor `()`, no `6`
```

El segundo tipo de sentencia en Rust es la *sentencia de expresion*. Su proposito es convertir cualquier expresion en una sentencia. En terminos practicos, la gramatica de Rust espera que una sentencia sea seguida por mas sentencias. Esto significa que puedes usar puntos y coma para separar una expresion de otra. Esto causa que Rust luzca muy similar a esos lenguajes en los cuales se requiere un punto y coma al final de cada linea, de hecho, veras puntos y coma al final de casi todas las lineas de Rust que veras.

Cual es la excepcion que nos hace decir "casi"? Ya lo has visto en este codigo:

```rust
fn suma_uno(x: i32) -> i32 {
    x + 1
}
```

Nuestra funcion clama retornar un `i32`, pero con un punto y coma, retornaria `()`. Rust determina que esto no es probablemente lo que queremos, y sugiere la remocion del punto y coma en el error que vimos anteriormente.

## Retornos tempranos

Pero que acerca de los retornos tempranos? Rust proporciona una palabra reservada para ello, `return`:

```rust
fn foo(x: i32) -> i32 {
    return x;

    // nunca alcanzaremos este codigo!
    x + 1
}
```

Usar un `return` como la ultima linea de una funcion funciona, pero es cosiderado, pobre estilo:

```rust
fn foo(x: i32) -> i32 {
    return x + 1;
}
```

La definicion previa sin el `return` puede lucir un poco extrana si nunca has trabajado con un lenguje basado en expresiones, pero esta se vuelve intuitiva con el tiempo.

## Funciones divergentes

Rust tiene una sintacis especial para las ‘funciones diverengentes’, que son funciones que no retornan:

```rust
fn diverge() -> ! {
    panic!("Esta funcion nunca retorna!");
}
```

`panic!` es una macro, similar a la `println!()` que ya hemos visto. A diferencia de  `println!()`, `panic!()` causa que el hilo de ejecucion actual termine de manera abrupta con el mensaje proporcionado.

Debido a que esta funcion causara una salida abrupta, esta nunca retornara, es por ello que posee el tipo ‘`!`’, que se lee ‘diverge’. Una funcion divergente puede ser usada como cualquier tipo:

Because this function will cause a crash, it will never return, and so it has
the type ‘`!`’, which is read ‘diverges’. A diverging function can be used
as any type:

```should_panic
# fn diverge() -> ! {
#    panic!("Esta funcion nunca retorna!");
# }
let x: i32 = diverge();
let x: String = diverge();
```

## Apuntadores a funcion

Tambien podemos crear enlaces a variables que apunten a funciones:

```rust
let f: fn(i32) -> i32;
```

`f` es un enlace a variable que apunta a una funcion que toma un `i32` como argumento y retorna un `i32`. Por ejemplo:

```rust
fn mas_uno(i: i32) -> i32 {
    i + 1
}

// sin inferencia de tipos
let f: fn(i32) -> i32 = mas_uno;

// inferencia de tipos
let f = mas_uno;
```

Podemos entonces hacer uso de `f` para llamar a la funcion:

```rust
# fn mas_uno(i: i32) -> i32 { i + 1 }
# let f = mas_uno;
let seis = f(5);
```
