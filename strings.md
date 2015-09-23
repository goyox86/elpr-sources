% Cadenas de Caracteres

Las cadenas de caracteres son un concepto importante a dominar para cualquier programador. El sistema de manejo de cadenas de caracteres en Rust es un poco distinto al de otros lenguajes, debido a su foco en programación de sistemas. Siempre que poseas una estructura de datos de tamaño variable, las cosas pueden ponerse un poco difíciles, y las cadenas de caracteres son una estructura de datos que puede variar en tamaño. Dicho esto, las cadenas de caracteres de Rust también funcionan de manera diferente que en algunos otros lenguajes de programación de sistemas, como C.

Entremos en los detalles. Una ‘cadena de caracteres’ (‘string’) es una secuencia de valores escalares Unicode codificada como un flujo de bytes UTF-8. Todas las cadenas de caracteres están garantizadas a ser una codificación valida de secuencias UTF-8. Adicionalmente, y a diferencia de otros lenguajes de sistemas, las cadenas de caracteres no son terminadas en null y pueden contener bytes null.

Rust posee dos tipos principales de cadenas de caracteres: `&str` y `String`. Hablemos primero acerca de `&str`. Estos son denominados ‘pedazos de cadenas de caracteres’ (‘string slices’). Los literales String son del tipo &'static str`:

```rust
let saludo = "Hola."; // saludo: &'static str
```

Esta cadena de caracteres es asignada estáticamente, significando que es almacenada dentro de nuestro programa compilado, y existe por la duración completa de su ejecución. El enlace `saludo` es una referencia a una cadena asignada estéticamente. Los pedazos de cadenas de caracteres poseen un tamaño fijo, y no pueden ser mutados.

Por otro lado, un `String`, es una cadena de caracteres asignada desde el montículo. Dicha cadena puede crecer, y también esta garantizada ser UTF-8. Los `String` son creados comúnmente a través de la conversión de un pedazo de cadena de carácter usando el método `to_string`.

```rust
let mut s = "Hola".to_string(); // mut s: String
println!("{}", s);

s.push_str(", mundo.");
println!("{}", s);
```

Los `String`s haran coercion a un `&str` con un `&`:

```rust
fn recibe_pedazo(pedazo: &str) {
    println!("Recibí: {}", pedazo);
}

fn main() {
    let s = "Hola".to_string();
    recibe_pedazo(&s);
}
```

Esta coerción no ocurre para las funciones que aceptan uno de los traits `&str`’s en lugar de `&str`. Por ejemplo, [`TcpStream::connect`][connect] posee un parámetro de tipo `ToSocketAddrs`. Un `&str` esta bien pero un `String` debe ser explícitamente convertido usando `&*`.

```rust,no_run
use std::net::TcpStream;

TcpStream::connect("192.168.0.1:3000"); // parametro &str

let cadena_direccion = "192.168.0.1:3000".to_string();
TcpStream::connect(&*cadena_direccion); // convirtiendo cadena_direccion a &str
```

Ver un `String` como un `&str` es barato, pero convertir el `&str` a un `String` involucra asignación de memoria. No hay razón para hacer eso a menos que sea necesario!

## Indexado

Debido a que las cadenas de caracteres son UTF-8 validos, no soportan indexado:

```rust,ignore
let s = "hola";

println!("La primera letra de s es {}", s[0]); // ERROR!!!
```

Usualmente, el acceso a un vector con `[]` es muy rápido. Pero, puesto a que cada carácter codificado en una cadena UTF-8 puede tener multiples bytes, debes recorrer toda la cadena para encontrar la nᵗʰ letra de una cadena. Esta es una operación significativamente mas cara, y no queremos generar confusiones. Incluso, ‘letra’ no es exactamente algo definido en Unicode. Podemos escoger ver a una cadena de caracteres como bytes individuales, o como codepoints:

```rust
let hachiko = "忠犬ハチ公";

for b in hachiko.as_bytes() {
    print!("{}, ", b);
}

println!("");

for c in hachiko.chars() {
    print!("{}, ", c);
}

println!("");
```

Lo anterior imprime:

```text
229, 191, 160, 231, 138, 172, 227, 131, 143, 227, 131, 129, 229, 133, 172,
忠, 犬, ハ, チ, 公,
```

Como puedes ver, hay mas bytes que caracteres (`char`s).

Puedes obtener algo similar a un indice de esta forma:

```rust
# let hachiko = "忠犬ハチ公";
let perro = hachiko.chars().nth(1); // algo como hachiko[1]
```

Esto enfatiza que tenemos que caminar desde el principio de la lista de `chars`.

## Cortado (Slicing)

Puedes obtener un pedazo de una cadena de caracteres con la sintaxis de cortado:

```rust
let perro = "hachiko";
let hachi = &perro[0..5];
```

Pero nota que estos son desplazamientos de _byte_, no desplazamientos de _character_. Entonces, lo siguiente fallara en tiempo de ejecución:


```rust,should_panic
let perro = "忠犬ハチ公";
let hachi = &perro[0..2];
```

con este error:

```text
thread '<main>' panicked at 'index 0 and/or 2 in `忠犬ハチ公` do not lie on
character boundary'
```

## Concatenación

Si posees un `String`, puedes concatenarle un `&str` al final:

```rust
let hola = "Hola ".to_string();
let mundo = "mundo!";

let hola_mundo = hola + mundo;
```

Pero si tienes dos `String`s, necesitas un `&`:

```rust
let hola = "Hola ".to_string();
let mundo = "mundo!".to_string();

let hola_mundo = hola + &world;
```

Esto es porque `&String` puede hacer coercion automática a un `&str`. Esta característica es denominada ‘[coerciones `Deref`][dc]’.

[dc]: deref-coercions.html
[connect]: ../std/net/struct.TcpStream.html#method.connect
