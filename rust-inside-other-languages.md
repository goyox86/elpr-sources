% Rust Dentro de Otros Lenguajes

Para nuestro tercer proyecto, elegiremos algo que demuestra una de las mayores fortalezas de Rust: la ausencia de un entorno de ejecución.

A medida que las organizaciones crecen, estas de manera progresiva, hacen uso de una multitud de lenguajes de programación. Diferentes lenguajes de programación poseen diferentes fortalezas y debilidades, y una arquitectura poliglota permite usar un lenguaje en particular donde sus fortalezas hagan sentido y otro lenguaje sea débil.

Un area muy común en donde muchos lenguajes de programación son débiles es el performance en tiempo de ejecución. Frecuentemente, usar un lenguaje que es lento, pero ofrece mayor productividad para el programador, es un equilibrio que vale la pena. Para ayudar a mitigar esto, dichos lenguajes proveen una manera de escribir una parte de tu sistema en C y luego llamar ese código como si hubiese sido escrito en un lenguaje de mas alto nivel. Esta facilidad es denominada ‘interfaz de funciones foráneas’ (‘foreign function interface’), comúnmente abreviando a ‘FFI’.

Rust posee soporte para FFI en ambas direcciones: puede llamar código en C de manera fácil, pero crucialmente puede ser _llamado_ tan fácilmente como C. Combinado con la ausencia de un recolector de basura y bajos requerimientos en tiempo de ejecución, Rust es un candidato para ser embebido dentro de otros lenguajes cuando necesites esos ooKmh extra.

Existe un [capitulo completo dedicado a FFI][ffi] y sus detalles en otra parte del libro, pero en este capitulo, examinaremos el uso particular de FFI con ejemplos en Ruby, Python, y JavaScript.

[ffi]: ffi.html

# El problema

Existen muchos problemas que podríamos haber escogido, pero elegiremos un ejemplo en el cual Rust tiene una ventaja clara por encima de otros lenguajes: computación numérica e hilos.

Muchos lenguajes, en honor a la consistencia, colocan números en el montículo, en vez de en la pila. Especialmente en lenguajes enfocados en programación orientada a objetos y el uso de un recolector de basura, la asignación de memoria desde el montículo es el comportamiento por defecto. Algunas veces optimizaciones pueden colocar ciertos números en la pila, pero en vez de confiar en un optimizador para realizar este trabajo, podríamos querer asegurarnos que siempre estemos usando numero primitivos en vez de algun tipo de objetos.

Segundo, muchos lenguajes poseen un  ‘bloqueo global del interprete’ (‘global interpreter lock’) (GIL), que limita la concurrencia en muchas situaciones. Esto es hecho en el nombre de la seguridad lo cual es un efecto positivo, pero limita la cantidad de trabajo que puede ser llevado a cabo de manera concurrente, lo cual es un gran negativo.

Para enfatizar estos 2 aspectos, crearemos un pequeño proyecto que usa estos dos aspectos en gran medida. Debido a que el foco del ejemplo es embeber Rust en otros lenguajes, en vez de el problema en si mismo, usaremos un ejemplo de juguete:

> Inicia diez hilos. Dentro de cada hilo, cuenta desde uno hasta cinco millones. Después que todos los hilos hayan finalizado, imprime "completado!".

He escogido cinco millones basado en mi computador en particular. He aquí un ejemplo de este código en Ruby:

```ruby
threads = []

10.times do
  threads << Thread.new do
    count = 0

    5_000_000.times do
      count += 1
    end
  end
end

threads.each { |t| t.join }
puts "completado!"
```

Intenta ejecutar este ejemplo, y escoge un numero que corra por unos segundos. Dependiendo en el hardware de tu computador, tendrás que incrementar o decrementar el numero.

En mi sistema, ejecutar este programa toma `2.156`. Si uso alguna tipo de herramienta de monitoreo de procesos, como `top`, puedo ver que solo usa un núcleo en mi maquina. El GIL presente haciendo su trabajo.

Si bien es cierto que este en un programa sintético, uno podría imaginar muchos problemas similares a este en el mundo real. Para nuestros propósitos, levantar unos pocos hilos y ocuparlos representa una especie de computación paralela y costosa.

# Una biblioteca Rust

Escribamos este problema en Rust. Primero, creemos un proyecto nuevo con Cargo:


```bash
$ cargo new embeber
$ cd embeber
```

Este programa es fácil de escribir en Rust:

```rust
use std::thread;

fn procesar() {
    let handles: Vec<_> = (0..10).map(|_| {
        thread::spawn(|| {
            let mut _x = 0;
            for _ in (0..5_000_000) {
                _x += 1
            }
        })
    }).collect();

    for h in handles {
        h.join().ok().expect("No se pudo unir un hilo!");
    }
}
```

Algo de esto debe lucir familiar a ejemplos anteriores. Iniciamos diez hilos, colectandolos en un vector `handles`. Dentro de cada hilo, iteramos cinco millones de veces, agregando uno a `_x` en cada iteración. Porque el sub-guion? Bueno, si lo removemos y luego compilamos:


```bash
$ cargo build
  Compiling embeber v0.1.0 (file:///Users/goyox86/Code/rust/embeber)
src/lib.rs:3:1: 16:2 warning: function is never used: `procesar`, #[warn(dead_code)] on by default
src/lib.rs:3 fn procesar() {
src/lib.rs:4     let handles: Vec<_> = (0..10).map(|_| {
src/lib.rs:5         thread::spawn(|| {
src/lib.rs:6             let mut x = 0;
src/lib.rs:7             for _ in (0..5_000_000) {
src/lib.rs:8                 x += 1
             ...
src/lib.rs:6:17: 6:22 warning: variable `x` is assigned to, but never used, #[warn(unused_variables)] on by default
src/lib.rs:6             let mut x = 0;
                             ^~~~~
```

La primera advertencia es debido es a consecuencia de estar construyendo una biblioteca. Si tuviéramos una prueba para esta función, la advertencia desaparecería. Pero por ahora nunca es llamada.

La segunda esta relacionada a `x` versus `_x`. Como producto de que efectivamente _no hacemos nada_ con `x` obtenemos una advertencia. Eso, en nuestro caso, esta perfectamente bien, puesto que queremos desperdiciar ciclos de CPU. Usando un sub-guión de prefijo eliminamos la advertencia.

Finalmente, hacemos join en cada uno de los hilos.

Hasta el momento, sin embargo, es una biblioteca Rust, y no expone nada que pueda ser llamado desde C. Si quisiéramos conectarla con otro lenguaje, en su estado actual, no funcionaria. Solo necesitamos hacer unos pequeños cambios para arreglarlo. Lo primero es modificar el principio de nuestro código:

```rust,ignore
#[no_mangle]
pub extern fn procesar() {
```

Debemos agregar un nuevo atributo, `no_mangle`. Cuando creamos una biblioteca Rust, este cambia el nombre de la función en la salida compilada. Las razones de esto escapan del alcance de este tutorial, pero para que otros lenguajes puedan saber como llamar a la función, debemos evitar que el compilador cambie el nombre en la salida compilada. Este atributo desactiva ese comportamiento.

El otro cambio es el `pub extern`. El `pub` significa que esta función puede ser llamada desde afuera de este modulo, y el `extern` dice que esta puede ser llamada desde C. Eso es todo! No muchos cambios.

La segunda cosa que necesitamos hacer es cambiar una configuración en nuestro  `Cargo.toml`. Agrega esto al final:


```toml
[lib]
name = "embeber"
crate-type = ["dylib"]
```

Estas lineas le informan a Rust que queremos compilar nuestra biblioteca en una biblioteca dinámica estándar. Rust compila un ‘rlib’, un formato especifico de Rust.

Ahora construyamos el proyecto:


```bash
$ cargo build --release
   Compiling embeber v0.1.0 (file:///Users/goyox86/Code/rust/embeber)
```

Hemos elegido `cargo build --release`, lo cual construye el proyecto con optimizaciones. Queremos que sea lo mas rápido posible! Puedes encontrar la salida de la biblioteca en `target/release`:


```bash
$ ls target/release/
build  deps  examples  libembeber.dylib native
```

Esa `libembeber.dylib` es nuestra biblioteca de ‘objetos compartidos’. Podemos usar esta biblioteca como cualquier biblioteca de objetos compartido escrita en C! Como nota, esta podría ser `libembeber.so` o `libembeber.dll`, dependiendo la plataforma.

Ahora que tenemos nuestra biblioteca Rust, usémosla desde Ruby.

# Ruby

Crea un archivo `embeber.rb` dentro de nuestro proyecto, y coloca esto dentro:

```ruby
require 'ffi'

module Hola
  extend FFI::Library
  ffi_lib 'target/release/libembeber.dylib'
  attach_function :procesar, [], :void
end

Hola.procesar

puts 'completado!'
```

Antes de que podamos ejecutarlo, necesitamos instalar la gema `ffi`:


```bash
$ gem install ffi # esto puede necesitar sudo
Fetching: ffi-1.9.8.gem (100%)
Building native extensions.  This could take a while...
Successfully installed ffi-1.9.8
Parsing documentation for ffi-1.9.8
Installing ri documentation for ffi-1.9.8
Done installing documentation for ffi after 0 seconds
1 gem installed
```

Finalmente, intentemos ejecutarlo:

```bash
$ ruby embeber.rb
completado!
$
```

Whoa, eso fue rápido! En mi sistema, tomo `0.086` segundos, a diferencia de los dos segundos que la version en Ruby puro. Analicemos este código Ruby:

```ruby
require 'ffi'
```

Primero necesitamos requerir la gema `ffi`. Nos permite interactuar con una biblioteca Rust como una biblioteca en C.


```ruby
module Hola
  extend FFI::Library
  ffi_lib 'target/release/libembeber.dylib'
```

El modulo `Hola` es usado para adjuntar las funciones nativas de la biblioteca compartida. Dentro, `extend`emos el modulo `FFI::Library` y luego llamamos el método `ffi_lib` para cargar nuestra biblioteca de objetos compartidos. Simplemente pasamos la ruta en la cual nuestra biblioteca esta almacenada, la cual, como vimos anteriormente, es `target/release/libembeber.dylib`.


```ruby
attach_function :procesar, [], :void
```

El método `attach_function` es proporcionado por la gema FFI. Es lo que conecta nuestra función `procesar()` en Rust a un método en Ruby con el mismo nombre. Debido a que `procesar()` no recibe argumentos, el segundo parámetro es un arreglo vacío, y ya que no retorna nada, pasamos `:void` como argumento final.

```ruby
Hola.procesar
```

Esta es la llamada a Rust. La combinación de nuestro modulo y la llamada a `attach_function` han configurado todo. Se ve como un método Ruby pero es en realidad código Rust!

```ruby
puts 'completado!'
```

Finalmente, y como requerimiento de nuestro proyecto, imprimimos `completado!`.

Eso es todo! Como hemos visto, hacer un puente entre los dos lenguajes es realmente fácil, y nos compra mucho performance.

A continuación, probemos Python!

# Python

Crea un archivo `embeber.py` en este directorio, y coloca esto en el:


```python
from ctypes import cdll

lib = cdll.LoadLibrary("target/release/libembeber.dylib")

lib.procesar()

print("completado!")
```

Aun mas fácil! Usamos `cdll` del modulo `ctypes`. Una llamada rápida a `LoadLibrary` después, y luego podemos llamar `procesar()`.

En mi sistema, toma `0.017` segundos. Rápidillo!

# Node.js

Node no es un lenguaje, pero es actualmente la implementación de Javascript dominante del lado del servidor.

Para hacer FFI con Node, primero necesitamos instalar la biblioteca:

```bash
$ npm install ffi
```

Después de que este instalada, podemos usarla:


```javascript
var ffi = require('ffi');

var lib = ffi.Library('target/release/libembeber', {
  'procesar': ['void', []]
});

lib.procesar();

console.log("completado!");
```

Luce mas parecido al ejemplo Ruby que al de Python. Usamos el modulo `ffi` para obtener acceso a `ffi.Library()`, la cual nos permite cargar nuestra biblioteca de objetos compartidos. Necesitamos anotar el tipo de retorno y los tipos de los argumentos de la función, que son `void` para el retorno y un arreglo vacío para representar ningún argumento. De allí simplemente llamamos a la función `procesar()` e imprimimos el resultado.

En my sistema, este ejemplo toma unos rápidos `0.092` segundos.


# Conclusion

Como puedes ver, las bases de hacer FFI son _muy_ fáciles. Por supuesto hay mucho mas que podríamos hacer aquí. Echa un vistazo al capitulo [FFI][ffi] para mas detalles.
