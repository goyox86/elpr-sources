% Conversion Entre Tipos

Rust, con su foco en seguridad, proporciona dos formas diferentes de conversion entre tipos. La primera, `as`, es para conversion segura. En contraste, `transmute` permite conversion arbitraria, is es una de las caractristicas mas peligrosas de Rust!

# `as`

La palabra reservada `as` lleva a cabo conversion basica:

```rust
let x: i32 = 5;

let y = x as i64;
```

`as`, sin embargo, solo permite ciertos tipos de conversion:

```rust,ignore
let a = [0u8, 0u8, 0u8, 0u8];

let b = a as u32; // cuatro ochos hacen 32
```

Esto produce un error:

```text
error: non-scalar cast: `[u8; 4]` as `u32`
let b = a as u32; // cuatro ochos hacen 32
        ^~~~~~~~
```

It’s a ‘non-scalar cast’ because we have multiple values here: the four
elements of the array. These kinds of casts are very dangerous, because they
make assumptions about the way that multiple underlying structures are
implemented. For this, we need something more dangerous.

# `transmute`

La funcion `transmute` es propocionada por una [intrinseca del compilador][intrinsics], y lo que hace es muy simple, pero al mismo tiempo muy peligroso. Le dice a Rust que trate un valor de un tipo como si fuera un valor de otro tipo. Lleva acabo esto sin respetar el sistema de tipos, y confia completamente en ti.

[intrinsics]: intrinsics.html

En nuestro ejemplo anterior, sabemos que nuestro arreglo de cuatro u8`s representa un `u32` de manera correcta, por lo tanto queremos hacer la conversion. Usando `transmute` en lugar de `as` Rust nos permite:

```rust
use std::mem;

unsafe {
    let a = [0u8, 0u8, 0u8, 0u8];

    let b = mem::transmute::<[u8; 4], u32>(a);
}
```

Debemos envolver la operacion en un bloque `unsafe` para que el codigo anteriror compile satisfactoriamente. Tecnicamente, solo la llamada a `mem::transmute` necesita estar dentro del bloque, pero en este caso esta bien tener todo lo relacionado a la conversion de manera que sepas en donde buscar. En este caso, los detalles acerca de `a` son tambien importantes, es por ello que estan dentro del bloque. Veras codigo en cualquiera de los dos estilos, algunas veces el contexto esta muy lejos, de manera que envolver todo el codigo en un bloque `unsafe` no es una gran idea.

Mientras que `transmute` hace muy pocos chequeos, al menos se asegura que los tipos sean del mismo tamano. Esto falla:

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

Dicho esto, estás sólo en esto!
