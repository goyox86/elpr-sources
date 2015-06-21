# Documentación

La documentación es una parte importante de cualquier proyecto de software y es un ciudadano de primera clase en Rust. Hablemos acerca de las herramientas que Rust the proporciona para documentar tus proyectos.

## Acerca de `rustdoc`

La distribución de Rust incluye una herramienta, `rustdoc`, que genera la documentación. `rustdoc` es también usado por Cargo a través de `cargo doc`.

La documentación puede ser generada de dos formas: desde el código fuente, o desde archivos Markdown.

## Documentando código fuente

La manera principal de documentar un proyecto Rust es a través de la anotación del código fuente. Puedes usar comentarios de documentación para este propósito:

```rust,ignore
/// Construye un nuevo `Rc<T>`.
///
/// # Examples
///
/// ```
/// use std::rc::Rc;
///
/// let cinco = Rc::new(5);
/// ```
pub fn new(value: T) -> Rc<T> {
    // la implementación va aqui
}
```

Este código genera documentación que luce [como esta][rc-new](version en ingles). He dejado la implementación por fuera, con un comentario normal en su lugar. Esa es la primera cosa a resaltar acerca de esta anotación: usa `///`, en vez de `//`. El slash triple indica que es un comentario de documentación.

Los comentarios de documentación son escritos en formato Markdown.

Rust mantiene un registro de esos comentarios, y los usa al momento de generar la documentación. Esto es importante cuando se documentan cosas como enumeraciones (enums):

```rust
/// El tipo `Option`. Vea [la documentación a nivel de modulo](../) para mas información.
enum Option<T> {
    /// Ningún valor
    None,
    /// Algún valor `T`
    Some(T),
}
```

Lo anterior funciona, pero esto no:

```rust,ignore
/// El tipo `Option`. Vea [la documentación a nivel de modulo](../) para mas información.
enum Option<T> {
    None, ///  Ningún valor
    Some(T), /// Algún valor `T`
}
```

Obtendras un error:

```text
hola.rs:4:1: 4:2 error: expected ident, found `}`
hola.rs:4 }
           ^
```

Este [desafortunado error](https://github.com/rust-lang/rust/issues/22547) es correcto: los comentarios de documentación aplican solo a lo que este después de ellos, y no hay nada después del ultimo comentario.

[rc-new]: http://doc.rust-lang.org/nightly/std/rc/struct.Rc.html#method.new

### Escribiendo comentarios de documentación

De cualquier modo, cubramos cada parte de este comentario en detalle:

```rust
/// Construye un nuevo `Rc<T>`.
# fn foo() {}
```

La primera linea de un comentario de documentación debe ser un resumen corto de sus funcionalidad. Una oración. Solo lo básico. Alto nivel.

```rust
///
/// Otros detalles acerca de la construcción de `Rc<T>`s, quizás describiendo semántica
/// complicada, tal vez opciones adicionales, cualquier cosa extra.
///
# fn foo() {}
```

Nuestro ejemplo original solo tenia una linea de resumen, pero si teníamos mas cosas que decir, pudimos haber agregado mas explicación en un párrafo nuevo.

#### Secciones especiales

```rust
/// # Examples
# fn foo() {}
```

Lo siguiente, son las secciones especiales. Estas son indicadas con una cabecera, `#`. Hay tres tipos de cabecera que son usados comúnmente. Estos no son sintaxis especial, solo convención, por ahora.

```rust
/// # Panics
# fn foo() {}
```

Malos usos irrecuperables de una función (e.j. errores de programación) en Rust son usualmente indicados por pánicos (panics), los cuales matan el hilo actual por lo mínimo. Si tu función posee un contrato no trivial como este, que es detectado/impuesto por pánicos, documentarlo es muy importante.

```rust
/// # Failures
# fn foo() {}
```

Si tu función o método retorna un `Result<T, E>`, entonces describir las condiciones bajo las cuales retorna `Err(E)` es una cosa buena por hacer. Esto es ligeramente menos importante que `Panics`, a consecuencia de que es codificado en el sistema de tipos, pero es aun, algo recomendable por hacer.

```rust
/// # Safety
# fn foo() {}
```

Si tu función es `unsafe` (insegura), deberías explicar cuales son las invariables deben ser mantenidas por el llamador.

```rust
/// # Examples
///
/// ```
/// use std::rc::Rc;
///
/// let cinco = Rc::new(5);
/// ```
# fn foo() {}
```

Tercero, `Examples`, Incluye uno o mas ejemplo del uso de tu función o método, y tus usuarios te querrán por ello. Estos ejemplo van dentro de anotaciones de bloques de código, de los cuales hablaremos en un monto, y que pueden tener mas de una sección:


```rust
/// # Examples
///
/// Patrones `&str` simples:
///
/// ```
/// let v: Vec<&str> = "Mary tenia un pequeno cordero".split(' ').collect();
/// assert_eq!(v, vec!["Mary", "tenia", "un", "pequeno", "cordero"]);
/// ```
///
/// Patrones mas complejos con lambdas:
///
/// ```
/// let v: Vec<&str> = "abc1def2ghi".split(|c: char| c.is_numeric()).collect();
/// assert_eq!(v, vec!["abc", "def", "ghi"]);
/// ```
# fn foo() {}
```

Discutamos los detalles de esos bloques de código.

#### Anotaciones de bloques de código

To write some Rust code in a comment, use the triple graves:

```rust
/// ```
/// println!("Hello, world");
/// ```
# fn foo() {}
```

Si quieres algo que no es codigo Rust, puedes agregar una anotación

```rust
/// ```c
/// printf("Hola, mundo\n");
/// ```
# fn foo() {}
```

Esto sera resaltado de acuerdo al lenguaje que estés mostrando. Si solo estas mostrando texto plano, usa `text`.

Es importante elegir la anotación correcta aquí, debido a que `rustdoc` la usa de manera interesante: Puede ser usada para probar tus ejemplos, de manera tal que no se desactualicen. Si tienes algún código C pero `rustdoc` piensa que es Rust es porque olvidaste la anotación, `rustdoc` se quejara al momento de tratar de generar la documentación.

It's important to choose the correct annotation here, because `rustdoc` uses it
in an interesting way: It can be used to actually test your examples, so that
they don't get out of date. If you have some C code but `rustdoc` thinks it's
Rust because you left off the annotation, `rustdoc` will complain when trying to
generate the documentation.

## Documentacion como pruebas

Discutamos nuestra documentación de ejemplo:

```rust
/// ```
/// println!("Hola, mundo");
/// ```
# fn foo() {}
```

Notaras que no necesitas una `fn main()` o algo mas aquí. `rustdoc` agregara un main() automaticamente alrededor de tu código, y en el lugar correcto. Por ejemplo:

```rust
/// ```
/// use std::rc::Rc;
///
/// let cinco = Rc::new(5);
/// ```
# fn foo() {}
```

Esto terminara probando:

```rust
fn main() {
    use std::rc::Rc;
    let cinco = Rc::new(5);
}
```

He aquí el algoritmos completo que rustdoc usa para post-procesar los ejemplos:

1. Cualquier atributo `#![foo]` sobrante es dejado intacto como atributo del crate.
2. Algunos atributos comunes son insertados, incluyendo `unused_variables`, `unused_assignments`, `unused_mut`,  `unused_attributes`, y `dead_code`. Ejemplos pequeños ocasionalmente disparan estos lints.
3. Si el ejemplo no contiene `extern crate`, entonces el `extern crate <micrate>;` es insertado.
4. Finalmente, si el ejemplo no contiene `fn main`, el texto es envuelto en `fn main() { tu_codigo }`

Algunas veces, esto no es suficiente. Por ejemplo, todos estos ejemplos de código con `///` de los que hemos estado hablando? El texto plano:

```text
/// Algo de documentación.
# fn foo() {}
```

luce diferente a la salida:

```rust
/// Algo de documentación.
# fn foo() {}
```

Si, eso es correcto: puedes agregar lineas que comienzan con `# `, y estas serán eliminadas de la salida, pero serán usadas en la compilación de tu código. Puedes usar esto para tu ventaja. En este caso, los comentarios de documentación necesitan aplicar a algún tipo de función, entonces si quiero mostrar solo un comentario de documentación,

Yes, that's right: you can add lines that start with `# `, and they will
be hidden from the output, but will be used when compiling your code. You
can use this to your advantage. In this case, documentation comments need
to apply to some kind of function, so if I want to show you just a
documentation comment, I need to add a little function definition below
it. At the same time, it's just there to satisfy the compiler, so hiding
it makes the example more clear. You can use this technique to explain
longer examples in detail, while still preserving the testability of your
documentation. For example, this code:

```rust
let x = 5;
let y = 6;
println!("{}", x + y);
```

Here's an explanation, rendered:

First, we set `x` to five:

```rust
let x = 5;
# let y = 6;
# println!("{}", x + y);
```

Next, we set `y` to six:

```rust
# let x = 5;
let y = 6;
# println!("{}", x + y);
```

Finally, we print the sum of `x` and `y`:

```rust
# let x = 5;
# let y = 6;
println!("{}", x + y);
```

Here's the same explanation, in raw text:

> First, we set `x` to five:
>
> ```text
> let x = 5;
> # let y = 6;
> # println!("{}", x + y);
> ```
>
> Next, we set `y` to six:
>
> ```text
> # let x = 5;
> let y = 6;
> # println!("{}", x + y);
> ```
>
> Finally, we print the sum of `x` and `y`:
>
> ```text
> # let x = 5;
> # let y = 6;
> println!("{}", x + y);
> ```

By repeating all parts of the example, you can ensure that your example still
compiles, while only showing the parts that are relevant to that part of your
explanation.

### Documenting macros

Here’s an example of documenting a macro:

```rust
/// Panic with a given message unless an expression evaluates to true.
///
/// # Examples
///
/// ```
/// # #[macro_use] extern crate foo;
/// # fn main() {
/// panic_unless!(1 + 1 == 2, “Math is broken.”);
/// # }
/// ```
///
/// ```should_panic
/// # #[macro_use] extern crate foo;
/// # fn main() {
/// panic_unless!(true == false, “I’m broken.”);
/// # }
/// ```
#[macro_export]
macro_rules! panic_unless {
    ($condition:expr, $($rest:expr),+) => ({ if ! $condition { panic!($($rest),+); } });
}
# fn main() {}
```

You’ll note three things: we need to add our own `extern crate` line, so that
we can add the `#[macro_use]` attribute. Second, we’ll need to add our own
`main()` as well. Finally, a judicious use of `#` to comment out those two
things, so they don’t show up in the output.

### Running documentation tests

To run the tests, either

```bash
$ rustdoc --test path/to/my/crate/root.rs
# or
$ cargo test
```

That's right, `cargo test` tests embedded documentation too. However, 
`cargo test` will not test binary crates, only library ones. This is
due to the way `rustdoc` works: it links against the library to be tested,
but with a binary, there’s nothing to link to.

There are a few more annotations that are useful to help `rustdoc` do the right
thing when testing your code:

```rust
/// ```ignore
/// fn foo() {
/// ```
# fn foo() {}
```

The `ignore` directive tells Rust to ignore your code. This is almost never
what you want, as it's the most generic. Instead, consider annotating it
with `text` if it's not code, or using `#`s to get a working example that
only shows the part you care about.

```rust
/// ```should_panic
/// assert!(false);
/// ```
# fn foo() {}
```

`should_panic` tells `rustdoc` that the code should compile correctly, but
not actually pass as a test.

```rust
/// ```no_run
/// loop {
///     println!("Hello, world");
/// }
/// ```
# fn foo() {}
```

The `no_run` attribute will compile your code, but not run it. This is
important for examples such as "Here's how to start up a network service,"
which you would want to make sure compile, but might run in an infinite loop!

### Documenting modules

Rust has another kind of doc comment, `//!`. This comment doesn't document the next item, but the enclosing item. In other words:

```rust
mod foo {
    //! This is documentation for the `foo` module.
    //!
    //! # Examples

    // ...
}
```

This is where you'll see `//!` used most often: for module documentation. If
you have a module in `foo.rs`, you'll often open its code and see this:

```rust
//! A module for using `foo`s.
//!
//! The `foo` module contains a lot of useful functionality blah blah blah
```

### Documentation comment style

Check out [RFC 505][rfc505] for full conventions around the style and format of
documentation.

[rfc505]: https://github.com/rust-lang/rfcs/blob/master/text/0505-api-comment-conventions.md

## Other documentation

All of this behavior works in non-Rust source files too. Because comments
are written in Markdown, they're often `.md` files.

When you write documentation in Markdown files, you don't need to prefix
the documentation with comments. For example:

```rust
/// # Examples
///
/// ```
/// use std::rc::Rc;
///
/// let five = Rc::new(5);
/// ```
# fn foo() {}
```

is just

~~~markdown
# Examples

```
use std::rc::Rc;

let five = Rc::new(5);
```
~~~

when it's in a Markdown file. There is one wrinkle though: Markdown files need
to have a title like this:

```markdown
% The title

This is the example documentation.
```

This `%` line needs to be the very first line of the file.

## `doc` attributes

At a deeper level, documentation comments are sugar for documentation attributes:

```rust
/// this
# fn foo() {}

#[doc="this"]
# fn bar() {}
```

are the same, as are these:

```rust
//! this

#![doc="/// this"]
```

You won't often see this attribute used for writing documentation, but it
can be useful when changing some options, or when writing a macro.

### Re-exports

`rustdoc` will show the documentation for a public re-export in both places:

```ignore
extern crate foo;

pub use foo::bar;
```

This will create documentation for bar both inside the documentation for the
crate `foo`, as well as the documentation for your crate. It will use the same
documentation in both places.

This behavior can be suppressed with `no_inline`:

```ignore
extern crate foo;

#[doc(no_inline)]
pub use foo::bar;
```

### Controlling HTML

You can control a few aspects of the HTML that `rustdoc` generates through the
`#![doc]` version of the attribute:

```rust
#![doc(html_logo_url = "http://www.rust-lang.org/logos/rust-logo-128x128-blk-v2.png",
       html_favicon_url = "http://www.rust-lang.org/favicon.ico",
       html_root_url = "http://doc.rust-lang.org/")]
```

This sets a few different options, with a logo, favicon, and a root URL.

## Generation options

`rustdoc` also contains a few other options on the command line, for further customization:

- `--html-in-header FILE`: includes the contents of FILE at the end of the
  `<head>...</head>` section.
- `--html-before-content FILE`: includes the contents of FILE directly after
  `<body>`, before the rendered content (including the search bar).
- `--html-after-content FILE`: includes the contents of FILE after all the rendered content.

## Security note

The Markdown in documentation comments is placed without processing into
the final webpage. Be careful with literal HTML:

```rust
/// <script>alert(document.cookie)</script>
# fn foo() {}
```

