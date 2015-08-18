% Comentarios

Ahora que hemos vistos algunas funciones, es buena idea aprender sobre los comentarios. Los comentarios son notas que dejas a otros programadores con la finalidad de explicar algún aspecto de tu código. El compilador mayormente, los ignora.

Rust posee dos tipos de comentarios que te deben interesar: *comentarios de linea* y *comentarios de de documentación*

```rust
// Los comentarios de linea son cualquier cosa después de ‘//’ y se extienden hasta el final de la linea

let x = 5; // este es también un comentario de linea

// Si tienes una larga explicación acerca de algo, puedes colocar comentarios
// de linea unos juntos. Pon un espacio entre tus // y tu comentario con el
// fin de hacerlos mas legibles.
```

El otro tipo de comentario es un comentario de documentación (o comentario doc). Los comentarios doc usan  `///` en lugar de `//`, y soportan notación Markdown en su interior:

```rust
/// Suma uno al numero proporcionado
///
/// # Examples
///
/// ```
/// let cinco = 5;
///
/// assert_eq!(6, suma_uno(5));
/// # fn suma_uno(x: i32) -> i32 {
/// #     x + 1
/// # }
/// ```
fn suma_uno(x: i32) -> i32 {
    x + 1
}
```

Existe otro estilo de comentarios, `//!`, con la finalidad de comentar los items contenedores (e.j crates, modulos o funciones), en lugar de los items que los siguen. Son usados comúnmente en la raiz de un crate (lib.rb) o en la raiz de un modulo (mod.rs):

```
//! # La Biblioteca Estándar de Rust
//!
//! La Biblioteca Estándar de Rust proporciona la funcionalidad
//! en tiempo de ejecución esencial para la construcción de software
//! Rust portable.
```

Cuando se escriben comentarios doc, proporcionar algunos ejemplos de uso es muy, muy util. Notaras que hemos hecho uso de una macro: `assert_eq!`. Esta compara dos valores, y hace `panic!`o si estos no son iguales. Es muy util en la documentación. También existe otra macro, `assert!`, la cual hace `panic!`o si el valor proporcionado es `false`.

Puedes usar la herramienta [`rustdoc`](documentation.html) para generar documentación en HTML a partir de dichos comentarios, así como ejecutar el código de los ejemplos como pruebas!
