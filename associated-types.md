% Tipos Asociados

Los tipos asociados son una poderosa parte de el sistema de tipos de Rust. Estan relacionados a la idea de ‘familiad de tipos’, en otras palabras, la agrupacion de multiples tipos. Esa descripcion es una poco abstracta, es mejor que nos adentremos de una vez en un ejemplo. Si quieres escribir un trait `Grafo`, tienes dos tipos por encima de los cuales debes ser generico: el tipo de los nodos y el tipo de los vertices. Podrias escribir un trait, `Grafo<N, V>` como este:

```rust
trait Grafo<N, V> {
    fn tiene_vertice(&self, &N, &N) -> bool;
    fn vertices(&self, &N) -> Vec<V>;
    // etc
}
```

Si bien esto de alguna manera funciona, termina siendo un poco raro. Por ejemplo, cualquier funcion que quiera recibir un `Grafo` como parametro ahora _tambien_ necesita ser generica por sobre los tipos `N`odo y `V`ertice:

```rust,ignore
fn distancia<N, V, G: Grafo<N, V>>(grafo: &G, inicio: &N, fin: &N) -> u32 { ... }
```

Nuestro calculo de la distancia funciona sin tomar en cuenta nuestro tipo `Vertice`, en consecuencia la `V` en esta firma es simplemente una distraccion.

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

Sin necesidad de lidiar con el tipo `V`ertice aqui!

Echemos un vistazo con mayor detalle a todo esto.

## Definiendo tipos asociados

Construyamos ese trait `Grafo`. He aqui la definicion:

```rust
trait Graph {
    type N;
    type V;

    fn tiene_vertice(&self, &Self::N, &Self::N) -> bool;
    fn vertices(&self, &Self::N) -> Vec<Self::V>;
}
```

Simple. Los tipos asociados usan la palabra reservada `type`, y van dentro del cuerpo del trait en conjunto con las funciones.

Dichas declaraciones `type` pueden tener lo mismo que las funciones poseen. Por ejemplo si desearamos que nuestro tipo `N` implementase `Display`, de manera que pudiesemos imprimir los nodos, podriamos hacer lo siguiente:

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

Justo como cualquier otro trait, los traits que usan tipos asociados hacen uso de la palabra reservada `impl` para proporcionar implementaciones. A continuacion una implementacion simple de Grafo:

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

Esta tonta implementacion siempre retorna `verdadero` y un `Vec<Vertice>` vacio, pero te da una idea de como se implementa este tipo de traits con tipos asociados. Primero necesitamos tres `struct`s, una para el grafo, uno para el nodo, y una para el vertice. De haber tenido mas sentido usar un tipo diferente, tambien hubiese funcionado, ahora usaremos `struct`s para los tres.

Lo sigueinte es la linea `impl`, que es identica a la iplementacion de cualquier otro trait.

De aqui en adelante, usamos `=` para definir nuestros tipos asociados. El nombre que el trait usa va del lado izquierdo del `=`, y el tipo en concreto para el  que estamos implementando este trait va del lado derecho. Finalmente, podemos usar tipos concretos en nuestras declaraciones de funcion.

## Objetos trait con tipos asociados

Hay otra sintaxis de la cual debemos hablar: objetos trait. Si desearamos crear un objeto trait a partir de un tipo asociado de esta manera:

```rust,ignore
# trait Grafo {
#     type N;
#     type V;
#     fn tiene_vertice(&self, &Self::N, &Self::N) -> bool;
#     fn vertices(&self, &Self::N) -> Vec<Self::E>;
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

Obtendras dos errores:

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

We can’t create a trait object like this, because we don’t know the associated
types. Instead, we can write this:

```rust
# trait Grafo {
#     type N;
#     type V;
#     fn has_edge(&self, &Self::N, &Self::N) -> bool;
#     fn edges(&self, &Self::N) -> Vec<Self::E>;
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

La sintaxis `N=Nodo` nos permite crear un tipo concreto, `Nodo`, para el parametro de tipo `N`. Lo mismo con `V=Vertice`. De no haber propocionado esta restriccion, no hubieramos podido determinar contra cual `impl` deberiamos usar el objeto trait.
