% Vectores

Un ‘vector’ es un arreglo dinámico, implementado como el tipo de la biblioteca estándar [`Vec<T>`][vec]. La `T` significa que podemos tener vectores de cualquier tipo (echa un vistazo al capitulo de [genéricos][generics] para mas información). Los vectores siempre alojan sus datos en el montículo. Puedes crear vectores con la macro `vec!`:

```rust
let v = vec![1, 2, 3, 4, 5]; // v: Vec<i32>
```

(Nota que a diferencia de la macro `println!` que hemos usado en el pasado, usamos corchetes `[]` con la macro `vec!`. Rust te permite usar cualquiera en cualquier situación, en esta oportunidad es por pura convención)

Existe una forma alternativa de `vec!` para repetir un valor inicial:

```rust
let v = vec![0; 10]; // diez ceros
```

## Accediendo a elementos

Para obtener el valor en un indice en particular del vector, usamos `[]`s:

```rust
let v = vec![1, 2, 3, 4, 5];

println!("El tercer elemento de v es {}", v[2]);
```

Los indices comienzan desde `0`, entonces, el tercer elemento es `v[2]`.

## Iterando

Una vez poseas un vector, puedes iterar a través de sus elementos con `for`. Hay tres versiones:

```rust
let mut v = vec![1, 2, 3, 4, 5];

for i in &v {
    println!("Una referencia a {}", i);
}

for i in &mut v {
    println!("Una referencia mutable a {}", i);
}

for i in v {
    println!("Tomando pertenencia del vector y su elemento {}", i);
}
```

Los vectores poseen muchos otros métodos útiles, acerca de los cuales puedes leer en su [documentation][vec]

[vec]: ../std/vec/index.html
[generic]: generics.html
