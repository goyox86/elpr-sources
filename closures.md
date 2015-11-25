% Closures

Algunas veces es util envolver una funcion y sus _variables libres_ para mejor claridad y reusabilidad. Las variables libres que pueden ser usadas provienen del ambito exterior y son ‘cerradas’ (‘closed over’) cuando son usadas en la funcion. De alli el nombre ‘closure’. Rust provee una muy buena implementacion de ellos, como veremos a continuacion.

# Sintaxis

Los closures lucen asi:

```rust
let suma_uno = |x: i32| x + 1;

assert_eq!(2, suma_uno(1));
```

Creamos un enlace a variable, `suma_uno` y lo asignamos a un closure. Los argumentos del closure van entre pipes  (`|`), y el cuerpo es una expresion, en este caso, `x + 1`. Recuerda que , `{ }` es una expresion, de manrea que podemos tener tambien closures multi-linea:

```rust
let suma_dos = |x| {
    let mut resultado: i32 = x;

    resultado += 1;
    resultado += 1;

    resultado
};

assert_eq!(4, suma_dos(2));
```

Notaras un par de cosas acerca de los closures que son un poco diferentes de las funciones regulares definidas con `fn`. Lo primero es que no necesitamos anotar los tipos de los arguymentos del closure o los valores que este retorna. Podemos:

```rust
let suma_uno = |x: i32| -> i32 { x + 1 };

assert_eq!(2, suma_uno(1));
```

Pero no necesitamos hacerlo. Porque esto? Basicamente, se implemento de esa manera por razones de ergonomia. Si bien especificar el tipo completo para funciones con nombre es de utilidad para cosas como documentacion e inferencia de tipos, la firma completa de los closure es raramente documentada puesto a que son anonimos, y no causan los problemas de tipo de error-a-distancia que la inferencia en funciones con nombre pueden causar.

La segunda sintaxis es similar, pero un tanto diferente. He agredado espacios aca para una mas facil comparacion:

```rust
fn  suma_uno_v1   (x: i32) -> i32 { x + 1 }
let suma_uno_v2 = |x: i32| -> i32 { x + 1 };
let suma_uno_v3 = |x: i32|          x + 1  ;
```

Pequenas diferencias, pero son similares.

# Closures y su entorno

El entorno para un closure puede incluir enlaces a variable del ambito que los envuelve en adicion a los parametros y variables locales. Luce de esta manera:

```rust
let num = 5;
let suma_num = |x: i32| x + num;

assert_eq!(10, suma_num(5));
```

El closure `suma_num`, hace referencia a el enlace `let` en su ambito: `num`. Mas especificamente, toma prestado el enlace. Si hacemos algo que resulte en un conflicto con dicho enlace, obtendriamos un error como este:

This closure, `plus_num`, refers to a `let` binding in its scope: `num`. More
specifically, it borrows the binding. If we do something that would conflict
with that binding, we get an error. Like this one:

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

Un error un tanto verbose pero igual de util! Como lo dice, no podemos tomar un prestamo mutable en `num` debido a que el closure ya esta tomandolo prestado. Si dejamos el closure fuera de ambito, entonces podemos:

```rust
let mut num = 5;
{
    let suma_num = |x: i32| x + num;

} // suma_num sale de ambito aca, el prestamo termina aqui

let y = &mut num;
```

Sin embargo, si tu closure asi lo requiere, Rust tomara pertenecia y movera el entorno. Lo siguiente no fucniona:

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

`Vec<T>` posee pertenecia de su contenido, y es por ello, que al hacer referencia a el dentro de nuestro closure, tenemos que tomar pertenencia de `nums`. Es lo mismo que si hubieramos proporcionado `nums` como argumento a una funcion que tomara pertenencia sobre el.

## Closures `move`

Podemos forzar nuestro closure a tomar pertenecia de su entorno con la palabra reservada `move`:

```rust
let num = 5;

let toma_pertenecia_num = move |x: i32| x + num;
```

Ahora, aun cuando la palabra reservada `move` esta presente, las variables siguen la semantica normal. En este caso, `5` implementa `Copy`, y en consecuencia, `toma_pertenecia_num` toma pertenecia de una copia de `num`. Entonces, cual es la diferencia?

```rust
let mut num = 5;

{
    let mut suma_num = |x: i32| num += x;

    suma_num(5);
}

assert_eq!(10, num);
```

En este caso, nuestro closure tomo una referencia mutable a `num`, y cuando llamamos a `suma_num`, este muto el valor subyacente, tal y como lo esperabamos. Tambien necesitamos declarar `suma_num` como `mut`, puesto a que estamos mutando su entorno.   

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

Otra forma de pensar acerca de los closures `move` es: estos proporcionan a el closure su propio registro de activacion. Sin `move`, dicho puede ser asociado a el registro de activacion que lo creo, mientras que un closure `move` es autocontenido. Lo que significa, por ejemplo, que generalmente no puedes retornar un closure no-`move` desde una funcion.

Pero antes de hablar de recibir closures como parametros y usarlos como valores de retorno, debemos hablar un poco mas acerca de su implementacion. Como un lenguaje de programacion de sistemas Rust te proporciona una tonelada de control acerca de lo que tu codigo hace, y los closures no son diferentes.

# Implementacion de los Closures

La implementacion de closures de Rust es un poco diferente a la de otros lenguajes. En Rust, los closures son efectivamente una sintaxis alterna para los traits. Antes de continuar, necesitaras haber leido el [capitulo de traits][traits]  asi como el capitulo acerca de [objetos trait][trait-objects].

[traits]: traits.html
[trait-objects]: trait-objects.html

Ya los has leido? Excelente.

La clave para entender como funcionan los closures es algo un poco extrano: Usar `()` para llamar una funcion, como `foo()`, es un operador sobrecargable. Partiendo desde esta premisa, todo lo demas encaja. En Rust hacemos uso de el sistema de traits para sobrecargar operadores. Llamar funciones no es diferente. Existen tres traits que podemos sobrecargar:

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

Notaras unas pocas diferencias entre dichos traits, pero una grande es `self`: `Fn` recibe `&self`, `FnMut` toma `&mut self` y `FnOnce` recibe `self`. Lo anterior cubre los tres tipos de `self` a traves de la sintaxis usual de llamadas a metodos. Pero han sido separados en tres traits, en lugar de uno solaits, en lugar de uno solo. Esto nos prroporciona gran control acerca del tipo de closures que podemos recibir.

La sintaxis `|| {}` es una sintaxis alterna para esos tres traits. Rust generara una estructura para el entorno, `impl` el trait apropiado, y luego hara uso de esta.

# Recxibiendo closures como argumentos

Ahora que sabemos que los closures son traits, entonces sabemos como aceptar y retronar closures: justo como cualquier otro trait!

Lo anterior tambien significa que podemos elegir entre despacho estatico o dinamico. Primero, creemos una funcion que reciba algo llamable, ejecute una llamada sobre el y luego retorne el resultado:

```rust
fn llamar_con_uno<F>(algun_closure: F) -> i32
    where F : Fn(i32) -> i32 {

    algun_closure(1)
}

let respuesta = llamar_con_uno(|x| x + 2);

assert_eq!(3, respuesta);
```

Pasamos nuestro closure, `|x| x + 2`, a `llamar_con_uno`. `llamar_con_uno` hace lo que sugiere: llama el closure, proporcionandole `1` como argumento.

Examinemos la firma de `llamar_con_uno` con mas detalle:

```rust
fn llamar_con_uno<F>(algun_closure: F) -> i32
#    where F : Fn(i32) -> i32 {
#    algun_closure(1) }
```

Recibimos un parametro, de tipo `F`. Tambien retornamos un `i32`. Esta parte no es interessante. La siguiente si lo es:

```rust
# fn llamar_con_uno<F>(algun_closure: F) -> i32
    where F : Fn(i32) -> i32 {
#   algun_closure(1) }
```

Debido a que `Fn` es un trait, podemos limitar nuestro generico con el. En este caso, nuestro closure recibe un `i32` y retorna un `i32`, es por ello que el limite de genericos que usamos es `Fn(i32) -> i32`.

Hay otro punto clave aca: debido a que estamos limitando un generico con un trait, la llamada sera monomorfizada, y en consecuencia, estaremos haciendo despcho estatico en el closure. Eso es super cool. En muchos lenguajes, los closures son inherentemente asignados desde el monticulo, y casi siempre involucraran despacho dinamico. En Rust podemos asignar el entorno de nuestros closures desde la pila, asi como despachar la llamada de manera estatica. Esto ocurre con bastante frecuencia con los iteradores y sus adaptadores, los cuales reciben closures como argumentos.

Por supuesto, si deseamos despacho dinamico, podemos tenerlo tambien. Un objeto trait, como es usual, maneja este caso:

```rust
fn llamar_con_uno(algun_closure: &Fn(i32) -> i32) -> i32 {
    algun_closure(1)
}

let answer = llamar_con_uno(&|x| x + 2);

assert_eq!(3, answer);
```

Ahora recibimos un objeto trait, un `&Fn`. Y tenemos que hacer una referencia a nuestro closure cuando lo pasemos a `llamar_con_uno`, es por ello que usamos `&||`.

# Apuntadores a funcion y closures

Un apuntador a funcion es una especie de closure que no posee entorno. Como consecuencia, puedes pasar un apuntador a funcion a cualquier funcion que reciba un closure como argumento:

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

En el ejemplo anterior no necesitamos la varieble intermedia `f` de manera estricta, el nombre de la funcion tambien sirve:

```ignore
let respuesta = llamar_con_uno(&suma_uno);
```

# Retornando closures

Es muy comun para codigo con estilo funcional el retornar closures en diversas situaciones. Si intentas retornar un closure, podrias incurrir en un error. Al principio puede parecer extrano, pero mas adelante lo entenderemos. Probablemente intentarias retornar un closure desde una funcion:

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

Para retornar algo desde una funcion, Rust necesita saber el tamano del tipo de retorno. Pero debido q que `Fn` es un trait, puede ser varias cosas de diversos tamanos: muchos tipos pueden impelmentar `Fn`. Una manera facil de darle tamano a algo es tomando una referencia a este, debido a que las referencias tienen un tamano conocido. En lugar de lo anterior podemos escribir:

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

Bien. Debido a que tenemos una referencia, necesitamos proporcionar un tiempo de vida. pero nuestra funcion  `factory()` no recibe ninguna argumento y por ello la [elision](lifetimes.html#lifetime-elision) no funciona en este caso. Entonces que opciones tenemos? Intentemnos con `'static`:

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

El error nos dice que no tenemos un `&'static Fn(i32) -> i32`, que tenemos un `[closure@<anon>:7:9: 7:20]`. Un momento, que?

Debido a que nuestro closure genera su propio `struct` para el entorno asi como una implementacion para `Fn`, `FnMut` y `FnOnce`, dichos tipos son anonymos. Solo existen para este closure. Es por ello que Rust los muestra como `closure@<anon>` en vez de algun nombre autogenerado.

El error tambien habla de que se espera que el tipo de retorno sea una referencia, pero lo que estamos tratando de retornar no lo es. Mas aun, no podemos asignar directamente un tiempo de vida `'static'` a un objeto. Entoces tomaremos un enfoque diferente y retornaremos un ‘trait object’ envolviendo el `Fn` en un `Box`. Lo siguiente _casi_ funciona:

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

Bueno, como discutimos anteriormente, los closures toman su entorno prestado. Y en este caso, nuestro entorno esta basaso en un `5` asignado desde la pila, la variable `num`. Debido a esto el prestamo posee el tiempo de vida del registro de activacion. De retornar este closure, la llamada a funcion podria terminar, el registro de activacion desapareceria, y nuestro closure estaria capturando un entorno de memoria basura! Con un ultimo arreglo, podemos hacer que funcione:

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

Al hacer el closure interno un `move Fn`, hemos creado un nuevo registro de activacion para nuestro closure. Envolviendolo con un `Box`, le hemos proporcionado un tamano conocido, permitiendole escapar nuestro registro de activacion.
