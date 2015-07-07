% Manejo de Errores

> Los planes mejor establecidos por ratones y hombres a menudo se tuercen.
> "Tae a Moose", Robert Burns

Al unas veces, las cosas simplemente salen mal. Es importante tener un plan para cuando lo inevitable suceda. Rust posee un soporte rico para el manejo de errores que podrían (seamos honestos: ocurrirán) ocurrir en tus programas.

Existen dos tipos de errores que pueden ocurrir en tus programas: fallas, y pánicos. Hablaremos de las diferencias entre estos dos, y luego discutiremos como como manejar cada uno. Después discutiremos como transformar fallas en pánicos.

# Falla vs. Pánico

Rust usa dos términos para diferenciar entre las dos formas de error: falla, y pánico. Una *falla* es un error del cual nos podemos recuperar de alguna forma. Un *pánico* es un error irrecuperable.

Que queremos decir con "recuperar"? Bueno, en la mayoría de los casos, la posibilidad de un error es esperada. Por ejemplo, considera la función `parse`:


```ignore
"5".parse();
```

Este método convierte una cadena de caracteres en otro tipo. Pero debido a que es una cadena de caracteres, no se puede estar seguro de que la conversion efectivamente tenga éxito. Por ejemplo, a que debería ser convertido esto?:

```ignore
"hola5mundo".parse();
```

Esto no funcionara. Sabemos que esta función solo tendrá éxito para algunas entradas. Es un comportamiento esperado. Llamamos a este error una *falla*.

Por otro lado, algunas veces, hay errores que son inesperados, o de los cuales no nos podemos recuperar. Un ejemplo clásico es un `assert!`:

```rust
# let x = 5;
assert!(x == 5);
```

Usamos `assert!` para declarar que algo es cierto (true). Si la declaración no es verdad, entonces algo esta muy mal. Suficientemente mal como para no poder continuar la ejecución en el estado actual. Otro ejemplo es el uso de la macro `unreachable!()`:


```rust,ignore
enum Evento {
    NuevoLanzamiento,
}

fn probabilidad(_: &Evento) -> f64 {
    // la implementacióncion real seria mas compleja, por supuesto
    0.95
}

fn probabilidad_descriptiva(evento: Evento) -> &'static str {
    match probabilidad(&evento) {
        1.00 => "cierto",
        0.00 => "imposible",
        0.00 ... 0.25 => "muy poco probable",
        0.25 ... 0.50 => "poco probable",
        0.50 ... 0.75 => "probable",
        0.75 ... 1.00 => "muy probable",
    }
}

fn main() {
    std::io::println(probabilidad_descriptiva(NuevoLanzamiento));
}
```

Lo anterior resultara en un error:

```text
error: non-exhaustive patterns: `_` not covered [E0004]
```

Si bien sabemos que hemos cubierto todos los casos posibles, Rust no puede saberlo. No sabe cual es la probabilidad entre 0.0 y 1.0. Es por ello que agregamos otro caso:

```rust
use Evento::NuevoLanzamiento;

enum Evento {
    NuevoLanzamiento,
}

fn probability(_: &Event) -> f64 {
    // la implementación real seria mas compleja, por supuesto
    0.95
}

fn probabilidad_descriptiva(evento: Evento) -> &'static str {
    match probabilidad_descriptiva(&evento) {
        1.00 => "cierto",
        0.00 => "imposible",
        0.00 ... 0.25 => "muy poco probable",
        0.25 ... 0.50 => "poco probable",
        0.50 ... 0.75 => "probable",
        0.75 ... 1.00 => "muy probable",
        _ => unreachable!()
    }
}

fn main() {
    println!("{}", probabilidad_descriptiva(NuevoLanzamiento));
}
```

Nunca deberíamos alcanzar nunca el caso `_`, debido a esto hacemos uso de la macro para indicarlo. `unreachable!()` produce un tipo diferente de error que `Result`. Rust llama a ese tipo de errores *pánicos*.


# Manejando errores con `Option` y `Result`

La manera mas simple de indicar que una función puede fallar es usando el tipo `Option<T>`. Por ejemplo, el método find en las cadenas de caracteres intenta localizar un patron en la cadena, retorna un `Option`:


```rust
let s = "foo";

assert_eq!(s.find('f'), Some(0));
assert_eq!(s.find('z'), None);
```

Esto es apropiado para casos simples, pero no nos da mucha información en el caso de una falla. Que tal si quisiéramos saber el *porque* la función fallo? Para ello, podemos usar el tipo `Result<T, E>`. Que luce así:


```rust
enum Result<T, E> {
   Ok(T),
   Err(E)
}
```

Esta enum es proporcionada por Rust, es por ello que no necesitas definirla si deseas hacer uso de ella en tu código. La variante `Ok(T)` representa éxito, y la variante `Err(E)` representa una falla. Retornar un `Result` en lugar de un `Option` es recomendable para la mayoría de los casos no triviales:

He aquí un ejemplo del uso de `Result`:


```rust
#[derive(Debug)]
enum Version { Version1, Version2 }

#[derive(Debug)]
enum ErrorParseo { LongitudCabeceraInvalida, VersionInvalida }

fn parsear_version(cabecera: &[u8]) -> Result<Version, ErrorParseo> {
    if cabecera.len() < 1 {
        return Err(ErrorParseo::LongitudCabeceraInvalida);
    }
    match cabecera[0] {
        1 => Ok(Version::Version1),
        2 => Ok(Version::Version2),
        _ => Err(ErrorParseo::LongitudCabeceraInvalida)
    }
}

let version = parsear_version(&[1, 2, 3, 4]);
match version {
    Ok(v) => {
        println!("trabajando con la version: {:?}", v);
    }
    Err(e) => {
        println!("error parseando cebecera: {:?}", e);
    }
}
```

Esta función hace uso de un enum, `ErrorParseo`, para enumerar los errores que pueden ocurrir.

El trait [`Debug`](../std/fmt/trait.Debug.html) es el que nos permite imprimir el valor del enum usando la operación de formato `{:?}`.

# Errores no recuperables con `panic!`

En el caso de un error inesperado del cual no se pueda recuperara, la macro `panic!` induce un pánico. Dicho pánico terminara abruptamente el hilo actual de ejecución, y proporcionará un error:

```rust,ignore
panic!("boom");
```

resulta en

```text
thread '<main>' panicked at 'boom', hello.rs:2
```

cuando lo ejecutas.

Debido a que estas situaciones son relativamente raras, usa los pánicos con moderación.

# Promoviendo fallas a pánicos

En ciertas circunstancias, aun sabiendo que una función puede fallar, podríamos querer tratar la falla como un pánico. Por ejemplo, `io::stdin().read_line(&mut buffer)` retorna un `Result<usize>`, cuando hay un error leyendo la linea. Esto nos permite manejar y posiblemente recuperar en caso de error.

Si no queremos manejar el error, y en su lugar simplemente abortar el programa, podemos usar el método `unwrap()`:

```rust,ignore
io::stdin().read_line(&mut buffer).unwrap();
```

`unwrap()` hara un pánico (`panic!`) si el `Result` es `Err`. Esto básicamente dice "Dame el valor, y si algo sale mal, simplemente aborta la ejecución". Esto es menos confiable que hacer match en el error y tratar de recuperarnos, pero al mismo tiempo es significativamente mas corto. Algunas veces, la terminación abrupta es apropiada.

Hay una manera que de hacer lo anterior que es un poco mejor que `unwrap()`:


```rust,ignore
let mut bufer = String::new();
let bytes_leidos = io::stdin().read_line(&mut bufer)
                                .ok()
                                .expect("Fallo al leer linea");
```

`ok()` convierte el `Result` en un `Option`, y `expect()` hace lo mismo que `unwrap()`, pero recibe un mensaje como argumento. Este mensaje es pasado a el `panic!` subyacente, proporcionando un mejor mensaje de error.


# Usando `try!`

Cuando escribimos código que llama a muchas funciones que retornan el tipo `Result`, el manejo de errores se puede tornar tedioso. La macro `try!` esconde algo de el código repetitivo correspondiente a la propagación de errores en la pila de llamadas.

Este reemplaza:

```rust
use std::fs::File;
use std::io;
use std::io::prelude::*;

struct Info {
    nombre: String,
    edad: i32,
    grado: i32,
}

fn escribir_info(info: &Info) -> io::Result<()> {
    let mut archivo = File::create("mis_mejores_amigos.txt").unwrap();

    if let Err(e) = writeln!(&mut archivo, "nombre: {}", info.nombre) {
        return Err(e)
    }
    if let Err(e) = writeln!(&mut archivo, "edad: {}", info.edad) {
        return Err(e)
    }
    if let Err(e) = writeln!(&mut archivo, "grado: {}", info.rgrado) {
        return Err(e)
    }

    return Ok(());
}
```

Con esto:

```rust
use std::fs::File;
use std::io;
use std::io::prelude::*;

struct Info {
    nombre: String,
    edad: i32,
    grado: i32,
}

fn escribir_info(info: &Info) -> io::Result<()> {
    let mut file = File::create("mis_mejores_amigos.txt").unwrap();

    try!(writeln!(&mut archivo, "nombre: {}", info.name));
    try!(writeln!(&mut archivo, "edad: {}", info.age));
    try!(writeln!(&mut archivo, "grado: {}", info.rating));

    return Ok(());
}
```

Envolver una expresión con `try!` resultara en el valor (`Ok`) exitoso desenvuelto, a menos que el resultado sea `Err`, caso en el cual `Err` es retornado de manera temprana por la función que envuelve al try.

Es importante hacer mención a el hecho de que solo puedes usar `try!` desde una función que retorna un `Result`, lo que se traduce en que no puedes usar `try!` dentro de `main()`, debido a que `main()` no retorna nada.

`try!` hace uso de [`From<Error>`](../std/convert/trait.From.html) (ingles) para determinar que retornar en el caso de error.
