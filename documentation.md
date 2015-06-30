% Documentación

La documentación es una parte importante de cualquier proyecto de software y un ciudadano de primera clase en Rust. Hablemos acerca de las herramientas que Rust te proporciona para documentar tus proyectos.

## Acerca de `rustdoc`

La distribución de Rust incluye una herramienta, `rustdoc`, encargada de generar la documentación. `rustdoc` es también usada por Cargo a través de `cargo doc`.

La documentación puede ser generada de dos formas: desde el código fuente, o desde archivos Markdown.

## Documentando código fuente

La principal forma de documentar un proyecto Rust es a través de la anotación del código fuente. Para este propósito, puedes usar comentarios de documentación:

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

El código anterior genera documentación que luce como [esta][rc-new](ingles). He dejado la implementación por fuera, con un comentario regular en su lugar. Esa es la primera cosa a resaltar acerca de esta anotación: usa `///`, en vez de `//`. El slash triple indica que es un comentario de documentación.

Los comentarios de documentación están escritos en formato Markdown.

Rust mantiene un registro de dichos comentarios, registro que usa al momento de generar la documentación. Esto es importante cuando se documentan cosas como enumeraciones (enums):

```rust
/// El tipo `Option`. Vea [la documentación a nivel de modulo](../) para mas información.
enum Option<T> {
    /// Ningún valor
    None
    /// Algún valor `T`
    Some(T),
}
```

Lo anterior funciona, pero esto, no:

```rust,ignore
/// El tipo `Option`. Vea [la documentación a nivel de modulo](../) para mas información.
enum Option<T> {
    None, ///  Ningún valor
    Some(T), /// Algún valor `T`
}
```

Obtendrás un error:

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

La primera linea de un comentario de documentación debe ser un resumen corto de sus funcionalidad. Una oración. Solo lo básico. De alto nivel.

```rust
///
/// Otros detalles acerca de la construcción de `Rc<T>`s, quizás describiendo semántica
/// complicada, tal vez opciones adicionales, cualquier cosa extra.
///
# fn foo() {}
```

Nuestro ejemplo original solo tenia una linea de resumen, pero si hubiésemos tenido mas cosas que decir, pudimos haber agregado mas explicación en un párrafo nuevo.

#### Secciones especiales

```rust
/// # Examples
# fn foo() {}
```

A continuación están las secciones especiales. Estas son indicadas con una cabecera, `#`. Hay tres tipos de cabecera que se usan comúnmente. Estos no son sintaxis especial, solo convención, por ahora.

```rust
/// # Panics
# fn foo() {}
```

Malos e irrecuperables usos de una función (e.j. Errores de programación) en Rust son usualmente indicados por pánicos (panics), los cuales matan el hilo actual como mínimo. Si tu función posee un contrato no trivial como este, que es detectado/impuesto por pánicos, documentarlo es muy importante.

```rust
/// # Failures
# fn foo() {}
```

Si tu función o método retorna un `Result<T, E>`, entonces describir las condiciones bajo las cuales retorna `Err(E)` es algo bueno por hacer. Esto es ligeramente menos importante que `Panics`, a consecuencia de que es codificado en el sistema de tipos, pero es aun, algo que se recomienda hacer.

```rust
/// # Safety
# fn foo() {}
```

Si tu función es `unsafe` (insegura), deberías explicar cuales son las invariantes que deben ser mantenidas por el llamador.

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

Tercero, `Examples`, incluye uno o mas ejemplos del uso de tu función o método, y tus usuarios te querrán. Estos ejemplos van dentro de anotaciones de bloques de código, de los cuales hablaremos en un momento, pueden tener mas de una sección:


```rust
/// # Examples
///
/// Patrones `&str` simples:
///
/// ```
/// let v: Vec<&str> = "Mary tenia un corderito".split(' ').collect();
/// assert_eq!(v, vec!["Mary", "tenia", "un", "corderito"]);
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

Para escribir alguna código Rust en un comentario, usa los graves triples:

```rust
/// ```
/// println!("Hola, mundo");
/// ```
# fn foo() {}
```

Si quieres código que no sea Rust, puedes agregar una anotación:

```rust
/// ```c
/// printf("Hola, mundo\n");
/// ```
# fn foo() {}
```

La sintaxis de esta sección sera resaltada de acuerdo al lenguaje que estés mostrando. Si solo estas mostrando texto plano, usa `text`.

Acá, es importante elegir la anotación correcta, debido a que `rustdoc` la usa de una manera interesante: Puede ser usada para probar tus ejemplos, de tal manera que no se vuelvan obsoletos con el tiempo. Si tienes algún código C pero `rustdoc` piensa que es Rust, es porque olvidaste la anotación, `rustdoc` se quejara al momento de tratar de generar la documentación.

## Documentación como pruebas

Discutamos nuestra documentación de ejemplo:

```rust
/// ```
/// println!("Hola, mundo");
/// ```
# fn foo() {}
```

Notaras que no necesitas una `fn main()` o algo mas. `rustdoc` agregara un main() automáticamente alrededor de tu código, y en el lugar correcto. Por ejemplo:

```rust
/// ```
/// use std::rc::Rc;
///
/// let cinco = Rc::new(5);
/// ```
# fn foo() {}
```

Se convertirá en la prueba:

```rust
fn main() {
    use std::rc::Rc;
    let cinco = Rc::new(5);
}
```

He aquí el algoritmo completo que `rustdoc` usa para post-procesar los ejemplos:

1. Cualquier atributo `#![foo]` sobrante es dejado intacto como atributo del crate.
2. Algunos atributos comunes son insertados, incluyendo `unused_variables`, `unused_assignments`, `unused_mut`,  `unused_attributes`, y `dead_code`. Ejemplos pequeños ocasionalmente disparan estos lints.
3. Si el ejemplo no contiene `extern crate`, entonces el `extern crate <micrate>;` es insertado.
4. Finalmente, si el ejemplo no contiene `fn main`, el texto es envuelto en `fn main() { tu_codigo }`

Algunas veces, todo esto no es suficiente. Por ejemplo, todos estos ejemplos de código con `///` de los que hemos estado hablando? El texto plano:

```text
/// Algo de documentación.
# fn foo() {}
```

Luce diferente a la salida:

```rust
/// Algo de documentación.
# fn foo() {}
```

Si, es correcto: puedes agregar lineas que comiencen con `# `, y estas serán eliminadas de la salida, pero serán usadas en la compilación de tu código. Puedes usar esto como ventaja. En este caso, los comentarios de documentación necesitan aplicar a algún tipo de función, entonces si quiero mostrar solo un comentario de documentación, necesito agregar una pequeña definición de función debajo. Al mismo tiempo, esta allí solo para satisfacer al compilador, de manera tal que esconderla hace el ejemplo mas limpio. Puedes usar esta técnica para explicar ejemplos mas largos en detalle, preservando aun la capacidad de tu documentación para ser probada. Por ejemplo, este código:

```rust
let x = 5;
let y = 6;
println!("{}", x + y);
```

He aquí una explicación, renderizada:

Primero, asignamos a `x` el valor de cinco:

```rust
let x = 5;
# let y = 6;
# println!("{}", x + y);
```

A continuación, asignamos seis a `y`:

```rust
# let x = 5;
let y = 6;
# println!("{}", x + y);
```

Finalmente, imprimimos la suma de  `x` y `y`:

```rust
# let x = 5;
# let y = 6;
println!("{}", x + y);
```

He aquí la misma explicación, en texto plano:

> Primero, asignamos a `x` el valor de cinco:
>
> ```text
> let x = 5;
> # let y = 6;
> # println!("{}", x + y);
> ```
>
> A continuación, asignamos seis a `y`:
>
> ```text
> # let x = 5;
> let y = 6;
> # println!("{}", x + y);
> ```
>
> Finalmente, imprimimos la suma de  `x` y `y`:
>
> ```text
> # let x = 5;
> # let y = 6;
> println!("{}", x + y);
> ```

Al repetir todas las partes del ejemplo, puedes asegurarte que tu ejemplo aun compila, mostrando solo las partes relevantes a tu explicación.

### Documentando macros

He aquí un ejemplo de la documentación a una macro:

```rust
/// Panic con un mensaje proporcionado a menos que la expression sea evaluada a true.
///
/// # Examples
///
/// ```
/// # #[macro_use] extern crate foo;
/// # fn main() {
/// panic_unless!(1 + 1 == 2, “Las mathematicas estan rotas.”);
/// # }
/// ```
///
/// ```should_panic
/// # #[macro_use] extern crate foo;
/// # fn main() {
/// panic_unless!(true == false, “Yo estoy roto.”);
/// # }
/// ```
#[macro_export]
macro_rules! panic_unless {
    ($condition:expr, $($rest:expr),+) => ({ if ! $condition { panic!($($rest),+); } });
}
# fn main() {}
```

Notaras tres cosas: necesitamos agregar nuestro propia linea `extern crate`, de tal manera que podamos agregar el atributo `#[macro_use]`. Segundo, necesitaremos agregar nuestra propia `main()`. Finalmente, un uso juicioso de `#`  para comentar esas dos cosas, de manera que nos se muestren en la salida.

### Ejecutando pruebas de documentación

Para correr las pruebas puedes:

```bash
$ rustdoc --test ruta/a/mi/crate/root.rs
# ó
$ cargo test
```

Correcto, `cargo test` prueba la documentación embebida también. Sin embargo, `cargo test`, no probara crates binarios, solo bibliotecas. Esto debido a la forma en la que `rustdoc` funciona: enlaza con la biblioteca a ser probada, pero en el caso de un binario, no hay nada a lo cual enlazar.

Hay unas pocas anotaciones mas que son útiles para ayudar a `rustdoc` a hacer la cosa correcta cuando pruebas tu código:

```rust
/// ```ignore
/// fn foo() {
/// ```
# fn foo() {}
```

La directiva `ignore` le dice a Rust que ignore el codigo. Esta es la forma que casi nunca querrás, pues es la mas genérica. En su lugar, considera el anotar con `text` de no ser codigo, o usar `#`s para obtener un ejemplo funcional que solo muestra la parte que te interesa.

```rust
/// ```should_panic
/// assert!(false);
/// ```
# fn foo() {}
```

`should_panic` le dice a `rustdoc` que el código debe compilar correctamente, pero sin la necesidad de pasar una prueba de manera satisfactoria.

```rust
/// ```no_run
/// loop {
///     println!("Hola, mundo");
/// }
/// ```
# fn foo() {}
```

El atributo `no_run` compilara tu código, pero no lo ejecutara. Esto es importante para ejemplos como "He aquí como iniciar un servicio de red," el cual debes asegurarte que compile, pero podría causar un ciclo infinito!

### Documentando módulos

Rust posee otro tipo de comentario de documentación, `//!`. Este comentario no documenta el siguiente item, este comenta el item que lo encierra. En otras palabras:

```rust
mod foo {
    //! Esta es documentation para el modulo `foo`.
    //!
    //! # Examples

    // ...
}
```

Es aquí en donde veras `//!` usado mas a menudo: para documentación de módulos. Si tienes un modulo en `foo.rs`, frecuentemente al a abrir su código veras esto:

```rust
//! Un modulo para usar `foo`s.
//!
//! El modulo `foo` contiene un monton de funcionalidad bla bla bla
```

### Estilo de comentarios de documentación

Echa un vistazo a el [RFC 505][rfc505] para un listado completo de convenciones acerca del estilo y formato de la documentación (ingles)


[rfc505]: https://github.com/rust-lang/rfcs/blob/master/text/0505-api-comment-conventions.md

## Otra documentación

Todo este comportamiento funciona en archivos no Rust también. Debido a que los comentarios son escritos en Markdown, frecuentemente son archivos `.md`.

Cuando escribes documentación en archivos Markdown, no necesitas prefijar la documentación con comentarios. Por ejemplo:

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

es solo

~~~markdown
# Examples

```
use std::rc::Rc;

let cinco = Rc::new(5);
```
~~~

cuando esta en un archivo Markdown. Solo hay un detalle, los archivos markdown necesitan tener un titulo como este:


```markdown
% Titulo

Esta es la documentación de ejemplo
```

Esta linea `%` deber estar ubicada en la primera linea del archivo.

## atributos `doc`

A un nivel mas profundo, los comentarios de documentación son otra forma de escribir atributos de documentación:

```rust
/// this
# fn foo() {}

#[doc="this"]
# fn bar() {}
```

son lo mismo que estos:

```rust
//! this

#![doc="/// this"]
```

No veras frecuentemente este atributo siendo usado para escribir documentación, pero puede ser util cuando se esten cambiando ciertas opciones, o escribiendo una macro.


### Re-exports

`rustdoc` mostrara la documentación para un re-export publico en ambos lugares:

```ignore
extern crate foo;

pub use foo::bar;
```

Lo anterior creara documentación para bar dentro de la documentación para el crate `foo`, así como la documentación para tu crate. Sera la misma documentación en ambos lugares.

Este comportamiento puede ser suprimido con `no_inline`:

```ignore
extern crate foo;

#[doc(no_inline)]
pub use foo::bar;
```

### Controlando HTML

Puedes controlar algunos aspectos de el HTML que `rustdoc` genera a través de la versión `#![doc]` del atributo:

```rust
#![doc(html_logo_url = "http://www.rust-lang.org/logos/rust-logo-128x128-blk-v2.png",
       html_favicon_url = "http://www.rust-lang.org/favicon.ico",
       html_root_url = "http://doc.rust-lang.org/")]
```

Esto configura unas pocas opciones, con un logo, favicon, y URL raíz.

## Opciones de generación

`rustdoc` también contiene unas pocas opciones en la linea de comandos, para mas personalización:

- `--html-in-header FILE`: incluye el contenido de FILE al final de la sección `<head>...</head>`.
- `--html-before-content FILE`: incluye el contenido de FILE después de `<body>`, antes del contenido renderizado (incluyendo la barra de búsqueda).
- `--html-after-content FILE`: incluye el contenido de FILE después de todo el contenido renderizado.

## Nota de seguridad

El Markdown en los comentarios de documentación es puesto sin procesar en la pagina final. Se cuidadoso con HTML literal:

```rust
/// <script>alert(document.cookie)</script>
# fn foo() {}
```
