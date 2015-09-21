% Enumeraciones

Una enumeracion (`enum`) en Rust es un tipo que representa data que puede ser una de un conjunto de variantes posible:

```rust
enum Mensaje {
    Salir,
    CambiarColor(i32, i32, i32),
    Mover { x: i32, y: i32 },
    Escribir(String),
}
```

Cada variante puede opcionalmente tener data asociada a ella. La sintaxis para la definicion de variantes es semejante a la sintaxis usada para definir estructuras (`struct`s): puedes tener variantes sin datos (como las estrcuturas tipo-unitario), variantes con data nombrada, y variantes con data sin nombres (como tupla estructuras). Sin embargo, a diferencia de las definiciones de estructuras, un `enum` es un unico tipo. Un valor de un enum puede coincidir con cualquiera de las variantes. Por esta razon, un enum es denominado algunas veces un ‘tipo suma’ (‘sum type’): el conjunto de valores posibles del enum es la suma de los conjuntos de valores posibles para cada variante.

Utilizamos la sintaxis `::` para hacer uso de cada variante: las variantes estan dentro del ambito del `enum`. Lo que hace que lo siguiente sea valido:

```rust
# enum Mensaje {
#     Mover { x: i32, y: i32 },
# }
let x: Mensaje = Mensaje::Mover { x: 3, y: 4 };

enum TurnoJuegoMesa {
    Mover { celdas: i32 },
    Pasar,
}

let y: TurnoJuegoMesa = TurnoJuegoMesa::Mover { celdas: 1 };
```

Ambas variantes poseen el nombre `Mover`, pero debido a que su ambito es dentro del nombre del enum, ambas pueden ser usadas sin conflicto.

Un valor de un tipo enum contiene informacion acerca de cual variante es, en adicion a cualquier data asociada con dicha variante. Esto es algunas veces denominado ‘union etiquetada’ (‘tagged union’), puesto a que la data incluye una ‘etiqueta’ (‘tag’) que indica que tipo es. El compilador usa esta informacion para asegurarse que estemos accediendo la data en el enum de manera segura. Por ejemplo, no puedes simplemente tratar de destructurar un valor como si este fuere una de las posibles variantes:

```rust,ignore
fn procesar_cambio_color(msj: Mensaje) {
    let Mensaje::CambiarColor(r, v, a) = msj; // compile-time error
}
```

No soportar este tipo de operaciones puede lucir un poco limitante, pero es una limitacion que podemos superar. Existen dos maneras, implentando la igualdad por nuestra cuenta, o a traves de el macheo de patrones con expresiones [`match`][match], que aprenderas en la siguiente seccion. Todavia no sabemos lo suficiente de Rust para implementar la igualdad por nuestra cuenta, pero lo veremos en la seccion [`traits`][traits].

[match]: match.html
[if-let]: if-let.html
[traits]: traits.html

# Constructores como funciones

Un constructor de un enum puede ser tambien usado como una funcion. Por ejemplo:

```rust
# enum Mensaje {
# Escribir(String),
# }
let m = Mensaje::Escribir("Hola, mundo".to_string());
```

Es lo mismo que

```rust
# enum Mensaje {
# Escribir(String),
# }
fn foo(x: String) -> Mensaje {
    Mensaje::Escribir(x)
}

let x = foo("Hello, world".to_string());
```

Esto no es inmediatamente util para nosotros, pero cuando lleguemos a los [`closures`][closures], hablaremos acerca de pasar funciones como argumentos a otras funciones. Por ejemplo, con los [iteradores][iterators], podemos hacer esto para convertir un vector de `String`s en un vector de `Message::Escribir`s:

```rust
# enum Mensaje {
# Escribir(String),
# }

let v = vec!["Hola".to_string(), "Mundo".to_string()];

let v1: Vec<Mensaje> = v.into_iter().map(Mensaje::Escribir).collect();
```

[closures]: closures.html
[iterators]: iterators.html
