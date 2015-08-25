% Pertenencia

Esta guía es una de las tres presentando el sistema de pertenencia de Rust. Este es una de las características mas únicas e irresistibles de Rust, con la que desarrolladores Rust deben estar familiarizados. A través de la pertenencia es como Rust logra su objetivo mas importante, la seguridad. Existen unos pocos conceptos distintos, cada uno con su propio capitulo:


* pertenencia, la que lees actualmente
* [préstamo][borrowing], y su característica asociada ‘referencias’
* [tiempo de vida][lifetimes], un concepto avanzado del préstamo

Estos tres capítulos están relacionados, en orden. Necesitaras los tres para entender completamente el sistema de pertenencia de Rust.

[borrowing]: references-and-borrowing.html
[lifetimes]: lifetimes.html

# Meta

Antes de entrar en detalle, dos notas importantes acerca del sistema de pertenencia.

Rust tiene foco en seguridad y velocidad. Rust logra esos objetivos a travez de muchas ‘abstracciones de cero costo’, lo que significa que en Rust, las abstracciones cuestan tan poco como sea posible para hacerlas funcionar. El sistema de pertenencia es un ejemplo primordial de una abstracción de cero costo. Todo el análisis del que estaremos hablando en la presente guía es _llevado a cabo en tiempo de compilación. No pagas ningún costo en tiempo de ejecución por ninguna de estas facilidades.

Sin embargo, este sistema tiene cierto costo: la curva de aprendizaje. Muchos usuarios nuevos Rust experimentan algo que nosotros denominamos ‘pelear con el comprobador de préstamo’ (‘fighting with the borrow checker’), situación en la cual el compilador de Rust se rehusa a compilar un programa el cual el autor piensa valido. Esto ocurre con frecuencia debido a que el modelo mental del programador acerca de como funciona la pertenencia no concuerda con las reglas actuales implementadas en Rust. Probablemente tu experimentes cosas similares al comienzo. Sin embargo, hay buenas noticias: otros desarrolladores Rust experimentados  reportan que una vez que trabajan con las reglas del sistema de pertenencia por un periodo de tiempo, pelean cada vez menos con el comprobador de préstamo.

Con eso en mente, aprendamos acerca de la pertenencia.

# Pertenencia

Los [Bindings a variable][bindings] poseen una propiedad en Rust: Estos ‘poseen pertenencia’ sobre lo que están asociados. Lo que se traduce a que cuando un binding a variable sale de ámbito, Rust libera los recursos asociados a este. Por ejemplo:

```rust
fn foo() {
    let v = vec![1, 2, 3];
}
```

Cuando `v` entra en ámbito, un nuevo [`Vec<T>`][vect] es creado. En este caso el vector también asigna algo de memoria desde el [montículo][heap], para los tres elementos. Cuando `v` sale de ámbito al final de `foo()`, Rust limpiara todo lo relacionado al vector, incluyendo la memoria asignada desde el montículo. Esto ocurre de manera determinista, al final de el ámbito.

[vect]: ../std/vec/struct.Vec.html
[heap]: the-stack-and-the-heap.html
[bindings]: variable-bindings.html

# Semantica de movimiento

Hay algo mas sutil acá, Rust se asegura que solo exista _exactamente un_ binding a cualquier recurso en particular. Por ejemplo, si tenemos un vector, podemos asignarlo a otro binding a variable:


```rust
let v = vec![1, 2, 3];

let v2 = v;
```

Pero, si intentamos usar `v` después, obtenemos un error:


```rust,ignore
let v = vec![1, 2, 3];

let v2 = v;

println!("v[0] es: {}", v[0]);
```

El error luce como este:

```text
error: use of moved value: `v` (uso de valor movido: `v`)
println!("v[0] es: {}", v[0]);
                        ^
```

Algo similar ocurre si definimos una función que tome pertenencia, y tratamos de usar algo después de habérselo pasado como argumento:


```rust,ignore
fn tomar(v: Vec<i32>) {
    // lo que sucede acá no es relevante
}

let v = vec![1, 2, 3];

tomar(v);

println!("v[0] es: {}", v[0]);
```

El mismo error: ‘use of moved value’ (uso de valor movido). Cuando transferimos la pertenencia a algo, decimos que hemos ‘movido’ la cosa a la cual nos estamos refiriendo. No necesitas ningún tipo de anotación especial para ello, es lo que Rust hace por defecto.

## Los detalles

La razón por la cual no podemos usar un binding a variable después de haberlo movido es sutil, pero importante. Cuando escribimos código como este:


```rust
let v = vec![1, 2, 3];

let v2 = v;
```

La primera linea asigna memoria para el objeto vector, `v`, y para la data que contiene. El objeto vector es entonces almacenado en la pila [pila][sh] y contiene un apuntador a el contenido (`[1, 2, 3]`) almacenado en el [monticulo][sh]. Cuando movemos `v` a `v2`, se crea una copia de dicho apuntador para `v2`. Todo esto significa que existirían dos apuntadores para el contenido del vector en el montículo. Lo cual viola las garantías de seguridad de Rust, introduciendo una condición de carrera. Es por ello que Rust prohibe el uso de `v` después que el movimiento ha sido llevado a cabo.


[sh]: the-stack-and-the-heap.html

Es importante destacar que algunas optimizaciones podrían remover la copia de los bytes en la pila, dependiendo de ciertas circunstancias. Así que puede no ser tan ineficiente a como luce inicialmente.


## Tipos `Copy`


Hemos establecido que cuando transferimos la pertenencia a otro binding a variable, no podemos usar el binding original. Sin embargo, existe un trait [trait][traits] que cambia este comportamiento, se llama `Copy`. No hemos discutido los traits (rasgos) todavía, pero por ahora puedes verlos como una anotación hecha a un tipo en particular la cual agrega comportamiento extra. Por ejemplo:


```rust
let v = 1;

let v2 = v;

println!("v es: {}", v);
```

En este caso, `v` es un `i32`, que implementa el trait `Copy`. Esto significa que, justo como en un movimiento, cuando asignamos `v` a `v2`, una copia de la data es hecha. Pero, a diferencia de lo que ocurre en un movimiento, podemos hacer uso de `v` después. Esto es debido a que un `i32` no posee apuntadores a data en ningún otro lugar, copiarlos significa copiado completo.

Todos los tipos primitivos implementan el trait `Copy` y su pertenencia no es movida como uno podría asumir, siguiendo las `reglas de pertenencia`. Como ejemplo, los siguientes dos pedazos de código solo compilan porque los tipos `i32` y `bool` implementan el trait `Copy`.

```rust
fn main() {
    let a = 5;

    let _y = doblar(a);
    println!("{}", a);
}

fn doblar(x: i32) -> i32 {
    x * 2
}
```

```rust
fn main() {
    let a = true;

    let _y = cambiar_verdad(a);
    println!("{}", a);
}

fn cambiar_verdad(x: bool) -> bool {
    !x
}
```

De haber tenido tipos que no implementasen el trait `Copy`, hubiésemos obtenido un error de compilación por tratar de usar un valor movido.


```text
error: use of moved value: `a`
println!("{}", a);
               ^
```

Discutiremos como hacer que tus propios tipos sean `Copy` en la sección de [traits][traits]

[traits]: traits.html

# Mas que pertenencia

Por supuesto, si necesitaremos devolver la pertenencia con cada función que escribiésemos:


```rust
fn foo(v: Vec<i32>) -> Vec<i32> {
    // hacer algo con v

    // devolviendo pertenencia
    v
}
```

Las cosas se volverían bastante tediosas. Se pone aun peor mientras tengamos mas cosas sobre las cuales queramos tener pertenencia:


```rust
fn foo(v1: Vec<i32>, v2: Vec<i32>) -> (Vec<i32>, Vec<i32>, i32) {
    // hacer algo con v1 y v2

    // devolviendo pertenencia, así como el resultado de nuestra función
    (v1, v2, 42)
}

let v1 = vec![1, 2, 3];
let v2 = vec![1, 2, 3];

let (v1, v2, answer) = foo(v1, v2);
```

Ugh! El tipo de retorno, la linea de retorno, y el llamado a la función se vuelven mucho mas complicados.

Por suerte, Rust ofrece una facilidad, el préstamo, facilidad que nos sirve para solucionar este problema.

Es el tópico de la siguiente sección!
