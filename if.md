% if

El enfoque de Rust con relación a `if` no es particularmente complejo, pero es mas similar a el `if` que encontraras en lenguajes con tipificado dinámico que al de los lenguajes de sistemas tradicionales. Hablemos un poco acerca de este, para estar seguros de que entiendas todos los matices.

`if` es una forma especifica de a un concepto mas general, el ‘branch’ (rama). El nombre proviene de una rama en un árbol: es un punto de decisión, en el cual dependiendo de una opción, multiples caminos pueden ser tomados.

En el caso de `if`, hay una sola elección que conduce a dos caminos:

```rust
let x = 5;

if x == 5 {
    println!("x es cinco!");
}
```

De haber cambiado el valor de `x` a algo diferente, esta linea no hubiese sido impresa. Para ser mas especifico, si la expresión después del `if` es evaluada a `true`, entonces el bloque de código es ejecutado. Si es `false`, dicho bloque no se invoca.

Si deseas que algo ocurra en el caso de `false`, debes hacer uso de un `else`:

```rust
let x = 5;

if x == 5 {
    println!("x es cinco!");
} else {
    println!("x no es cinco :(");
}
```

De haber mas de un caso, usa un `else if`:

```rust
let x = 5;

if x == 5 {
    println!("x es cinco!");
} else if x == 6 {
    println!("x es seis!");
} else {
    println!("x no es ni cinco ni seis :(");
}
```

Todo esto es bien estándar. Sin embargo, también puedes hacer esto:

```rust
let x = 5;

let y = if x == 5 {
    10
} else {
    15
}; // y: i32
```

Lo cual podemos (y probablemente deberiamos) escribir así:

```rust
let x = 5;

let y = if x == 5 { 10 } else { 15 }; // y: i32
```

Esto funciona porque `if` es una expresión. El valor de la expresión es el valor de la ultima expresión en el bloque que haya sido seleccionado. Un `if` sin un `else` siempre resulta en `()` como valor.
