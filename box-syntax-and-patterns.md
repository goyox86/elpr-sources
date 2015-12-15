%  Sintaxis Box y Patrones

Actualmente la unica forma estable de crear un `Box` es a traves del metodo `Box::new`. Tampoco es posible destructurar un `Box` en un patron match. La palabra reservada inestable `box` puede ser usada para ambos, crear y destructurar un `Box`. Un ejemplo de su uso seria:

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

Nota que estas facilidades estan escondidas actualmente detras de los feture gates `box_syntax` (creacion de boxes) y `box_patterns` (destructuracion y coincidencia de patrones) debido a que la sintaxis podria cambiar en el futuro.

# Retornando apuntadores

En muchos lenguajes con apuntadores, podrias retornar un apuntadro desde una funcion con el objetivo de evitar la copia de una estrcutura de datos grande. Por ejemplo:

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

Esta forma te dota de felxibilidad sin sacrificar desempeno.

Podrias pensar que lo anterior resulta en un desempeno terrible: retornar un valor e inmediatamente ponerlo en un box?! No es este patron lo peor de ambos mundos? Rust es mucho mas inteligente que eso. No hay copiado en este codigo. `main` asigna suficiente espacio para el `box`, pasa un apuntador a esa memoria en `foo` como `x`, y luego `foo` escribe el valor de manera directa dentro del `Box<T>`.

Esto es lo suficientemente importante que se merece una repeticion: los apuntadores no son para optimizar valores de retorno en tu codigo. Permite a el llamador decidir como este desea usar tu salida.
