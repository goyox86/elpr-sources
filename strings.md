% Cadenas de Caracteres

Las cadenas de caracteres son un concepto importante a dominar para cualquier programador. El sistema de manejo de cadenas de caracteres en Rust es un poquito distinto al de otros lenguajes, debido a su foco en programcion de sistemas. Siempre que poseas una estructura de datos de tamano variable, las cosas pueden ponerse un poco dificiles, y las cadenas de caracteres son una estructura de datos que puede variar en tamamo. Dicho esto, las cadenas de caracteres de Rust tambien funcionan de manera diferente que en algunos otros lenguajes de programacion de sistemas, como C.

Entremos en los detalles. Una ‘cadena de caracteres’ (‘string’) es una secuencia de valores escalares Unicode codificada como un flujo de bytes UTF-8. Todas las cadenas de caracteres estan garantizadas ser una codificacion valida de secuencias UTF-8. Adicionalmente, y a diferencia de otros lenguajes de sistemas, las cadenas de caracteres no son terminadas en null y pueden contener bytes null.

Rust posee dos tipos principales de cadenas de caracteres: `&str` y `String`. Hablemos acerca de `&str` primero. Estos son denominados ‘pedazos de cadenas de caracteres’ (‘string slices’). Los literales String son del tipo &'static str`:

```rust
let saludo = "Hola."; // saludo: &'static str
```

Esta cadena de caracteres es asignada estaticamente, significando esto que es almacenada dentro de nuestro programa compilado, y existe por la duracion completa de la ejecucion. El enlace `saludo` es una referencia a una cadena asignada estaticamente. Los pedazos de cadenas de caracteres poseen un tamano fijo, y no pueden ser mutados.

Por otro lado, un `String`, es una cadena de caracteres asignada desde el monticulo. Dicha cadena puede crecer, y tambien esta garantizada ser UTF-8. Los `String` son creados comunmente a traves de la conversion de un pedazo de cadena de caracter usando el metodo `to_string`.

```rust
let mut s = "Hola".to_string(); // mut s: String
println!("{}", s);

s.push_str(", mundo.");
println!("{}", s);
```

Los `String`s haran coercion a un `&str` con un `&`:

```rust
fn recibe_pedazo(pedazo: &str) {
    println!("Recibi: {}", pedazo);
}

fn main() {
    let s = "Hola".to_string();
    recibe_pedazo(&s);
}
```

Esta coercion no ocurre para las funciones que aceptan uno de los traits `&str`’s en lugar de `&str`. Por ejemplo, [`TcpStream::connect`][connect] posee un parametro de tipo `ToSocketAddrs`. Un `&str` esta bien pero un `String` debe ser explicitamente convertido usando `&*`.

```rust,no_run
use std::net::TcpStream;

TcpStream::connect("192.168.0.1:3000"); // parametro &str

let cadena_direccion = "192.168.0.1:3000".to_string();
TcpStream::connect(&*addr_string); // convert cadena_direccion a &str
```

Ver un `String` como un `&str` es barato, pero convertir el `&str a un `String` involucra asignacion de memoria. No hay razon para hacer eso a menos que sea necesario!

## Indexado

Debido a que las cadenas de caracteres son UTF-8 validos, estas no soportan indexado:

```rust,ignore
let s = "hola";

println!("La primera letra de s es {}", s[0]); // ERROR!!!
```

Usualmente, elacceso a un vector con `[]` es muy rapido. Pero, puesto a que cada caracter codificado en una cadena UTF-8 puede tener multiples bytes, debes recorrer toda la cadena para encontrar la nᵗʰ letra de una cadena. Esta es una operacion significativamente mas cara, y no queremos gerenrar confusiones. Incluso, ‘letra’ no es exactamente algo definido en Unicode. Podemos escojer ver a una cadena de caracteres como bytes individuales, o como codepoints:

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

Como puedes ver, hay mas bytes que caracteres (`char`s):

Puedes obtener algo similar a un indice asi:

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

Pero nota que estos son intervalos de _byte_, no intervalos de _character_. Entonces, lo siguiente fallara en tiempo de ejecucion:


```rust,should_panic
let perro = "忠犬ハチ公";
let hachi = &perro[0..2];
```

con este error:

```text
thread '<main>' panicked at 'index 0 and/or 2 in `忠犬ハチ公` do not lie on
character boundary'
```

## Concatenacion

Si posees un `String`, puedes concatenarle un `&str` al final:

```rust
let hola = "Hola ".to_string();
let mundo = "mundo!";

let hola_mundo = hola + mundo;
```

Pero si tienes dos `String`s, necesitas un `&`:

```rust
let hola = "Hello ".to_string();
let mundo = "world!".to_string();

let hola_mundo = hello + &world;
```

Esto es porque `&String` puede hacer coercion automatica a un `&str`. Esta caracteristica es denominada ‘[coerciones `Deref`][dc]’.

[dc]: deref-coercions.html
[connect]: ../std/net/struct.TcpStream.html#method.connect
