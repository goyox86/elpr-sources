% if

El enfoque de Rust con relacion a `if` no es particularmente complejo, pero es mas similar a el `if` que encontraras en lenguajes con tipificado dinamico que al de los lenguajes de sistemas tradicionales. Hablemos un poco acerca de este, para estar seguros de que entiendas todos los matices.

`if` es una forma especfifiva de a un concepto mas general, el ‘branch’ (rama). El nombre proviene de ua rama en un arbol: es un punto de desicion, en el cual dependiendo de una opcion, multiples camin9os pueden ser tomados.

En el cvaso de `if`, hay una sola eleccion que conduce a dos caminos:

```rust
let x = 5;

if x == 5 {
    println!("x es cinco!");
}
```

De haber cambiado el valor de `x` a algo diferente, esta linea no hubiese sido impresa. Para ser mas especificios, si la expresion despues del `if` es evaluada a `true`, entonces el bloque de codigo es ejecutado. Si es `false`, dicho bloque no es invocado.

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

Todo esto es bien estandar. Sin embargo, tambien puedes hacer esto:

```rust
let x = 5;

let y = if x == 5 {
    10
} else {
    15
}; // y: i32
```

Lo cual podemos (y probablemente debamos) escribir asi:

```rust
let x = 5;

let y = if x == 5 { 10 } else { 15 }; // y: i32
```

Esto funciona porque `if` es una expresion. El valor de la expresion es el valor de la ultima expresion en el bloque que haya sido seleccionado. Un `if` sin un `else` siempre rresulta en `()` como valor.
