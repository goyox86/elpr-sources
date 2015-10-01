% Traits

Un trait es una facilidad del lenguaje que le dice al compilador de Rust acerca de la funcionalidad que un tipo debe proveer.

Recuerdas la palabra reservada `impl`, usada para llamar una funcion con la [sintaxis de metodos][methodsyntax]?

```rust
struct Circulo {
    x: f64,
    y: f64,
    radio: f64,
}

impl Circulo {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radio * self.radio)
    }
}
```

[methodsyntax]: method-syntax.html

Los traits son similares, excepto que definimos un trait con solo la firma de metodo y luego implementamos el trait pata esa estructura. Asi:

```rust
struct Circulo {
    x: f64,
    y: f64,
    radio: f64,
}

trait TieneArea {
    fn area(&self) -> f64;
}

impl TieneArea for Circulo {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radio * self.radio)
    }
}
```

Como puedes ver, el bloque `trait` luce muy similar a el bloque `impl`, pero no definimos un bloque, solo la firma de tipos. Cuando implementamos un trait, usamos `impl Trait for Item`, en vez de solo `impl Item`.

## Limites trait para funciones genericas

Los traits son utiles porque permiten a un tipo hacer ciertas promesas acerca de su comportamiento. La funciones genericas pueden explotar esto para restringir los tipos que aceptan. Considera esta funcion, la cual no compila:

```rust,ignore
fn imrimir_area<T>(figura: T) {
    println!("Esta figura tiene un area de {}", figura.area());
}
```

Rust se queja:

```text
error: no method named `area` found for type `T` in the current scope
```

Debido a que `T` puede ser de cualquier tipo, no podemos estar seguros que iplementa el metodo `area`. Pero podemos agregar una ‘restriccion de trait’ a nuestro `T` generico, asegurandonos de que lo implemente:

```rust
# trait TieneArea {
#     fn area(&self) -> f64;
# }
fn imrimir_area<T: TieneArea>(shape: T) {
    println!("Esta figura tiene un area de {}", figura.area());
}
```

La sintaxis `<T: HasArea>` se traduce en “cualquier tipo que implemente el trait `TieneArea`.”. A consecuencia de que los traits definen firmas de tipos de funcion, podemos estar seguros que cualquier tipo que implemente `TieneArea` tendra un metodo `.area()`.

He aqui un ejemplo extendido de como esto funciona:

```rust
trait TieneArea {
    fn area(&self) -> f64;
}

struct Circulo {
    x: f64,
    y: f64,
    radio: f64,
}

impl TieneArea for Circulo {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radio * self.radio)
    }
}

struct Cuadrado {
    x: f64,
    y: f64,
    lado: f64,
}

impl TieneArea for Cuadrado {
    fn area(&self) -> f64 {
        self.lado * self.lado
    }
}

fn imrimir_area<T: TieneArea>(figura: T) {
    println!("Esta figura tiene un area de {}", figura.area());
}

fn main() {
    let c = Circulo {
        x: 0.0f64,
        y: 0.0f64,
        radio: 1.0f64,
    };

    let s = Cuadrado {
        x: 0.0f64,
        y: 0.0f64,
        lado: 1.0f64,
    };

    imrimir_area(c);
    imrimir_area(s);
}
```

Este programa produce la salida:

```text
Esta figura tiene un area de 3.141593
Esta figura tiene un area de 1
```

Como puedes ver, `imrimir_area` ahora es generica, pero tambien asegura que hallamos porporcionado los tipos correctos. Si pasamos un tipo incorrecto:

```rust,ignore
imrimir_area(5);
```

Obetenemos un error en tiempo de compilacion:

```text
error: the trait `TieneArea` is not implemented for the type `_` [E0277]
```

## Limites de tratit para estructuras genericas

Tus estructuras genericas pueden beneficiarse tambien de las restricciones de trait. Todo lo que necesitas es agregar la restriccion cuando declaras los tipos de parametros. A continuacion un nuevo tipo `Rectangulo<T>` y su operacion `es_cuadrado`:

```rust
struct Rectangulo<T> {
    x: T,
    y: T,
    ancho: T,
    altura: T,
}

impl<T: PartialEq> Rectangulo<T> {
    fn es_cuadrado(&self) -> bool {
        self.ancho == self.altura
    }
}

fn main() {
    let mut r = Rectangulo {
        x: 0,
        y: 0,
        ancho: 47,
        altura: 47,
    };

    assert!(r.es_cuadrado());

    r.altura = 42;
    assert!(!r.es_cuadrado());
}
```

`es_cuadrado()` necesita chequear que los lados son iguales, y para ello los tipos deben ser de un tipo que implemente el trait [`core::cmp::PartialEq`][PartialEq]:

```ignore
impl<T: PartialEq> Rectangulo<T> { ... }
```

Ahora, un rectangulo puede ser definido en funcion de cualquier tipo que pueda ser comparado por igualdad.


[PartialEq]: ../core/cmp/trait.PartialEq.html

Hemos definido una nueva estructura `Rectangulo` que acepta numeros de cualquier precision, objetos de cualquier tipo siempre y cuando puedan ser comparados por igualdad. Podriamos hacer lo mismo para nuestras estructuras `TieneArea`, `Cuadrado` y `Circulo`? Si, pero estas necesitan multiplicacion, y para trabajar con eso necesitamos saber mas de los [traits de operador][operators-and-overloading].

[operators-and-overloading]: operators-and-overloading.html

# Reglas para la implementacion de traits

Hasta ahora, solo hemos agregado implementaciones de traits a estructuras, pero puedes implementar cualquier trait para cualquier tipo. Tecnicamente, _podriamos_ implementar `TieneArea` para `i32`:

```rust
trait TieneArea {
    fn area(&self) -> f64;
}

impl TieneArea for i32 {
    fn area(&self) -> f64 {
        println!("esto es tonto");

        *self as f64
    }
}

5.area();
```

Se considera pobre estilo implementar metodos en eso tipos primitivos, aun cuando es posible.

Esto puede lucir como el viejo oeste, pero hay dos restricciones acerca de la implementacion de traits que previenen que las cosas se slagan de las manos. La primera es que si el trait no esta definido en tu ambito, no aplica. He aqui un ejemplo: la biblioteca estandar provee un trait [`Write`][write] que agrega funcionalidad extra a los `File`s, que posibilita la E/S de archivos. Por defecto, un `File` no tendria sus metodos:

[write]: ../std/io/trait.Write.html

```rust,ignore
let mut f = std::fs::File::open("foo.txt").ok().expect("No se pudo abrir foo.txt");
let buf = b"cualquier cosa"; // literal de cadena de bytes. buf: &[u8; 8]
let resultado = f.write(buf);
# resultado.unwrap(); // ignorar el error
```

He aqui el error:

```text
error: type `std::fs::File` does not implement any method in scope named `write`
let resultado = f.write(buf);
               ^~~~~~~~~~
```

Necesitamos hacer `use` del trait `Write` primero:

```rust,ignore
use std::io::Write;

let mut f = std::fs::File::open("foo.txt").ok().expect("No se pudo abrir foo.txt");
let buf = b"cualquier cosa";
let resultado = f.write(buf);
# resultado.unwrap(); // ignorar el error
```

Lo anterior compilara sin errores.

Esto significa que incluso si alguien hace algo malo como agregar metodos a `i32`, no te afectara, a menos que hagas `use` de ese trait.

Hay una restriccion mas acerca de la implementacion de traits: uno de los dos bien sea el trait o el tipo para el cual estas escribiendo la `impl`, debe ser definido por ti. Entonces, podriamos implementar el trait `TieneArea` para el tipo `i32`, puesto que `TieneArea` esta en nustro codigo. Pero si intentaramos implementar `ToString`, un trait proporcionado por Rust para `i32`, no podriamos, debido a que ni el trait o el tipo estan en nuestro codigo.

Una ultima cosa acerca de los traits: las funciones genericas con un limite de trait usan ‘monomorfizacion’ (‘monomorphization’) (mono: uno, morfos: forma), y po ello son depachadas estaticamente. Que significa esto? ECha un vistazo a el capitulo acerca de [objetos trait][to] para mas detalles.

[to]: trait-objects.html

# Multiples limites de trait

Has visto que puedes limitar un parametro de tipo generico con un trait:

```rust
fn foo<T: Clone>(x: T) {
    x.clone();
}
```

Si necesitas mas de un limite, puedes hacer uso de `+`:

```rust
use std::fmt::Debug;

fn foo<T: Clone + Debug>(x: T) {
    x.clone();
    println!("{:?}", x);
}
```

`T` necesita ahora ser ambos `Clone` y `Debug`.

# La clausula where

Escribir funciones con solo unos pocos tipos genericos y un pequeno numero de limites de trait no es tan feo, pero a medida que el numero se incrementa, la sintaxis se vuelve un poco extrana:

```rust
use std::fmt::Debug;

fn foo<T: Clone, K: Clone + Debug>(x: T, y: K) {
    x.clone();
    y.clone();
    println!("{:?}", y);
}
```

El nombre de la funcion esta lejos a la izquierda, y la lista de parametros esta lejos a la derecha. Los limites de trait se interponen en la mitad.

Rust tiene una solucion, y se llama ‘clausula `where`’:

```rust
use std::fmt::Debug;

fn foo<T: Clone, K: Clone + Debug>(x: T, y: K) {
    x.clone();
    y.clone();
    println!("{:?}", y);
}

fn bar<T, K>(x: T, y: K) where T: Clone, K: Clone + Debug {
    x.clone();
    y.clone();
    println!("{:?}", y);
}

fn main() {
    foo("Hola", "mundo");
    bar("Hola", "mundo");
}
```

`foo()` usa la sintaxis demostrada previamente, y `bar()` usa una clausula `where`. Todo lo que necesitas es dejar los limites por fuera cuando definas tus parametros de tipo y luego agregar un `where` despues de la lista de parametros. Para listas mas largas, espcios en blanco pueden ser anadidos:

```rust
use std::fmt::Debug;

fn bar<T, K>(x: T, y: K)
    where T: Clone,
          K: Clone + Debug {

    x.clone();
    y.clone();
    println!("{:?}", y);
}
```

Dicha flexibilidad puede agregar claridad en situaciones complejas.

La clausula `where` es tambien mas poderosa que la sintaxis mas simple. Por ejemplo:

```rust
trait ConvertirA<Salida> {
    fn convertir(&self) -> Salida;
}

impl ConvertirA<i64> for i32 {
    fn convertir(&self) -> i64 { *self as i64 }
}

// puede ser llamada con T == i32
fn normal<T: ConvertirA<i64>>(x: &T) -> i64 {
    x.convertir()
}

// puede ser llamada con T == i64
fn inversa<T>() -> T
        // this is using ConvertTo as if it were "ConvertTo<i64>"
        where i32: ConvertirA<T> {
    42.convertir()
}
```

Lo anterior demuestra una caracteristica adicional de `where`: permite limites en los que el lado izquierdo es un tipo arbitrario (`i32` en este casoa), no solo un simple parametro de tipo (como `T`).

# Metodos por defecto

Si ya sabes como un implementador tipico definira un metodo, puedes permitir a tu trait porporcionar uno por defecto:

```rust
trait Foo {
    fn es_valido(&self) -> bool;

    fn es_invalido(&self) -> bool { !self.es_valido() }
}
```

Los implementadores de el trait `Foo` necesitan implementar `es_valido()`, pero no necesitan implementar `es_invalido()`. Lo obtendran por defecto. Tambien pueden sobreescribir la implementacion por defecto si lo desean:

```rust
# trait Foo {
#     fn es_valido(&self) -> bool;
#
#     fn es_invalido(&self) -> bool { !self.es_valido() }
# }
struct UsaDefault;

impl Foo for UsaDefault {
    fn es_valido(&self) -> bool {
        println!("UsaDefault.es_valid llamada.");
        true
    }
}

struct SobreescribeDefault;

impl Foo for SobreescribeDefault {
    fn es_valido(&self) -> bool {
        println!("SobreescribeDefault.es_valido llamada.");
        true
    }

    fn es_invalido(&self) -> bool {
        println!("SobreescribeDefault.es_invalido llamada!");
        true // esta implementacion es una auto-contradiccion!
    }
}

let default = UsaDefault;
assert!(!default.es_valido()); // imprime "UsaDefault.es_valid llamada."

let sobre = SobreescribeDefault;
assert!(sobre.is_invalid()); // prints "SobreescribeDefault.es_invalido llamada!"
```

# Herencia

Algunas veces, implementar un trait requiere implementar otro:

```rust
trait Foo {
    fn foo(&self);
}

trait FooBar : Foo {
    fn foobar(&self);
}
```

Los implementadores de `FooBar` deben tambien implementar `Foo`, de esta manera:

```rust
# trait Foo {
#     fn foo(&self);
# }
# trait FooBar : Foo {
#     fn foobar(&self);
# }
struct Baz;

impl Foo for Baz {
    fn foo(&self) { println!("foo"); }
}

impl FooBar for Baz {
    fn foobar(&self) { println!("foobar"); }
}
```

Si olvidamos implementar `Foo`, Rust nos lo dira:

```text
error: the trait `main::Foo` is not implemented for the type `main::Baz` [E0277]
```
