% Genéricos

Algunas veces, cuando se escribe una funcion o una estructura de datos, podriamos desear que este pudiese funcionar con multiples tipos de argumentos. En Rust podemos lograr esto a traves de los genericos. Los genericos son llamados ‘polimorfismo parametrico’ en la en teoria de tipos, lo cual significa que son tipos o funciones que poseen multiples formas  (‘poly’ es multiple, ‘morph’ es forma) sobre un determinado parametro (‘parametrico’).

De cualquier modo, suficiente acerca de teoria de tipos, veamos un poco de codigo generico. La biblioteca estandar de Rust provee un tipo, `Option<T>`, el cual es generico:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

La parte `<T>`, parte que has visto unas cuantas veces anteriormente, indica que este es un tipo de datos generico. Dentro de la declaracion de nuestro enum, donde sea que veamos una `T` susutituimos ese typo por el mismo tipo usado en el generico. He aqui un ejemplo del uso de `Option<T>`, con algunos anoteciones de tipo extra:

```rust
let x: Option<i32> = Some(5);
```

En la declaracion del tipo, decimos `Option<i32>`. Nota cuan similar luce esto a `Option<T>`. Entonces en este `Option`, `T` posee el valor de `i32`. En el lado derecho del binding, hacemos un `Some(T)`, en donde `T` es `5`. Debido a que `5` es un `i32`, ambos lados coinciden, y Rust es feliz. De no haber coincidido, hubiesemos obtenido un error:

```rust,ignore
let x: Option<f64> = Some(5);
// error: mismatched types: expected `core::option::Option<f64>`,
// found `core::option::Option<_>` (expected f64 but found integral variable)
```

Eso no significa que no podamos crear `Option<T>`s que contengan un `f64` simplemente deben coincidir:

```rust
let x: Option<i32> = Some(5);
let y: Option<f64> = Some(5.0f64);
```

Esto esta muy bien. Una definicion, multiples usos.

Los genericos no deben necesariamente ser genericos sobre un solo tipo. Considera otro tipo de la biblioteca estandar de Rust que es similar, `Result<T, E>`:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

Este tipo es generiaco sobre _dos_ tipos: `T` y `E`. Por cierto, las letras mayusculas pueden ser cualquiera. Pudimos haber definido `Result<T, E>` como:

```rust
enum Result<A, Z> {
    Ok(A),
    Err(Z),
}
```

de haber querido. La convencion dice que el primer parametro generico debe ser `T`, de ‘tipo’, y que usemos `E` para ‘error’. A Rust, sin embargo no le importa.

El tipo `Result<T, E>` se usa para retornar el resultado de una computacion, con la posibilidad de retornar un error en caso de que dicha computacion no haya sido exitosa.

## Funciones genericas

Podemos escribir funciones que tomen tipos genericos con una sintaxis similar:

```rust
fn recibe_cualquier_cosa<T>(x: T) {
    // hacer algo con x
}
```

La sintaxis posee dos partes: el `<T>` dice “esta funcion es generica sobre un tipo, `T`”, y la parte `x: T` dice  “x posee el tipo `T`.”

Multiples argumentos pueden tener el mismo tipo:

Multiple arguments can have the same generic type:

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

Tambien puedes almacenar un tipo generico en una estructura:

```rust
struct Punto<T> {
    x: T,
    y: T,
}

let origen_entero = Punto { x: 0, y: 0 };
let origen_flotante = Punto { x: 0.0, y: 0.0 };
```

Similarmente a las funciones, la seccion `<T>` es en donde declaramos los parametros genericos, despues de ello hacemos uso de `x: T` en la declaracion del tipo, tambien.

Cuando deseamos agregar una implementacion para la estructura generica, simplemente declaras el parametro de tipo despues de `impl`:

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

Hasta ahora solo has visto genericos que aceptan absolutamente cualquier tipo. Estas son utiles en muchos casos, ya has visto `Option<T>`, y mas tarde conceras contenedores universales como [`Vec<T>`][Vec]. Por otro lado, a veces querras intercambiar esa flexibilidad por poder expresivo. Lee acerca de [limites trait][traits] para ver porque y como.

[traits]: traits.html
[Vec]: ../std/vec/struct.Vec.html
