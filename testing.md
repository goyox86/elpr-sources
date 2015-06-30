% Pruebas

> Probar programas puede ser una forma efectiva de mostrar la presencia de bugs, pero es desesperanzadamente inadecuada para mostrar su ausencia.
> Edsger W. Dijkstra, "The Humble Programmer" (1972)

Hablaremos acerca de como probar código Rust. De lo que no estaremos hablando es acerca de la manera correcta de probar código Rust. Hay muchas escuelas de pensamiento en relación a la forma correcta e incorrecta de escribir pruebas. Todos esos enfoques usan las mismas herramientas básicas, en esta sección te mostraremos la sintaxis para hacer uso de ellas.

# El atributo `test`

En esencia, una prueba en Rust es una función que esta anotada con el atributo `test`. Vamos a crear un nuevo proyecto llamado `sumador` con Cargo:

```bash
$ cargo new sumador
$ cd adder
```
Cargo generara automáticamente una prueba simple cuando creas un proyecto nuevo.
He aqui el contenido de `src/lib.rs`:

```rust
#[test]
fn it_works() {
}
```

Nota el `#[test]`. Este atributo indica que esta es una función de prueba. Actualmente no tiene cuerpo. Pero eso es suficiente para pasar! Podemos ejecutar los tests con `cargo test`:

```bash
$ cargo test
  Compiling sumador v0.1.0 (file:///Users/goyox86/Code/rust/sumador)
     Running target/debug/sumador-ba17f4f6708ca3b9

running 1 test
test it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

   Doc-tests sumador

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

Cargo compilo y ejecuto nuestros tests. Hay dos conjuntos de salida aquí: uno para las pruebas que nosotros escribimos, y otro para los tests de documentación. Hablaremos acerca de estos mas tarde. Por ahora veamos esta linea:

```text
test it_works ... ok
```

Nota el `it_works`. Proviene del nombre de nuestra función:

```rust
fn it_works() {
# }
```

También obtenemos una linea de resumen:

```text
test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured
```

Entonces, porque nuestras pruebas vacías pasan? Cualquier prueba que no haga `panic!` pasa, y cualquier prueba que hace `panic!` falla. Hagamos fallar a nuestra prueba:

```rust
#[test]
fn it_works() {
    assert!(false);
}
```

`assert!` es una macro proporcionada por Rust la cual toma un argumento: si el argumento es `true`, nada pasa. Si el argumento es `false`, `assert!` hace `panic!`. Ejecutemos nuestras pruebas otra vez:

```bash
$ cargo test
   Compiling sumador v0.1.0 (file:///Users/goyox86/Code/rust/sumador)
     Running target/debug/sumador-ba17f4f6708ca3b9

running 1 test
test it_works ... FAILED

failures:

---- it_works stdout ----
	thread 'it_works' panicked at 'assertion failed: false', src/lib.rs:3



failures:
    it_works

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured

thread '<main>' panicked at 'Some tests failed', /Users/rustbuild/src/rust-buildbot/slave/stable-dist-rustc-mac/build/src/libtest/lib.rs:259
```

Rust nos indica que nuestra prueba ha fallado:

```text
test it_works ... FAILED
```

Y se refleja en la linea de resumen:

```text
test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured
```

También obtenemos un valor de retorno diferente a cero:

```bash
$ echo $?
101
```

Esto es muy útil para integrar `cargo test` con otras herramientas.

Podemos invertir la falla de nuestras pruebas con otro atributo: `should_panic`:

```rust
#[test]
#[should_panic]
fn it_works() {
    assert!(false);
}
```

Estas pruebas tendrán éxito si hacemos `panic!` y fallaran si se completan. Probemos:


```bash
$ cargo test
   Compiling sumador v0.1.0 (file:///Users/goyox86/Code/rust/sumador)
     Running target/debug/sumador-ba17f4f6708ca3b9

running 1 test
test it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

   Doc-tests sumador

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

Rust proporciona otra macro, `assert_eq!`, que compara dos argumentos para verificar igualdad:


```rust
#[test]
#[should_panic]
fn it_works() {
    assert_eq!("Hola", "mundo");
}
```

Esta prueba pasa o falla? Debido a la presencia del atributo `should_panic`, pasa:

```bash
$ cargo test
   Compiling sumador v0.1.0 (file:///Users/goyox86/Code/rust/sumador)
     Running target/debug/sumador-ba17f4f6708ca3b9

running 1 test
test it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

   Doc-tests sumador

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

Las pruebas `should_panic` pueden ser frágiles, es difícil garantizar que la prueba no falló por una razón inesperada. Para ayudar en esto, un parámetro opcional `expected` puede ser agregado a el atributo `should_panic`. La prueba se asegurara que el mensaje de error contenga el mensaje proporcionado. Una versión mas segura de la prueba seria:

```rust
#[test]
#[should_panic(expected = "assertion failed")]
fn it_works() {
    assert_eq!("Hola", "mundo");
}
```

Eso fue todo para lo básico! Escribamos una prueba 'real':

```rust,ignore
pub fn suma_dos(a: i32) -> i32 {
    a + 2
}

#[test]
fn it_works() {
    assert_eq!(4, suma_dos(2));
}
```

Este es un uso muy común de `assert_eq!`: llamar alguna función con algunos argumentos conocidos y comparar la salida de dicha llamada con la salida esperada.

# El modulo `tests`

Hay una forma en la cual nuestro ejemplo no es idiomático: le falta el modulo `tests`. La manera idiomática de escribir nuestro ejemplo luce así:

```rust,ignore
pub fn suma_dos(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::suma_dos;

    #[test]
    fn it_works() {
        assert_eq!(4, suma_dos(2));
    }
}
```

Hay unos cuantos cambios acá. El primero es la inclusion de un `mod tests` con un atributo `cfg`. El modulo nos permite agrupar todas nuestras pruebas, y también nos permite definir funciones de soporte de ser necesario, todo eso no forma parte de nuestro crate. El atributo `cfg` solo compila nuestro código de pruebas si estuviéramos intentando correr las pruebas. Esto puede ahorrar tiempo de compilación, también se asegura que nuestras pruebas queden completamente excluidas de una compilación normal.

El segundo cambio es la declaración `use`. Debido a que estamos en un modulo interno, necesitamos hace disponible a nuestra prueba dentro de nuestro ámbito actual. Esto puede ser molesto si posees un modulo grande, y es por ello que es común el uso de la facilidad `glob`. Cambiemos nuestro `src/lib.rs` para que haga uso de ello:

```rust,ignore

pub fn suma_dos(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(4, suma_dos(2));
    }
}
```

Nota la linea `use` diferente. Ahora ejecutamos nuestras pruebas:


```bash
$ cargo test
   Compiling sumador v0.1.0 (file:///Users/goyox86/Code/rust/sumador)
     Running target/debug/sumador-ba17f4f6708ca3b9

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

   Doc-tests sumador

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

Funciona!

La convención actual es usar el modulo `tests` para contener tus pruebas de "estilo-unitario". Cualquier cosa que solo pruebe un pequeño pedazo de funcionalidad va aquí. Pero que acerca de las pruebas "estilo-integracion"? Para ellas, tenemos el directorio `tests`.

# El directorio `tests`

Para escribir una prueba de integración, creemos un directorio `tests` y coloquemos un archivo `tests/lib.rs` dentro, con el siguiente de contenido:

```rust,ignore
extern crate sumador;

#[test]
fn it_works() {
    assert_eq!(4, sumador::suma_dos(2));
}
```

Luce similar a nuestras pruebas anteriores, pero ligeramente diferente. Ahora tenemos un `extern crate sumador` al principio. Esto es debido a que las pruebas en el directorio son un crate separado, entonces debemos importar nuestra biblioteca. Esto es también el porque `tests` es un lugar idóneo pera escribir tests de integración: estas pruebas usan la biblioteca justo como cualquier otro consumidor lo haría.

Ejecutemoslas:

```bash
$ cargo test
   Compiling sumador v0.1.0 (file:///Users/goyox86/Code/rust/sumador)
     Running target/debug/lib-f71036151ee98b04

running 1 test
test it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

     Running target/debug/sumador-ba17f4f6708ca3b9

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

   Doc-tests sumador

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

Ahora tenemos tres secciones: nuestras pruebas anteriores también fueron ejecutadas, junto con la nueva prueba de integración.

Eso fue todo para el directorio `tests`. El modulo `tests` no es necesario aquí, debido a que el modulo completo esta dedicado a pruebas.

Finalmente echemos un vistazo a esa tercera sección: pruebas de documentación.

# Pruebas de documentación

Nada es mejor que documentación con ejemplos. Nada es peor que ejemplos que no funcionan, debido a que el código a cambiado desde que la documentación fue escrita. Respecto a esto, Rust soporta la ejecución automática de los ejemplos presentes en tu documentación. He aquí un `src/lib.rs` pulido con ejemplos:

```rust,ignore
//! The `adder` crate provides functions that add numbers to other numbers.
//!
//! # Examples
//!
//! ```
//! assert_eq!(4, adder::add_two(2));
//! ```

/// This function adds two to its argument.
///
/// # Examples
///
/// ```
/// use adder::add_two;
///
/// assert_eq!(4, add_two(2));
/// ```
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(4, add_two(2));
    }
}
```

Nota la documentación a nivel de modulo con `//!` y la documentación a nivel de función con `///`. La documentación de Rust soporta Markdown en comentarios y graves (\`\`\`) triples delimitan bloques de código. Es convencional incluir la sección `# Examples`, exactamente asi, seguida por los ejemplos.

Ejecutemos las pruebas nuevamente:

```bash
$ cargo test
  Compiling sumador v0.1.0 (file:///Users/goyox86/Code/rust/sumador)
     Running target/debug/lib-f71036151ee98b04

running 1 test
test it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

     Running target/debug/sumador-ba17f4f6708ca3b9

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

   Doc-tests sumador

running 2 tests
test _0 ... ok
test suma_dos_0 ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured
```

Ahora tenemos los tres tipos de pruebas corriendo! Nota los nombres de las pruebas de documentación: el `_0` es generado para la prueba del modulo, y `suma_dos_0` para la prueba de función. Estos números se auto incrementaran con nombres como `suma_dos_1` a medida que mas ejemplos son agregados.
