% Tipos Asociados

Los tipos asociados son una parte poderosa del sistema de tipos de Rust. Se relacionan con la idea de una ‘familia de tipos’, en otras palabras, la agrupación de multiples tipos. Esa descripción es una poco abstracta, es mejor que nos adentremos de una vez en un ejemplo. Si queremos escribir un trait `Grafo`, tenemos dos tipos por encima de los cuales debemos ser genéricos: el tipo de los nodos y el tipo de los vertices. Podríamos escribir un trait, `Grafo<N, V>` como este:

```rust
trait Grafo<N, V> {
    fn tiene_vertice(&self, &N, &N) -> bool;
    fn vertices(&self, &N) -> Vec<V>;
    // etc
}
```

Si bien esto de alguna manera funciona, termina siendo un poco raro. Por ejemplo, cualquier función que quiera recibir un `Grafo` como parámetro ahora también necesita ser genérica por sobre los tipos `N`odo y `V`ertice:

```rust,ignore
fn distancia<N, V, G: Grafo<N, V>>(grafo: &G, inicio: &N, fin: &N) -> u32 { ... }
```

Nuestro calculo de la distancia funciona sin tomar en cuenta nuestro tipo `Vértice`, en consecuencia la `V` en esta firma es simplemente una distracción.

Lo que realmente queremos expresar es que ciertos tipos `V`ertice y `N`odo vienen juntos para formar cada clase de `Grafo`. Podemos hacer esto con tipos asociados:

```rust
trait Grafo {
    type N;
    type V;

    fn tiene_vertice(&self, &Self::N, &Self::N) -> bool;
    fn vertices(&self, &Self::N) -> Vec<Self::V>;
    // etc
}
```

Ahora, nuestros clientes pueden abstraerse por encima de un determinado `Grafo`:

```rust,ignore
fn distancia<G: Grafo>(grafo: &G, inicio: &G::N, fin: &G::N) -> u32 { ... }
```

Sin necesidad de lidiar con el tipo `V`ertice!

Echemos un vistazo con mayor detalle a todo esto.

## Definiendo tipos asociados

Construyamos ese trait `Grafo`. He aquí la definición:

```rust
trait Graph {
    type N;
    type V;

    fn tiene_vertice(&self, &Self::N, &Self::N) -> bool;
    fn vertices(&self, &Self::N) -> Vec<Self::V>;
}
```

Simple. Los tipos asociados usan la palabra reservada `type`, y van dentro del cuerpo del trait en conjunto con las funciones.

Dichas declaraciones `type` pueden tener lo mismo que las funciones. Por ejemplo si deseáramos que nuestro tipo `N` implementase `Display`, de manera que pudiésemos imprimir los nodos, podríamos hacer lo siguiente:

```rust
use std::fmt;

trait Grafo {
    type N: fmt::Display;
    type V;

    fn tiene_vertice(&self, &Self::N, &Self::N) -> bool;
    fn vertices(&self, &Self::N) -> Vec<Self::V>;
}
```

## Implementando tipos asociados

Justo como cualquier otro trait, los traits que usan tipos asociados hacen uso de la palabra reservada `impl` para proporcionar implementaciones. A continuación una implementación simple de Grafo:

```rust
# trait Grafo {
#     type N;
#     type V;
#     fn tiene_vertice(&self, &Self::N, &Self::N) -> bool;
#     fn vertices(&self, &Self::N) -> Vec<Self::V>;
# }
struct Nodo;

struct Vertice;

struct MiGrafo;

impl Grafo for MiGrafo {
    type N = Nodo;
    type V = Vertice;

    fn tiene_vertice(&self, n1: &Nodo, n2: &Nodo) -> bool {
        true
    }

    fn vertices(&self, n: &Nodo) -> Vec<Vertice> {
        Vec::new()
    }
}
```

Esta tonta implementación siempre retorna `true` y un `Vec<Vertice>` vacío, pero te da una idea de como se implementa este tipo de traits con tipos asociados. Primero necesitamos tres `struct`s, una para el grafo, una para el nodo, y una para el vértice. De haber tenido mas sentido usar un tipo diferente, también hubiese funcionado, ahora usaremos `struct`s para los tres.

Lo siguiente es la linea `impl`, que es idéntica a la implementación de cualquier otro trait.

De aquí en adelante, usamos `=` para definir nuestros tipos asociados. El nombre que el trait usa va del lado izquierdo del `=`, y el tipo en concreto para el  que estamos implementando este trait va del lado derecho. Finalmente, podemos usar tipos concretos en nuestras declaraciones de función.

## Objetos trait con tipos asociados

Hay otra sintaxis de la cual debemos hablar: objetos trait. Si deseáramos crear un objeto trait a partir de un tipo asociado de esta manera:

```rust,ignore
# trait Grafo {
#     type N;
#     type V;
#     fn tiene_vertice(&self, &Self::N, &Self::N) -> bool;
#     fn vertices(&self, &Self::N) -> Vec<Self::V>;
# }
# struct Nodo;
# struct Vertice;
# struct MiGrafo;
# impl Grafo for MiGrafo {
#     type N = Node;
#     type V = Vertice;
#     fn tiene_vertice(&self, n1: &Nodo, n2: &Nodo) -> bool {
#         true
#     }
#     fn vertices(&self, n: &Nodo) -> Vec<Vertice> {
#         Vec::new()
#     }
# }
let grafo = MiGrafo;
let obj = Box::new(grafo) as Box<MiGrafo>;
```

Obtendríamos estos dos errores:

```text
error: the value of the associated type `V` (from the trait `main::Grafo`) must
be specified [E0191]
let obj = Box::new(grafo) as Box<Grafo>;
          ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~
24:44 error: the value of the associated type `N` (from the trait
`main::Grafo`) must be specified [E0191]
let obj = Box::new(grafo) as Box<Grafo>;
          ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

No podemos crear objetos trait de esta manera, puesto a que no conocemos los tipos asociados. En su lugar podríamos escribir:


```rust
# trait Grafo {
#     type N;
#     type V;
#     fn has_edge(&self, &Self::N, &Self::N) -> bool;
#     fn edges(&self, &Self::N) -> Vec<Self::V>;
# }
# struct Nodo;
# struct Vertice;
# struct MiGrafo;
# impl Grafo for MiGrafo {
#     type N = Nodo;
#     type V = Vertice;
#     fn tiene_vertice(&self, n1: &Nodo, n2: &Nodo) -> bool {
#         true
#     }
#     fn vertices(&self, n: &Nodo) -> Vec<Vertice> {
#         Vec::new()
#     }
# }
let grafo = MiGrafo;
let obj = Box::new(grafo) as Box<Grafo<N=Nodo, V=Vertice>>;
```

La sintaxis `N=Nodo` nos permite crear un tipo concreto, `Nodo`, para el parámetro de tipo `N`. Lo mismo con `V=Vertice`. De no haber proporcionado esta restricción, no hubiéramos podido determinar contra cual `impl` debe ser usado el objeto trait.
