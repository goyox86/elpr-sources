% Alias `type`

La palabra reservada `type` te permite declarar un alias a otro tipo:

```rust
type Nombre = String;
```

Ahora, puedes usar este tipo como si fuera un tipo real:

```rust
type Name = String;

let x: Nombre = "Hola".to_string();
```

Sin embargo, nota, que es un _alias_, no un nuevo tipo. En otras palabras, debido a que Rust es fuertemente tipificado, podrías esperar que una comparación entre dos tipos diferentes falle:

```rust,ignore
let x: i32 = 5;
let y: i64 = 5;

if x == y {
   // ...
}
```

lo anterior, produce:

```text
error: mismatched types:
 expected `i32`,
    found `i64`
(expected i32,
    found i64) [E0308]
     if x == y {
             ^
```

Pero, si tuviéramos un alias:

```rust
type Num = i32;

let x: i32 = 5;
let y: Num = 5;

if x == y {
   // ...
}
```

Compilaría sin errores. Valores de un tipo `Num` son lo mismo que un valor de tipo `i32`, en todos los aspectos.

Puedes también hacer uso de alias de tipos en genéricos:

```rust
use std::result;

enum ErrorConcreto {
    Foo,
    Bar,
}

type Result<T> = result::Result<T, ErrorConcreto>;
```

El código anterior, crea una version especializada de el tipo `Result`, la cual posee siempre un `ErrorConcreto` para la parte `E` de `Result<T, E>`. Esta practica es usada comúnmente en la biblioteca estándar para la creación de errores personalizados para cada sub-seccion. Por ejemplo, [io::Result][ioresult].

[ioresult]: ../std/io/type.Result.html
