% Conversión Entre Tipos

Rust, con su foco en seguridad, proporciona dos formas diferentes de conversión entre tipos. La primera, `as`, es para conversión segura. En contraste, `transmute` permite conversion arbitraria, y es una de las características mas peligrosas de Rust!

# `as`

La palabra reservada `as` lleva a cabo conversion básica:

```rust
let x: i32 = 5;

let y = x as i64;
```

`as`, sin embargo, solo permite ciertos tipos de conversión:

```rust,ignore
let a = [0u8, 0u8, 0u8, 0u8];

let b = a as u32; // cuatro ochos hacen 32
```

Lo anterior produce un error:

```text
error: non-scalar cast: `[u8; 4]` as `u32`
let b = a as u32; // cuatro ochos hacen 32
        ^~~~~~~~
```

Es una ‘conversion no escalar’ (‘non-scalar cast’) porque tenemos multiples valores: los cuatro elementos en el arreglo. Este tipo de conversiones son muy peligrosas, debido a que asumen cosas acerca de como las multiples estructuras subyacentes están implementadas. Para esto, necesitamos algo mas peligroso.

# `transmute`

La función `transmute` es proporcionada por una [intrínseca del compilador][intrinsics], y lo que hace es muy simple, pero al mismo tiempo muy peligroso. Le dice a Rust que trate un valor de un tipo como si fuera un valor de otro tipo. Lleva acabo esto sin respetar el sistema de tipos, y confía completamente en ti.

[intrinsics]: intrinsics.html

En nuestro ejemplo anterior, sabemos que nuestro arreglo de cuatro `u8`s representa un `u32` de manera correcta, por lo tanto queremos hacer la conversión. Usando `transmute` en lugar de `as` Rust nos permite:

```rust
use std::mem;

unsafe {
    let a = [0u8, 0u8, 0u8, 0u8];

    let b = mem::transmute::<[u8; 4], u32>(a);
}
```

Debemos envolver la operación en un bloque `unsafe` para que el código anterior compile satisfactoriamente. Técnicamente, solo la llamada a `mem::transmute` necesita estar dentro del bloque, pero en este caso esta bien tener todo lo relacionado a la conversion de manera que sepas en donde buscar. En este caso, los detalles acerca de `a` son también importantes, es por ello que están dentro del bloque. Veras código en cualquiera de los dos estilos, algunas veces el contexto esta tan lejos que envolver todo el código en un bloque `unsafe` no es una gran idea.

Mientras que `transmute` hace muy pocos chequeos, se asegura, al menos,  que los tipos sean del mismo tamaño. Lo siguiente, falla:

```rust,ignore
use std::mem;

unsafe {
    let a = [0u8, 0u8, 0u8, 0u8];

    let b = mem::transmute::<[u8; 4], u64>(a);
}
```

con:

```text
error: transmute called on types with different sizes: [u8; 4] (32 bits) to u64
(64 bits)
```

Dicho esto, estás por tu cuenta!
