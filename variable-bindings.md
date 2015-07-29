% Enlaces a Variable

Virtualmente cualquier programa no-'Hola Mundo’ usa *enlaces a variables*. Dichos enlaces a variables lucen así:

```rust
fn main() {
    let x = 5;
}
```

Colocar `fn main() {` en cada ejemplo es un poco tedioso, así que en el futuro lo omitiremos. Si estas siguiendo paso a paso, asegurate de editar tu función `main()`, en lugar de dejarla por fuera. De otro modo obtendrás un error.

En muchos lenguajes, esto es llamado una *variable*, pero los enlaces a variable de Rust tienen un par de trucos bajo la manga. Por ejemplo el lado izquierdo de una expresión `let` es un ‘[patron][pattern]’, no un simple nombre de variable. Esto se traduce en que podemos hacer cosas como:

```rust
let (x, y) = (1, 2);
```

Después de que esta expresión es evaluada, `x` sera uno, y `y` sera dos. Los patrones son realmente poderosos, y tienen [su propia sección][pattern] en el libro. No necesitamos esas facilidades por ahora, solo mantengamos esto en nuestras mentes mientras avanzamos.

[pattern]: patterns.html

Rust es un lenguaje estáticamente tipificado, lo que significa que especificamos nuestros tipos por adelantado, y estos son chequeados en tiempo de compilación. Entonces, porque nuestro primer ejemplo compila? Bueno, Rust tiene esta cosa llamada ‘inferencia de tipos’. Si puede determinar el tipo de algo, Rust no requiere que escribas el tipo.

Podemos agregar el tipo si lo deseamos. Los tipos vienen después de dos puntos (`:`):

```rust
let x: i32 = 5;
```

Si te pidiera leer esto en voz alta al resto de la clase, dirías “`x` es un enlace con el tipo `i32` y el valor `cinco`.”

En este caso decidimos representar `x` como un entero con signo de 32 bits. Rust posee muchos tipos de enteros primitivos diferentes. Estos comienzan con `i` para los enteros con signo y con `u` para los enteros sin signo. Los tamaños posibles para enteros son 8, 16, 32, y 64 bits.

En ejemplos futuros, podríamos anotar el tipo en un comentario. El ejemplo luciría así:

```rust
fn main() {
    let x = 5; // x: i32
}
```

Nota las similitudes entre la anotación y la sintaxis que usas con `let`. Incluir esta clase de comentarios no es idiomático en Rust, pero los incluiremos ocasionalmente para ayudarte a entender cuales son los tipos que Rust infiere.

Por defecto, los enlaces son *inmutables*. Este código no compilara:

```rust,ignore
let x = 5;
x = 10;
```

Dara como resultado el siguiente error:

```text
error: re-assignment of immutable variable `x`
     x = 10;
     ^~~~~~~
```

Si deseas que un enlace a variable sea mutable, puedes hacer uso de `mut`:


```rust
let mut x = 5; // mut x: i32
x = 10;
```

No hay una razón única por la cual los enlaces a variable son inmutables por defecto, pero podemos pensar acerca de ello a través de uno de los focos principales de Rust: seguridad. Si olvidas decir `mut`. el compilador lo notará, y te hará saber que has mutado algo que no tenias intención de mutar. Si los enlaces a variables fuesen mutables por defecto, el compilador no seria capaz de decirte esto. Si tenias la intención de mutar algo, entonces la solución es muy fácil: agregar `mut`.

Hay otras buenas razones para evitar estado mutable siempre que sea posible, pero estan fuera del alcance de esta guisa. En general, puedes frecuentemente evitar mutación explicita, y esto es preferible en Rust. Dicho esto, algunas veces, la mutación es justo lo que necesitas, es por ello que no esta prohibida.

Volvamos a los enlaces a variables. Los enlaces a variable en Rust poseen un aspecto mas que difiere de otros lenguajes: se requiere esten inicializados a un valor antes que puedas usarlos.

Pongamos esto a prueba. Cambia tu archivo `src/main.rs` para que luzca así: 

```rust
fn main() {
    let x: i32;

    println!("Hola, mundo!");
}
```

Puedes usar `cargo build` en la linea de comando para compilarlo. Obtendrás una advertencia pero aun así imprimirá "Hola, mundo!":


```text
   Compiling hola_mundo v0.0.1 (file:///home/tu/proyectos/hola_mundo)
src/main.rs:2:9: 2:10 warning: unused variable: `x`, #[warn(unused_variable)]
   on by default
src/main.rs:2     let x: i32;
                      ^
```

Rust nos advierte que nunca hacemos uso del enlace a variable, pero debido a que nunca la usamos, no hay peligro, no hay falta. Sin embargo, las cosas cambian si efectivamente intentamos usar `x`. Hagamos eso. Cambia tu programa para que luzca de la siguiente manera:

```rust,ignore
fn main() {
    let x: i32;

    println!("El valor de x es: {}", x);
}
```

Intenta compilarlo. Obtendrás un error:


```bash
$ cargo build
   Compiling hola_mundo v0.0.1 (file:///home/tu/proyectos/hola_mundo)
src/main.rs:4:39: 4:40 error: use of possibly uninitialized variable: `x`
src/main.rs:4     println!("El valor de x es: {}", x);
                                                    ^
note: in expansion of format_args!
<std macros>:2:23: 2:77 note: expansion site
<std macros>:1:1: 3:2 note: in expansion of println!
src/main.rs:4:5: 4:42 note: expansion site
error: aborting due to previous error
Could not compile `hello_world`.
```

Rust no te permitirá usar un valor que no haya sido inicializado previamente. A continuación hablemos de lo que le hemos agregado a `println!`.

Si incluyes un par de llaves (`{}`, algunas personas los llaman bigotes/moustaches...) en la cadena de caracteres a imprimir, Rust lo interpretara como una petición para interpolar alguna clase de valor. *La interpolación en cadenas de caracteres* es un termino en ciencias de la computación que significa "coloca esto dentro de la cadena de caracteres". Agregamos una coma, y luego `x`, para indicar que queremos que este sea el valor interpolado. La coma es usada para separar los argumentos que pasamos a las funciones y los macros, en el caso de pasar mas de uno.

Cuando usas las llaves, Rust intentará mostrar el valor de una forma que tenga sentido después de chequear su tipo. Si deseamos especificar un formato mas detallado, existen [un amplio numero de opciones disponible][format]. Por ahora, nos apegaremos al comportamiento por defecto: los números enteros no son muy complicados de imprimir.

[format]: ../std/fmt/index.html

