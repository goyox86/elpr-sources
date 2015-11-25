% Closures

Algunas veces es util envolver una función y sus _variables libres_ para mejor claridad y reusabilidad. Las variables libres que pueden ser usadas provienen del ámbito exterior y son ‘cerradas’ (‘closed over’) cuando son usadas en la función. De allí el nombre ‘closure’. Rust provee una muy buena implementación, como veremos a continuación.

# Sintaxis

Los closures lucen así:

```rust
let suma_uno = |x: i32| x + 1;

assert_eq!(2, suma_uno(1));
```

Creamos un enlace a variable, `suma_uno` y lo asignamos a un closure. Los argumentos del closure van entre pipes (`|`), y el cuerpo es una expresión, en este caso, `x + 1`. Recuerda que `{ }` es una expresión, de manera que podemos tener también closures multi-linea:

```rust
let suma_dos = |x| {
    let mut resultado: i32 = x;

    resultado += 1;
    resultado += 1;

    resultado
};

assert_eq!(4, suma_dos(2));
```

Notaras un par de cosas acerca de los closures que son un poco diferentes de las funciones regulares definidas con `fn`. Lo primero es que no necesitamos anotar los tipos de los argumentos los valores de retorno. Podemos:

```rust
let suma_uno = |x: i32| -> i32 { x + 1 };

assert_eq!(2, suma_uno(1));
```

Pero no necesitamos hacerlo. Porque? Básicamente, se implemento de esa manera por razones de ergonomia. Si bien especificar el tipo completo para funciones con nombre es de utilidad para cosas como documentación e inferencia de tipos, la firma completa en los closures es raramente documentada puesto a que casi siempre son anónimos, y no causan los problemas de tipo de error-a-distancia que la inferencia en funciones con nombre pueden causar.

La segunda sintaxis es similar, pero un tanto diferente. He agregado espacios acá para facilitar la comparación:

```rust
fn  suma_uno_v1   (x: i32) -> i32 { x + 1 }
let suma_uno_v2 = |x: i32| -> i32 { x + 1 };
let suma_uno_v3 = |x: i32|          x + 1  ;
```

Pequenas diferencias, pero son similares.

# Closures y su entorno

El entorno para un closure puede incluir enlaces a variable del ámbito que los envuelve en adición a los parámetros y variables locales. De esta manera:

```rust
let num = 5;
let suma_num = |x: i32| x + num;

assert_eq!(10, suma_num(5));
```

El closure `suma_num`, hace referencia a el enlace `let` en su ámbito: `num`. Mas específicamente, toma prestado el enlace. Si hacemos algo que resulte en un conflicto con dicho enlace, obtendríamos un error como este:

```rust,ignore
let mut num = 5;
let suma_num = |x: i32| x + num;

let y = &mut num;
```

Que falla con:

```text
error: cannot borrow `num` as mutable because it is also borrowed as immutable
    let y = &mut num;
                 ^~~
note: previous borrow of `num` occurs here due to use in closure; the immutable
  borrow prevents subsequent moves or mutable borrows of `num` until the borrow
  ends
    let suma_num = |x| x + num;
                   ^~~~~~~~~~~
note: previous borrow ends here
fn main() {
    let mut num = 5;
    let suma_num = |x| x + num;

    let y = &mut num;
}
^
```

Un error un tanto verbose pero igual de util! Como lo dice, no podemos tomar un préstamo mutable en `num` debido a que el closure ya esta tomándolo prestado. Si dejamos el closure fuera de ámbito, entonces es posible:

```rust
let mut num = 5;
{
    let suma_num = |x: i32| x + num;

} // suma_num sale de ambito, el prestamo termina aqui

let y = &mut num;
```

Sin embargo, si tu closure así lo requiere, Rust tomara pertenecia y moverá el entorno. Lo siguiente no funciona:

```rust,ignore
let nums = vec![1, 2, 3];

let toma_nums = || nums;

println!("{:?}", nums);
```

Obtenemos este error:

```text
note: `nums` moved into closure environment here because it has type
  `[closure(()) -> collections::vec::Vec<i32>]`, which is non-copyable
let toma_nums = || nums;
                 ^~~~~~~
```

`Vec<T>` posee pertenecia de su contenido, y es por ello, que al hacer referencia a el dentro de nuestro closure, tenemos que tomar pertenencia de `nums`. Es lo mismo que si hubiéramos proporcionado `nums` como argumento a una función que tomara pertenencia sobre el.

## Closures `move`

Podemos forzar nuestro closure a tomar pertenecia de su entorno con la palabra reservada `move`:

```rust
let num = 5;

let toma_pertenecia_num = move |x: i32| x + num;
```

Ahora, aun cuando la palabra reservada `move` esta presente, las variables siguen la semántica normal. En este caso, `5` implementa `Copy`, y en consecuencia, `toma_pertenecia_num` toma pertenecia de una copia de `num`. Entonces, cual es la diferencia?

```rust
let mut num = 5;

{
    let mut suma_num = |x: i32| num += x;

    suma_num(5);
}

assert_eq!(10, num);
```

En este caso, nuestro closure tomo una referencia mutable a `num`, y cuando llamamos a `suma_num`, este muto el valor subyacente, tal y como lo esperábamos. También necesitamos declarar `suma_num` como `mut`, puesto a que estamos mutando su entorno.   

Si lo cambiamos a un closure `move`, es diferente:


```rust
let mut num = 5;

{
    let mut suma_num = move |x: i32| num += x;

    suma_num(5);
}

assert_eq!(5, num);
```

Obtenemos solo `5`. En lugar de tomar un prestamo mutable en nuestro `num`, tomamos pertenecia sobre una copia.

Otra forma de pensar acerca de los closures `move` es: estos proporcionan a el closure su propio registro de activación. Sin `move`, el closure puede ser asociado a el registro de activación que lo creo, mientras que un closure `move` es autocontenido. Lo que significa, por ejemplo, que generalmente no puedes retornar un closure no-`move` desde una función.

Pero antes de hablar de recibir closures como parámetros y usarlos como valores de retorno, debemos hablar un poco mas acerca de su implementación. Como un lenguaje de programación de sistemas Rust te proporciona una tonelada de control acerca de lo que tu código hace, y los closures no son diferentes.

# Implementación de los Closures

La implementación de closures de Rust es un poco diferente a la de otros lenguajes. En Rust, los closures son efectivamente una sintaxis alterna para los traits. Antes de continuar, necesitaras haber leído el [capitulo de traits][traits] asi como el capitulo acerca de [objetos trait][trait-objects].

[traits]: traits.html
[trait-objects]: trait-objects.html

Ya los has leído? Excelente.

La clave para entender como funcionan los closures es algo un poco extraño: Usar `()` para llamar una función, como `foo()`, es un operador sobrecargable. Partiendo desde esta premisa, todo lo demás encaja. En Rust hacemos uso de el sistema de traits para sobrecargar operadores. Llamar funciones no es diferente. Existen tres traits que podemos sobrecargar:

```rust
# mod foo {
pub trait Fn<Args> : FnMut<Args> {
    extern "rust-call" fn call(&self, args: Args) -> Self::Output;
}

pub trait FnMut<Args> : FnOnce<Args> {
    extern "rust-call" fn call_mut(&mut self, args: Args) -> Self::Output;
}

pub trait FnOnce<Args> {
    type Output;

    extern "rust-call" fn call_once(self, args: Args) -> Self::Output;
}
# }
```

Notaras unas pocas diferencias entre dichos traits, pero una grande es `self`: `Fn` recibe `&self`, `FnMut` toma `&mut self` y `FnOnce` recibe `self`. Lo anterior cubre los tres tipos de `self` a través de la sintaxis usual de llamadas a métodos. Pero han sido separados en tres traits, en lugar de uno solo. Esto nos proporciona gran control acerca del tipo de closures que podemos recibir.

La sintaxis `|| {}` es una sintaxis alterna para esos tres traits. Rust generara un `struct` para el entorno, `impl` el trait apropiado, y luego hará uso de este.

# Recibiendo closures como argumentos

Ahora que sabemos que los closures son traits, entonces sabemos como aceptar y retornar closures: justo como cualquier otro trait!

Lo anterior también significa que podemos elegir entre despacho estático o dinámico. Primero, creemos una función que reciba algo llamable, ejecute una llamada sobre el y luego retorne el resultado:

```rust
fn llamar_con_uno<F>(algun_closure: F) -> i32
    where F : Fn(i32) -> i32 {

    algun_closure(1)
}

let respuesta = llamar_con_uno(|x| x + 2);

assert_eq!(3, respuesta);
```

Pasamos nuestro closure, `|x| x + 2`, a `llamar_con_uno`. `llamar_con_uno` hace lo que sugiere: llama el closure, proporcionándole `1` como argumento.

Examinemos la firma de `llamar_con_uno` con mayor detalle:

```rust
fn llamar_con_uno<F>(algun_closure: F) -> i32
#    where F : Fn(i32) -> i32 {
#    algun_closure(1) }
```

Recibimos un parámetro, de tipo `F`. También retornamos un `i32`. Esta parte no es interesante. La siguiente lo es:

```rust
# fn llamar_con_uno<F>(algun_closure: F) -> i32
    where F : Fn(i32) -> i32 {
#   algun_closure(1) }
```

Debido a que `Fn` es un trait, podemos limitar nuestro genérico con el. En este caso, nuestro closure recibe un `i32` y retorna un `i32`, es por ello que el limite de genéricos que usamos es `Fn(i32) -> i32`.

Hay otro punto clave acá: debido a que estamos limitando un genérico con un trait, la llamada sera monomorfizada, y en consecuencia, estaremos haciendo despacho estático en el closure. Eso es super cool. En muchos lenguajes, los closures son inherentemente asignados desde el montículo, y casi siempre involucraran despacho dinámico. En Rust podemos asignar el entorno de nuestros closures desde la pila, así como despachar la llamada de manera estática. Esto ocurre con bastante frecuencia con los iteradores y sus adaptadores, quienes reciben closures como argumentos.

Por supuesto, si deseamos despacho dinámico, podemos tenerlo también. Un objeto trait, como es usual, maneja este caso:

```rust
fn llamar_con_uno(algun_closure: &Fn(i32) -> i32) -> i32 {
    algun_closure(1)
}

let answer = llamar_con_uno(&|x| x + 2);

assert_eq!(3, answer);
```

Ahora recibimos un objeto trait, un `&Fn`. Y tenemos que hacer una referencia a nuestro closure cuando lo pasemos a `llamar_con_uno`, es por ello que usamos `&||`.

# Apuntadores a función y closures

Un apuntador a función es una especie de closure que no posee entorno. Como consecuencia, podemos pasar un apuntador a función a cualquier función que reciba un closure como argumento:

```rust
fn llamar_con_uno(algun_closure: &Fn(i32) -> i32) -> i32 {
    algun_closure(1)
}

fn suma_uno(i: i32) -> i32 {
    i + 1
}

let f = suma_uno;

let respuesta = llamar_con_uno(&f);

assert_eq!(2, respuesta);
```

En el ejemplo anterior no necesitamos la variable intermedia `f` de manera estricta, el nombre de la función también sirve:

```ignore
let respuesta = llamar_con_uno(&suma_uno);
```

# Retornando closures

Es muy común para código con estilo funcional el retornar closures en diversas situaciones. Si intentas retornar un closure, podrías incurrir en un error. Al principio puede parecer extraño, pero mas adelante lo entenderemos. Probablemente intentarías retornar un closure desde una función de esta manera:

```rust,ignore
fn factory() -> (Fn(i32) -> i32) {
    let num = 5;

    |x| x + num
}

let f = factory();

let respuesta = f(1);
assert_eq!(6, respuesta);
```

Lo anterior genera estos largos, pero relacionados errores:

```text
error: the trait `core::marker::Sized` is not implemented for the type
`core::ops::Fn(i32) -> i32` [E0277]
fn factory() -> (Fn(i32) -> i32) {
                ^~~~~~~~~~~~~~~~
note: `core::ops::Fn(i32) -> i32` does not have a constant size known at compile-time
fn factory() -> (Fn(i32) -> i32) {
                ^~~~~~~~~~~~~~~~
error: the trait `core::marker::Sized` is not implemented for the type `core::ops::Fn(i32) -> i32` [E0277]
let f = factory();
    ^
note: `core::ops::Fn(i32) -> i32` does not have a constant size known at compile-time
let f = factory();
    ^
```

Para retornar algo desde una función, Rust necesita saber el tamaño del tipo de retorno. Pero debido q que `Fn` es un trait, puede ser varias cosas de diversos tamaños: muchos tipos pueden implementar `Fn`. Una manera fácil de darle tamaño a algo es tomando una referencia a este, debido a que las referencias tienen un tamaño conocido. En lugar de lo anterior podemos escribir:

```rust,ignore
fn factory() -> &(Fn(i32) -> i32) {
    let num = 5;

    |x| x + num
}

let f = factory();

let respuesta = f(1);
assert_eq!(6, respuesta);
```

Pero obtenemos otro error:

```text
error: missing lifetime specifier [E0106]
fn factory() -> &(Fn(i32) -> i32) {
                ^~~~~~~~~~~~~~~~~
```

Bien. Debido a que tenemos una referencia, necesitamos proporcionar un tiempo de vida. pero nuestra función `factory()` no recibe ningun argumento y por ello la [elision](lifetimes.html#lifetime-elision) de tiempos de vida no es posible en este caso. Entonces que opciones tenemos? Intentemos con `'static`:

```rust,ignore
fn factory() -> &'static (Fn(i32) -> i32) {
    let num = 5;

    |x| x + num
}

let f = factory();

let respuesta = f(1);
assert_eq!(6, respuesta);
```

Pero obtenemos otro error:

```text
error: mismatched types:
 expected `&'static core::ops::Fn(i32) -> i32`,
    found `[closure@<anon>:7:9: 7:20]`
(expected &-ptr,
    found closure) [E0308]
         |x| x + num
         ^~~~~~~~~~~

```

El error nos dice que no tenemos un `&'static Fn(i32) -> i32`, sino un `[closure@<anon>:7:9: 7:20]`. Un momento, que?

Debido a que nuestro closure genera su propio `struct` para el entorno así como una implementación para `Fn`, `FnMut` y `FnOnce`, dichos tipos son anónimos. Solo existen para este closure. Es por ello que Rust los muestra como `closure@<anon>` en vez de algún nombre autogenerado.

El error también habla de que se espera que el tipo de retorno sea una referencia, pero lo que estamos tratando de retornar no lo es. Mas aun, no podemos asignar directamente un tiempo de vida `'static'` a un objeto. Entonces tomaremos un enfoque diferente y retornaremos un ‘trait object’ envolviendo el `Fn` en un `Box`. Lo siguiente _casi_ funciona:

```rust,ignore
fn factory() -> Box<Fn(i32) -> i32> {
    let num = 5;

    Box::new(|x| x + num)
}
# fn main() {
let f = factory();

let respuesta = f(1);
assert_eq!(6, respuesta);
# }
```

Hay un ultimo problema:

```text
error: closure may outlive the current function, but it borrows `num`,
which is owned by the current function [E0373]
Box::new(|x| x + num)
         ^~~~~~~~~~~
```

Bueno, como discutimos anteriormente, los closures toman su entorno prestado. Y en este caso, nuestro entorno esta basado en un `5` asignado desde la pila, la variable `num`. Debido a esto el prestamo posee el tiempo de vida del registro de activación. De retornar este closure, la llamada a función podría terminar, el registro de activación desaparecería y nuestro closure estaría capturando un entorno de memoria basura! Con un ultimo arreglo, podemos hacer que funcione:

```rust
fn factory() -> Box<Fn(i32) -> i32> {
    let num = 5;

    Box::new(move |x| x + num)
}
# fn main() {
let f = factory();

let respuesta = f(1);
assert_eq!(6, respuesta);
# }
```

Al hacer el closure interno un `move Fn`, hemos creado un nuevo registro de activación para nuestro closure. Envolviéndolo con un `Box`, le hemos proporcionado un tamaño conocido, permitiéndole escapar nuestro registro de activación.
