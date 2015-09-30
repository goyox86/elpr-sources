% Patrones

Los patrones son bastante comunes en Rust. Los usamos en [enlaces a variable][bindings], [sentencias match][match], y otros casos. Embarquemonos en un tour torbellino por todas las cosas que los patrones son capaces de hacer!

[bindings]: variable-bindings.html
[match]: match.html

Un repaso rápido: puedes probar patrones contra literales directamente, y `_` actúa como un caso `cualquiera`:

```rust
let x = 1;

match x {
    1 => println!("uno"),
    2 => println!("dos"),
    3 => println!("tres"),
    _ => println!("cualquiera"),
}
```

Imprime `uno`.

# Multiples patrones

Puedes probar multiples patrones con `|`:

```rust
let x = 1;

match x {
    1 | 2 => println!("uno o dos"),
    3 => println!("tres"),
    _ => println!("cualquiera"),
}
```

Lo anterior imprime `uno o dos`.

# Destructuracion

Si posees un tipo de datos compuesto, como un [`struct`][struct], puedes destructurarlo dentro de un patron:

```rust
struct Punto {
    x: i32,
    y: i32,
}

let origen = Punto { x: 0, y: 0 };

match origen {
    Punto { x, y } => println!("({},{})", x, y),
}
```

[struct]: structs.html

Puedes usar `:` para darle un nombre diferente a un valor.

```rust
struct Punto {
    x: i32,
    y: i32,
}

let origen = Punto { x: 0, y: 0 };

match origen {
    Punto { x: x1, y: y1 } => println!("({},{})", x1, y1),
}
```

Si solo nos importan algunos valores, no tenemos que darle nombres a todos:

```rust
struct Punto {
    x: i32,
    y: i32,
}

let origen = Punto { x: 0, y: 0 };

match origen {
    Punto { x, .. } => println!("x es {}", x),
}
```

Esto imprime `x es 0`.

Puedes hacer este tipo de pruebas en cualquier miembro, no solo el primero:

```rust
struct Punto {
    x: i32,
    y: i32,
}

let origen = Punto { x: 0, y: 0 };

match origen {
    Punto { y, .. } => println!("y es {}", y),
}
```

Lo anterior imprime `y es 0`.

Este comportamiento de ‘destructuracion’ funciona en cualquier tipo de datos compuesto, como [tuplas][tuples] o [enums][enums].

[tuples]: primitive-types.html#tuples
[enums]: enums.html

# Ignorando enlaces a variables

Puedes usar `_` en un patron para ignorar tanto el tipo como el valor.

Por ejemplo, he aquí un `match` contra un `Result<T, E>`:


```rust
# let algun_valor: Result<i32, &'static str> = Err("Hubo un error");
match algun_valor {
    Ok(valor) => println!("valor obtenido: {}", valor),
    Err(_) => println!("ha ocurrido un error"),
}
```

En el primer brazo, enlazamos el valor dentro de la variante `Ok` a la variable `valor`. Pero en el brazo `Err` usamos `_` para ignorar el error especifico, y solo imprimir un mensaje de error general.

`_` es valido en cualquier patron que cree un enlace a variable. También puede ser util para ignorar porciones de una estructura mas grande:

```rust
fn coordenada() -> (i32, i32, i32) {
    // generar y retornar algún tipo de tupla de tres elementos
# (1, 2, 3)
}

let (x, _, z) = coordenada();
```

Aquí, asociamos ambos el primer y ultimo elemento de la tupla a `x` y `z` respectivamente, ignorando el elemento de la mitad.

Similarmente, puedes usar `..` en un patrón para ignorar multiples valores.


```rust
enum TuplaOpcional {
    Valor(i32, i32, i32),
    Faltante,
}

let x = TuplaOpcional::Valor(5, -2, 3);

match x {
    TuplaOpcional::Valor(..) => println!("Tupla obtenida!"),
    TuplaOpcional::Faltante => println!("Sin suerte."),
}
```

Esto imprime `Tupla obtenida!`.

# ref y ref mut

Si deseas obtener una [referencia][ref], debes usar la palabra reservada `ref`:

```rust
let x = 5;

match x {
    ref r => println!("Referencia a {} obtenida", r),
}
```

Imprime `Referencia a 5 obtenida`.

[ref]: references-and-borrowing.html

Acá, la `r` dentro del `match` posee el tipo `&i32`. En otras palabras la palabra reservada `ref` _crea_ una referencia, para ser usada dentro del patrón. Si necesitas una referencia mutable `ref mut` funcionara de la misma manera:

```rust
let mut x = 5;

match x {
    ref mut rm => println!("Referencia mutable a {} obtenida", rm),
}
```

# Rangos

Puedes probar un rango de valors con `...`:


```rust
let x = 1;

match x {
    1 ... 5 => println!("uno al cinco"),
    _ => println!("cualquier cosa"),
}
```

Esto imprime `uno al cinco`.

Los rangos son usados mayormente con enteros y `chars`s:

```rust
let x = '💅';

match x {
    'a' ... 'j' => println!("letra temprana"),
    'k' ... 'z' => println!("letra tardia"),
    _ => println!("algo mas"),
}
```

This prints `algo mas`.

# Enlaces a variable

Puedes asociar valores a nombres con `@`:

```rust
let x = 1;

match x {
    e @ 1 ... 5 => println!("valor de rango {} obtenido", e),
    _ => println!("lo que sea"),
}
```

This prints `valor de rango 1 obtenido`. Lo anterior es util cuando desees hacer un match complicado a una parte de una estructura de datos:

```rust
#[derive(Debug)]
struct Persona {
    nombre: Option<String>,
}

let nombre = "Steve".to_string();
let mut x: Option<Persona> = Some(Persona { nombre: Some(nombre) });
match x {
    Some(Persona { nombre: ref a @ Some(_), .. }) => println!("{:?}", a),
    _ => {}
}
```

Dicho código imprime `Some("Steve")`: hemos asociado el `nombre` interno a `a`.

Si usas `@` con `|`, necesitas asegurarte de que el nombre sea asociado en cada parte del patron:

```rust
let x = 5;

match x {
    e @ 1 ... 5 | e @ 8 ... 10 => println!("valor de rango {} obtenido", e),
    _ => println!("lo que sea"),
}
```

# Guardias

Puedes introducir `guardias match` (‘match guards’) con `if`:

```rust
enum EnteroOpcional {
    Valor(i32),
    Faltante,
}

let x = EnteroOpcional::Value(5);

match x {
    EnteroOpcional::Valor(i) if i > 5 => println!("Entero mayor a cinco obtenido!"),
    EnteroOpcional::Valor(..) => println!("Entero obtenido!"),
    EnteroOpcional::Faltante => println!("Sin suerte."),
}
```

Esto imprime `Entero obtenido!"`.

Si estas usando `if` con multiples patrones, el `if` aplica a ambos lados:

```rust
let x = 4;
let y = false;

match x {
    4 | 5 if y => println!("si"),
    _ => println!("no"),
}
```

Lo anterior imprime `no`, debido a que el `if` aplica a el `4 | 5` completo, y no solo al `5`. En otras palabras, la precedencia del `if` se comporta de la siguiente manera:

```text
(4 | 5) if y => ...
```

y no así:

```text
4 | (5 if y) => ...
```

# Mezcla y Match

Uff! Eso fue un montón de formas diferentes para probar cosas, y todas pueden ser mezcladas y probadas, dependiendo de los que estés haciendo:

```rust,ignore
match x {
    Foo { x: Some(ref nombre), y: None } => ...
}
```

Los patrones son muy poderosos. Haz buen uso de ellos.
