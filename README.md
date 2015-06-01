## El Lenguage de Programacion Rust

Bienvenido! Este libro te ensenara acerca del [Lenguage de Programacion Rust][rust].
Rust es un lenguaje de programacion de sistemas enfocado en tres objetivos: seguridad, velocidad y concurrencia. Rust logra estos objetivos sin tener un recolector de basura, haciendolo util para un numero de casos de uso para los cuales otros lenguajes no son tan buenos: embebir en otros lenguajes, programas con requeriminetos especificos de tiempo y espacio, escritura de codigo de bajo nivel, como controladores de dispositivo y sistemas operativos. Rust mejora por sobre los lenguajes actuales en este nicho a traves de un numero de chequeos en tiempo de compilacion que no incurren ninguna penalidad en tiempo de ejecucion, eliminando al mismo tiempo las condiciones de carrera. Rust también implementa 'abstraciones con cero costo', abstracciones que se sienten como las de un lenguaje de alto nivel. Aun asi, Rust permite control preciso tal y como un lenguaje de bajo nivel lo haria.

[rust]: http://rust-lang.org

“The Lenguaje de Programacion Rust” esta dividido en siete secciones. Esta introduccion es la primera. Despues de esta:

* [Primeros Pasos][gs] - Configura tu maquina para el desarrollo en Rust.
* [Aprende Rust][lr] - Aprende programacion en Rust a traves de pequenos proyectos.
* [Rust Efectivo][er] - Conceptos de alto nivel para escribir excelente codigo Rust.
* [Sintaxis and Semantica][ss] - Cada pieza de Rust, dividida en pequenos pedazos.
* [Rust Nocturno][nr] - Caracteristicas todavia no disponiles en builds estables.
* [Gloario][gl] - Referencia de los terminos usados en el libro.
* [Investigacion Academica][ar] - Literatura que influencio Rust.

[gs]: getting-started.md
[lr]: learn-rust.html
[er]: effective-rust.html
[ss]: syntax-and-semantics.html
[nr]: nightly-rust.html
[gl]: glossary.html
[ar]: academic-research.html

Despues de leer esta introduccion, dependiendo de tu preferencia, querras leer ‘Aprende Rust’ o ‘Sintaxis y Semantica’: ‘Aprende Rust’ si quieres comenzar con un proyecto o ‘Sintaxis y Semantica’, si prefieres comenzar por lo mas pequeno aprendiendo un unico concepto detalladamente antes de moverte al siguiente. Abundantes enlaces cruzados conectan dichas partes.

### Contribuyendo

Los archivos fuente de los cuales este libro es generado pueden ser encontrados en Github:
[github.com/rust-lang/rust/tree/master/src/doc/trpl](https://github.com/rust-lang/rust/tree/master/src/doc/trpl)

## Una breve introduccion a Rust

Es Rust un lenguaje en el cual estarias interesado? Examinemos unos pequenos ejemplo de codigo que demuestran algunas de sus fortalezas.

El concepto principal que hace unico a Rust es llamado ‘pertenencia’. Considera este pequeno ejemplo:

```rust
fn main() {
    let mut x = vec!["Hola", "mundo"];
}
```

Este programa crea un una [variable][var] llamada `x`.El valor de esta variable es `Vec<T>`, un ‘vector’, que creamos a traves de una [macro][macro] definida en la biblioteca estandar. Esta macro se llama `vec`, las macros son invocadas con un `!`. Todo esto siguiendo un principio general en Rust: hacer las cosas explicitas. Las macros pueden hacer cosas significativamente mas complejas que llamadas a funciones, es por ello que son visualmente distintas. El `!` also ayuda tambien al analisis sintactico, haciendo la escritura de herramientas mas facil, lo cual es tambien importante.

Hemos usado `mut` para hacer `x` mutable: En Rust las variable son immutables por defecto. Mas tarde en este ejemplo estaremos mutando este vector.

Es importate mencionar que no necesitamos una anotacion de tipos aqui: si bien Rust es estaticamente tipado, no necesitamos anotar el tipo de forma explicita. Rust posee inferencia de tipos para balancear el poder de el tipado estatico con la verbosidad de las anotaciones de tipos.

Rust prefiere asignacion en la pila que asignacion en el monticulo: `x` es puesto directamente en la pila. Sin embargo, el tipo `Vec<T>` asigna espacio para los elementos del vector en el monticulo. Si no estas familiarizado con esta distincion puedes ignorarla por ahora o echar un vistazo [‘La Pila y el Monticulo’][heap]. Rust como un lenguaje de programacion de sistemas, te da la habilidad de controlar como la memoria es asignada, pero como estamos comenzando no es tan relevante. 

[var]: variable-bindings.html
[macro]: macros.html
[heap]: the-stack-and-the-heap.html

Anteriormente mencionamos que la ‘pertenencia’ es nuevo concepto clave en Rust. En terminologia Rust, `x` es el ‘dueno’ del vector. Esto significa que cuando `x` salga de ambito, la memoria asignada a el vector sera liberada. Esto es hecho por el compilador de Rust de manera deterministica, sin la necesidad de un mecanismo como un recolector de basura. En otras palabras, en Rust, no haces llamadas a funciones como `malloc` y `free` explicitamente: el compilador determina de manera estatica cuando se necesita asignar o liberar memoria, e inserta esas llamadas por ti. Errar es de humanos, pero los compiladores nunca olvidan.

Agreguemos otra linea a nuestro ejemplo:

```rust
fn main() {
    let mut x = vec!["Hola", "mundo"];

    let y = &x[0];
}
```

Hemos introducido otra variable, `y`. En este caso, `y` es una ‘referencia’ a el primer elemento de el vector. Las referencias en Rust son similares a los apuntadores en otros lenguajes, pero con chequeos de seguridad adicionales en tiempo de compilacion. Las referencias interactuan con el sistema de pertenencia a traves de el [‘prestamo’][borrowing], ellas toman prestado a lo que apuntan, en vez de aduenarse de ello. La diferencia es que cuando la referencia salga de ambito, la memoria subyacente no sera liberada. De ser ese el caso estariamos liberando la misma memoria dos veces, lo cual es malo.

[borrowing]: references-and-borrowing.html

Agreguemos una tercera linea. Dicha linea luce inocente pero causa un eror de compilacion:

```rust,ignore
fn main() {
    let mut x = vec!["Hola", "mundo"];

    let y = &x[0];

    x.push("foo");
}
```

`push` es un metodo en los vectores que agrega un elemento al final del vector. Cuando tratamos de compilar el programa obtenemos un error:

```text
error: cannot borrow `x` as mutable because it is also borrowed as immutable
    x.push("foo");
    ^
note: previous borrow of `x` occurs here; the immutable borrow prevents
subsequent moves or mutable borrows of `x` until the borrow ends
    let y = &x[0];
             ^
note: previous borrow ends here
fn main() {

}
^
```

Uff! El compilador de Rust algunas veces puede proporcionar errores bien detallados y esta vez una de ellas. Como el error lo explica, mientras hacemos la variable mutable no podemos llamar a `push`. Esto es porque ya tenemos una referencia a un elemento del vector, `y`. Mutar algo mientras existe una referencia a ello es peligroso, porque podemos invalidar la referencia. En este caso en especifico, cuando creamos el vector, solo hemos asignado espacio para dos elementos. Agregar un tercero significaria asignar un nuevo segmento de memoria para todos los elementos, copiar todos los valores anteriores y actualizar el apuntador interno a esa memoria. Todo eso esta bien. El problema es que `y` no seria actualizado, generando un ‘puntero colgante’. Lo cual esta mal. Cualquier uso de `y` seria un error en este caso, y el compilador nos ha prevenido de ello.

Entonces, como resolvermos este problema? Hay dos enfoques que podriamos tomar. El primero es hacer una copia en lugar de una referencia:

```rust
fn main() {
    let mut x = vec!["Hola", "mundo"];

    let y = x[0].clone();

    x.push("foo");
}
```

Rust tiene por defecto [semantica de movimiento][move], entonces si queremos hacer una copia de alguna data, llamamos el metodo `clone()`. En este ejemplo `y` ya no es una referencia a el vector almacenado en `x`, sino una copia de su primer elemento, `"Hola"`. Debido a que no tenemos una referencia nuestro `push()` funciona perfectamente.

[move]: ownership.html#move-semantics

Si realmente queremos una referencia, necesitamos otra opcion: asegurarnos de que nuestra referencia salga de ambito antes que tratamos de hacer la mutacion. De esta manera:

```rust
fn main() {
    let mut x = vec!["Hola", "mundo"];

    {
        let y = &x[0];
    }

    x.push("foo");
}
```

Con el par adicional de llaves hemos creado un ambito interno. `y` saldra de ambito antes que llamemos a `push()`, entonces no hay problema.

Este concepto de pertenencia no es solo bueno para prevenir punteros colgantes, sino un conjunto entero de problemas, como invalidacion de iteradores, concurrencia y mas.

