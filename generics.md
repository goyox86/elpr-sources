% Genéricos

Algunas veces, cuando se escribe una función o una estructura de datos, podríamos desear que esta pudiese funcionar con multiples tipos de argumentos. En Rust podemos lograr esto a través de los genéricos. Los genéricos son llamados ‘polimorfismo parametrico’ en la teoría de tipos, lo que significa que son tipos o funciones que poseen multiples formas  (‘poly’ de multiple, ‘morph’ de forma) sobre un determinado parámetro (‘parametrico’).

De cualquier modo, suficiente acerca de teoría de tipos, veamos un poco de código genérico. La biblioteca estándar de Rust provee un tipo, `Option<T>`, el cual es genérico:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

La parte `<T>`, la cual has visto unas cuantas veces anteriormente, indica que este es un tipo de datos genérico. Dentro de la declaración de nuestro enum, donde sea que veamos una `T` sustituimos ese tipo por el mismo tipo usado en el genérico. He aquí un ejemplo del uso de `Option<T>`, con algunos anotaciones de tipo extra:

```rust
let x: Option<i32> = Some(5);
```

En la declaración del tipo, decimos `Option<i32>`. Nota cuan similar esto luce esto a `Option<T>`. Entonces, en este `Option`, `T` posee el valor de `i32`. En el lado derecho del binding, hacemos un `Some(T)`, en donde `T` es `5`. Debido a que `5` es un `i32`, ambos lados coinciden, y Rust es feliz. De no haber coincidido, hubiésemos obtenido un error:

```rust,ignore
let x: Option<f64> = Some(5);
// error: mismatched types: expected `core::option::Option<f64>`,
// found `core::option::Option<_>` (expected f64 but found integral variable)
```

Eso no significa que no podamos crear `Option<T>`s que contengan un `f64`. Simplemente deben coincidir:

```rust
let x: Option<i32> = Some(5);
let y: Option<f64> = Some(5.0f64);
```

Muy bueno. Una definición, multiples usos.

Los genéricos no deben necesariamente ser genéricos sobre un solo tipo. Considera otro tipo similar en la biblioteca estándar de Rust, `Result<T, E>`:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

Este tipo es genérico sobre _dos_ tipos: `T` y `E`. Por cierto, las letras mayusculas pueden ser cualquiera. Pudimos haber definido `Result<T, E>` como:

```rust
enum Result<A, Z> {
    Ok(A),
    Err(Z),
}
```

de haber querido. La convención dice que el primer parametro generico debe ser `T`, de ‘tipo’, y que usemos `E` para ‘error’. A Rust, sin embargo no le importa.

El tipo `Result<T, E>` se usa para retornar el resultado de una computación, con la posibilidad de retornar un error en caso de que dicha computación no haya sido exitosa.

## Funciones genéricas

Podemos escribir funciones que tomen tipos genéricos con una sintaxis similar:

```rust
fn recibe_cualquier_cosa<T>(x: T) {
    // hacer algo con x
}
```

La sintaxis posee dos partes: el `<T>` dice “esta función es generica sobre un tipo, `T`”, y la parte `x: T` dice “x posee el tipo `T`.”

Multiples argumentos pueden tener el mismo tipo:


```rust
fn recibe_dos_cosas_del_mismo_tipo<T>(x: T, y: T) {
    // ...
}
```

Pudimos haber escrito una version que recibe multiples tipos:


```rust
fn recibe_dos_cosas_de_distintos_tipos<T, U>(x: T, y: U) {
    // ...
}
```

## Structs genericos

También puedes almacenar un tipo genérico en una estructura:

```rust
struct Punto<T> {
    x: T,
    y: T,
}

let origen_entero = Punto { x: 0, y: 0 };
let origen_flotante = Punto { x: 0.0, y: 0.0 };
```

Similarmente a las funciones, la sección `<T>` es en donde declaramos los parámetros genéricos, después de ello hacemos uso de `x: T` en la declaración del tipo, también.

Cuando deseamos agregar una implementación para la estructura genérica, simplemente declaras el parámetro de tipo después de `impl`:

```rust
# struct Punto<T> {
#     x: T,
#     y: T,
# }
#
impl<T> Punto<T> {
    fn intercambiar(&mut self) {
        std::mem::swap(&mut self.x, &mut self.y);
    }
}
```

Hasta ahora solo has visto genéricos que aceptan absolutamente cualquier tipo. Estas son útiles en muchos casos, ya has visto `Option<T>`, y mas tarde conoceras contenedores universales como [`Vec<T>`][Vec]. Por otro lado, a veces querrás intercambiar esa flexibilidad por mayor poder expresivo. Lee acerca de [limites trait][traits] para ver porque y como.

[traits]: traits.html
[Vec]: ../std/vec/struct.Vec.html
