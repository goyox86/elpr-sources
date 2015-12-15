%  Sintaxis Box y Patrones

Actualmente la única forma estable de crear un `Box` es a través del método `Box::new`. Tampoco es posible destructurar un `Box` en un patron match. La palabra reservada inestable `box` puede ser usada para ambos, crear y destructurar un `Box`. Un ejemplo de su uso seria:

```rust
#![feature(box_syntax, box_patterns)]

fn main() {
    let b = Some(box 5);
    match b {
        Some(box n) if n < 0 => {
            println!("El Box contiene un numero negativo {}", n);
        },
        Some(box n) if n >= 0 => {
            println!("El Box contiene un numero no negativo {}", n);
        },
        None => {
            println!("No box");
        },
        _ => unreachable!()
    }
}
```

Nota que estas facilidades están actualmente escondidas detrás de los feature gates `box_syntax` (creación de boxes) y `box_patterns` (destructuracion y coincidencia de patrones) debido a que la sintaxis podría cambiar en el futuro.

# Retornando apuntadores

En muchos lenguajes con apuntadores, podrías retornar un apuntador desde una función con el objetivo de evitar la copia de una estructura de datos grande. Por ejemplo:

```rust
struct GranStruct {
    uno: i32,
    dos: i32,
    // etc
    cien: i32,
}

fn foo(x: Box<GranStruct>) -> Box<GranStruct> {
    Box::new(*x)
}

fn main() {
    let x = Box::new(GranStruct {
        uno: 1,
        dos: 2,
        cien: 100,
    });

    let y = foo(x);
}
```

La idea es que al pasar un box, solo estas copiando un apuntador, en lugar de los cien `i32` que conforman a `GranStruct`.

Lo anterior es un anti patron en Rust. En su lugar, escribe esto:

```rust
#![feature(box_syntax)]

struct GranStruct {
    uno: i32,
    dos: i32,
    // etc
    cien: i32,
}

fn foo(x: Box<GranStruct>) -> GranStruct {
    *x
}

fn main() {
    let x = Box::new(GranStruct {
        uno: 1,
        dos: 2,
        cien: 100,
    });

    let y: Box<GranStruct> = box foo(x);
}
```

Esta forma te dota de flexibilidad sin sacrificar desempeno.

Podrías pensar que lo anterior resulta en un desempeño terrible: retornar un valor e inmediatamente ponerlo en un box?! No es este patron lo peor de ambos mundos? Rust es mucho mas inteligente que eso. En efecto, no hay copiado en este código. `main` asigna suficiente espacio para el `box`, pasa un apuntador a esa memoria en `foo` como `x`, y luego `foo` escribe el valor de manera directa dentro del `Box<T>`.

Esto es lo suficientemente importante que se merece una repetición: los apuntadores no son para optimizar valores de retorno en tu código. Permite a el llamador decidir como usar tu salida.
