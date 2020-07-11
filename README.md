# El Lenguaje de Programación Rust

¡Bienvenido! Este libro te enseñará acerca del [Lenguaje de Programación Rust][rust].
Rust es un lenguaje de programación de sistemas enfocado en tres objetivos: seguridad, velocidad y concurrencia. Rust logra estos objetivos sin tener un recolector de basura, haciendolo util para un numero de casos de uso para los cuales otros lenguajes no son tan buenos: embeber en otros lenguajes, programas con requerimientos específicos de tiempo y espacio, escritura de código de bajo nivel, como controladores de dispositivo y sistemas operativos. Rust mejora por sobre los lenguajes actuales en este nicho a través de un numero de chequeos en tiempo de compilación que no incurren ninguna penalidad en tiempo de ejecución, eliminando al mismo tiempo las condiciones de carrera. Rust también implementa 'abstracciones con cero costo', abstracciones que se sienten como las de un lenguaje de alto nivel. Aun así, Rust permite control preciso tal y como un lenguaje de bajo nivel lo haria.

[rust]: http://rust-lang.org

“El Lenguaje de Programación Rust” esta dividido en siete secciones. Esta introducción es la primera. Después de esta:

* [Primeros Pasos][gs] - Configura tu maquina para el desarrollo en Rust.
* [Aprende Rust][lr] - Aprende programación en Rust a través de pequeños proyectos.
* [Rust Efectivo][er] - Conceptos de alto nivel para escribir excelente código Rust.
* [Sintaxis y Semántica][ss] - Cada pieza de Rust, dividida en pequenos pedazos.
* [Rust Nocturno][nr] - Características todavía no disponibles en builds estables.
* [Glosario][gl] - Referencia de los términos usados en el libro.
* [Investigación Académica][ar] - Literatura que influencio Rust.

[gs]: getting-started.html
[lr]: learn-rust.html
[er]: effective-rust.html
[ss]: syntax-and-semantics.html
[nr]: nightly-rust.html
[gl]: glossary.html
[ar]: academic-research.html

Después de leer esta introducción, dependiendo de tu preferencia, querrás leer ‘Aprende Rust’ o ‘Sintaxis y Semántica’: ‘Aprende Rust’ si quieres comenzar con un proyecto o ‘Sintaxis y Semántica’, si prefieres comenzar por lo más pequeño aprendiendo un único concepto detalladamente antes de moverte al siguiente. Abundantes enlaces cruzados conectan dichas partes.

### Contribuyendo

Los archivos fuente de los cuales este libro es generado pueden ser encontrados en Github:
[github.com/goyox86/elpr-sources](https://github.com/goyox86/elpr-sources)

## Una breve introducción a Rust

¿Es Rust un lenguaje en el cual estarías interesado? Examinemos unos pequeños ejemplos de código que demuestran algunas de sus fortalezas.

El concepto principal que hace único a Rust es llamado ‘pertenencia’ (‘ownership’). Considera este pequeño ejemplo:

```rust
fn main() {
    let mut x = vec!["Hola", "mundo"];
}
```

Este programa crea un una [variable][var] llamada `x`.El valor de esta variable es `Vec<T>`, un ‘vector’, que creamos a través de una [macro][macro] definida en la biblioteca estandar. Esta macro se llama `vec`, las macros son invocadas con un `!`. Todo esto siguiendo un principio general en Rust: hacer las cosas explícitas. Las macros pueden hacer cosas significativamente más complejas que llamadas a funciones, es por ello que son visualmente distintas. El `!` ayuda también al análisis sintáctico, haciendo la escritura de herramientas más fácil, lo cual es también importante.

Hemos usado `mut` para hacer `x` mutable: En Rust las variables son inmutables por defecto. Más tarde en este ejemplo estaremos mutando este vector.

Es importate mencionar que no necesitamos una anotación de tipos aquí: si bien Rust es estaticamente tipado, no necesitamos anotar el tipo de forma explicita. Rust posee inferencia de tipos para balancear el poder de el tipado estático con la verbosidad de las anotaciones de tipos.

Rust prefiere asignación de memoria desde la pila que desde el montículo: `x` es puesto directamente en la pila. Sin embargo, el tipo `Vec<T>` asigna espacio para los elementos del vector en el montículo. Si no estas familiarizado con esta distinción puedes ignorarla por ahora o echar un vistazo [‘La Pila y el Monticulo’][heap]. Rust como un lenguaje de programación de sistemas, te da la habilidad de controlar como la memoria es asignada, pero como estamos comenzando no es tan relevante.

[var]: variable-bindings.html
[macro]: macros.html
[heap]: the-stack-and-the-heap.html

Anteriormente mencionamos que la ‘pertenencia’ es nuevo concepto clave en Rust. En terminología Rust, `x` es el ‘dueño’ del vector. Esto significa que cuando `x` salga de ámbito, la memoria asignada a el vector sera liberada. Esto es hecho por el compilador de Rust de manera deterministica, sin la necesidad de un mecanismo como un recolector de basura. En otras palabras, en Rust, no haces llamadas a funciones como `malloc` y `free` explícitamente: el compilador determina de manera estática cuando se necesita asignar o liberar memoria, e inserta esas llamadas por ti. Errar es de humanos, pero los compiladores nunca olvidan.

Agreguemos otra línea a nuestro ejemplo:

```rust
fn main() {
    let mut x = vec!["Hola", "mundo"];

    let y = &x[0];
}
```

Hemos introducido otra variable, `y`. En este caso, `y` es una ‘referencia’ a el primer elemento de el vector. Las referencias en Rust son similares a los apuntadores en otros lenguajes, pero con chequeos de seguridad adicionales en tiempo de compilación. Las referencias interactuan con el sistema de pertenencia a través de el [‘prestamo’][borrowing] (‘borrowing’), ellas toman prestado a lo que apuntan, en vez de adueñarse de ello. La diferencia es que cuando la referencia salga de ámbito, la memoria subyacente no sera liberada. De ser ese el caso estaríamos liberando la misma memoria dos veces, lo cual es malo.

[borrowing]: references-and-borrowing.html

Agreguemos una tercera línea. Dicha línea luce inocente pero causa un error de compilación:

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

¡Uff! El compilador de Rust algunas veces puede proporcionar errores bien detallados y esta vez una de ellas. Como el error lo explica, mientras hacemos la variable mutable no podemos llamar a `push`. Esto es porque ya tenemos una referencia a un elemento del vector, `y`. Mutar algo mientras existe una referencia a ello es peligroso, porque podemos invalidar la referencia. En este caso en especifico, cuando creamos el vector, solo hemos asignado espacio para dos elementos. Agregar un tercero significaría asignar un nuevo segmento de memoria para todos los elementos, copiar todos los valores anteriores y actualizar el apuntador interno a esa memoria. Todo eso esta bien. El problema es que `y` no seria actualizado, generando un ‘puntero colgante’. Lo cual esta mal. Cualquier uso de `y` seria un error en este caso, y el compilador nos ha prevenido de ello.

Entonces, ¿cómo resolvemos este problema? Hay dos enfoques que podríamos tomar. El primero es hacer una copia en lugar de una referencia:

```rust
fn main() {
    let mut x = vec!["Hola", "mundo"];

    let y = x[0].clone();

    x.push("foo");
}
```

Rust tiene por defecto [semántica de movimiento][move], entonces si queremos hacer una copia de alguna data, llamamos el método `clone()`. En este ejemplo `y` ya no es una referencia a el vector almacenado en `x`, sino una copia de su primer elemento, `"Hola"`. Debido a que no tenemos una referencia nuestro `push()` funciona perfectamente.

[move]: ownership.html#move-semantics

Si realmente queremos una referencia, necesitamos otra opción: asegurarnos de que nuestra referencia salga de ámbito antes que tratamos de hacer la mutación. De esta manera:

```rust
fn main() {
    let mut x = vec!["Hola", "mundo"];

    {
        let y = &x[0];
    }

    x.push("foo");
}
```

Con el par adicional de llaves hemos creado un ámbito interno. `y` saldrá de ámbito antes que llamemos a `push()`, entonces no hay problema.

Este concepto de pertenencia no es solo bueno para prevenir punteros colgantes, sino un conjunto entero de problemas, como invalidación de iteradores, concurrencia y más.
