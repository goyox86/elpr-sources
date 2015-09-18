% Estructuras (Structs)

Las estructuras (`struct`s) son una forma de crear tipos de datos mas complejos. Por ejemplo, si estuviéramos haciendo cálculos que involucraran coordenadas en un espacio 2D, necesitaríamos ambos, un valor `x` y un valor `y`:

```rust
let origen_x = 0;
let origen_y = 0;
```

una `struct` nos permite combinar ambos en un único tipo de datos unificado:

```rust
struct Punto {
    x: i32,
    y: i32,
}

fn main() {
    let origen = Punto { x: 0, y: 0 }; // origin: Punto

    println!("El origen esta en ({}, {})", origen.x, origen.y);
}
```

Hay muchas cosas pasando acá, así que analicémoslo por partes. Declaramos una estructura con la palabra reservada `struct`, seguida de un nombre. Por convención, los `struct`s comienzan con una letra mayúscula y son camel cased: `PuntoEnElEspacio`, no `Punto_En_El_Espacio`.

Podemos crear una instancia de nuestra `struct` via `let`, como es usual, pero usamos una sintaxis estilo `clave: valor` para asignar cada campo. El order no necesita ser el mismo que en la declaración original.

Finalmente, debido a que tenemos nombres de campos, podemos acceder a ellos a través de la notación de puntos: `origen.x`.

Los valores en `struct`s son inmutables por defecto, justo como los demás enlaces a variables en Rust. Usa `mut` para hacerlos mutables:

```rust
struct Punto {
    x: i32,
    y: i32,
}

fn main() {
    let mut punto = Punto { x: 0, y: 0 };

    punto.x = 5;

    println!("El origen esta en ({}, {})", punto.x, punto.y);
}
```

Esto imprimirá `El origen esta en (5, 0)`.

Rust no soporta mutabilidad de campos a nivel de lenguaje, es por ello que no puedes escribir algo como esto:

```rust,ignore
struct Punto {
    mut x: i32,
    y: i32,
}
```

La mutabilidad es una propiedad del enlace a variable, no de la estructura en si misma. Si estas acostumbrado a la mutabilidad a nivel de campos, esto puede parecer un poco extraño al principio, pero simplifica las cosas de manera significativa. Incluso te permite hacer a las cosas mutables solo por un periodo corto de tiempo:

```rust,ignore
struct Punto {
    x: i32,
    y: i32,
}

fn main() {
    let mut punto = Punto { x: 0, y: 0 };

    punto.x = 5;

    let punto = punto; // ahora, este nuevo enlace a variable no puede ser cambiado

    punto.y = 6; // esto causa un error
}
```

# Sintaxis de actualización

Una `struct` puede incluir `..` para indicar que deseas usar una copia de algún otro `struct` para algunos de los valores. Por ejemplo:

```rust
struct Punto3d {
    x: i32,
    y: i32,
    z: i32,
}

let mut punto = Punto3d { x: 0, y: 0, z: 0 };
punto = Punto3d { y: 1, .. punto };
```

Lo anterior, asigna a `punto` un nuevo `y`, pero mantiene los antiguos valores de `x` y `z`. Tampoco tiene que ser la misma estructura, puedes hacer uso de esta sintaxis cuando creas nuevas, y esta copiara los valores que no especifiques:

```rust
# struct Punto3d {
#     x: i32,
#     y: i32,
#     z: i32,
# }
let origen = Punto3d { x: 0, y: 0, z: 0 };
let punto = Punto3d { z: 1, x: 2, .. origen };
```

# Tupla estructuras (Tuple structs)

Rust posee otro tipo de datos que es como un híbrido entre una [tupla][tuple] y una `struct`, llamado `tulla estructura` (`tuple struct`). Las tupla estructuras poseen un nombre, pero sus campos no:

```rust
struct Color(i32, i32, i32);
struct Punto(i32, i32, i32);
```

[tuple]: primitive-types.html#tuples

Las dos anteriores no serán iguales, incluso si poseen los mismos valores:

```rust
# struct Color(i32, i32, i32);
# struct Punto(i32, i32, i32);
let negro = Color(0, 0, 0);
let origen = Punto(0, 0, 0);
```

Casi siempre es mejor usar una `struct` que una `tuple struct`. En su lugar, podríamos escribir `Color` and `Punto` así:

```rust
struct Color {
    rojo: i32,
    azul: i32,
    verde: i32,
}

struct Punto {
    x: i32,
    y: i32,
    z: i32,
}
```

Ahora, tenemos nombres, en lugar de posiciones. Los buenos nombres son importantes, y con una `struct`, tenemos nombres.

_Hay_ un caso en el cual una tupla estructura es muy útil, y es una tupla estructura con un solo elemento. Se denomina patrón `nuevotipo` (`newtype`), puesto a que crear un nuevo tipo, distinto a el valor que contiene, expresando su propia semántica:

```rust
struct Pulgadas(i32);

let longitud = Pulgadas(10);

let Pulgadas(longitud_enteros) = longitud;
println!("la longitud es {} pulgadas", longitud_enteros);
```

Como notaras, puedes extraer el entero contenido con un `let` de destructuración, justo como en las tuplas regulares. En este caso, el `let Pulgadas(longitud_enteros)` asigna `10` a `longitud_enteros`.

# Estructuras tipo-unitario (unit-like structs)

Puedes definir una `struct` sin ningún miembro:

```rust
struct Electron;
```

Dicha `struct` es denominada `tipo-unitario` (`unit-like`) por su semejanza a la tupla vacía, `()`, algunas veces llamada `unidad` (`unit`). Como una tupla estructura, define un nuevo tipo.

Lo anterior es raramente util en si mismo (aunque algunas veces puede servir como un tipo marcador), pero en combinación con otras características, puede tornarse útil. Por ejemplo, una librería puede requerir crear una estructura que implementa cierto [trait][trait] para manejar eventos. De no poseer ninguna data para guardar en la estructura, simplemente puedes crear una `struct` tipo-unitario.

[trait]: traits.html
