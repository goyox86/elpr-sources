% Tipos Primitivos

Rust posee un conjunto de tipos que son considerados ‘primitivos’. Esto significa que estan integrados en el lenguaje. Rust esta estructurado de tal manera que la biblioteca estandar tambien provea una numero de tipos utiles basados en los primitivos, pero estos son los mas primitivos.

# Booleanos

Rust posee un tipo booleano integrado, denominado `bool`. Tiene dos posibles valores  `true` y `false`:

```rust
let x = true;

let y: bool = false;
```

Un uso comun de los booleanos es en [condicionales `if`][if].

[if]: if.html

Puedes encontrar mas documentacion para los `bool`eanos [en la documentacion de la biblioteca estandar][bool] (ingles).

[bool]: ../std/primitive.bool.html

# `char`

El tipo `char` representa un unico valor escalar Unicode. Puedes crear `char`s con una comilla simple: (`'`)


```rust
let x = 'x';
let two_hearts = '💕';
```

A diferencia de otros lenguajes, esto significa que `char` en Rust no es un solo byte, sino cuatro.

Puedes encontrar mas documentacion para los `char`s [en la documentacion de la biblioteca estandar][char] (ingles).

[char]: ../std/primitive.char.html

# Tipos numericos

Rust posee una variedad de tipos numericos en unas pocas catgorias: con signo y sin signo, fijos y variables, de punto flotante y enteros.

Dihcos tipos consisten de dos partes: la categoria, y el tamano. Por ejemplo, `u16` es un tipo sin signo con un tamano de dieciseis bits. Mas bits te permiten almacenar numeros mas grandes.

Si un literal de numero no posee nada que cause la inferencia de su tipo, se usan tipos por defecto:

```rust
let x = 42; // x tiene el tipo i32

let y = 1.0; // y tiene el tipo f64
```

He aqui una lista de los diferentes tipos numericos, con enlaces a su documentacion en la biblioteca estandar (ingles):

* [i8](../std/primitive.i8.html)
* [i16](../std/primitive.i16.html)
* [i32](../std/primitive.i32.html)
* [i64](../std/primitive.i64.html)
* [u8](../std/primitive.u8.html)
* [u16](../std/primitive.u16.html)
* [u32](../std/primitive.u32.html)
* [u64](../std/primitive.u64.html)
* [isize](../std/primitive.isize.html)
* [usize](../std/primitive.usize.html)
* [f32](../std/primitive.f32.html)
* [f64](../std/primitive.f64.html)

Let’s go over them by category:

## Con signo y sin signo

Los tipos de enteros vienen en dos variedades: con signo y sin signo. para entender la diferencia, consideremos un numero con cuatro bits de tamano. Un numero de cuatro bits con signo te permitiria almacenar los numeros desde `-8` a `+7`. Los numeros con signo usan “la representacion del complemento a dos”. Un numero de cuatros bits sin signo , debido a que no necesita guardar los valores negativos, puede almacenar valores desde `0` hasta `+15`.

Los tipos sin signo usan una `u` para su categoria, y los tipos con signo usan `i`. La `i` es de `integer` (entero en espanol). Entonces, `u8` es un numero de ocho bits sin signo, y un `i8` es un numero de ocho bits con signo.

## Tipos de tamano fijo

Los tipos de tamano fijo poseen un numero especifico de bits en su representacion. Los tamanos validos son `8`, `16`, `32`, and `64`. Entonces `u32` es un entero sin signo de 32 bits, e `i64` es un entero con signo de 64 bits.

## Tipos de tamano variable

Rust tambien provee tipos para los cuales el tamano depende del tamano del apuntador en la maquina subyacente. Dichos tipos poseen la categoria ‘size’, y vienen en variantes con y sin signo. Esto resulta en dos variantes `isize` y `usize`.

## Tipos de punto flotante

Rust tambien posee dos tipos de punto flotante: `f32` y `f64`. Estos corresponden a los numeros de precision simple y doble IEEE-754.

# Arreglos

Como muchos lenguajes de programacion, Rust posee tipos lista para represntar una secuencia de cosas. El mas basico es el *arreglo*, una lista de elementos del mismo tipo y de tamano fijo. Por defecto los arreglos son inmutables.

```rust
let a = [1, 2, 3]; // a: [i32; 3]
let mut m = [1, 2, 3]; // m: [i32; 3]
```

Los arreglos tienen el tipo `[T; N]`.  Hablaremos de la notacion `T` [en la seccion de genericos][generics]. La `N` es una constante en tiempo de compilacion, para la longitud del arreglo.

Hay un atajo para la inicializacion de cada uno de los elementos del arreglo a el mismo valor. En este ejemplo, cada elemento de `a` sera inicializado a `0`:

```rust
let a = [0; 20]; // a: [i32; 20]
```

Puedes obtener el numero de elementos del arreglo `a` con `a.len()`

You can get the number of elements in an array `a` with `a.len()`:

```rust
let a = [1, 2, 3];

println!("a tiene {} elementos", a.len());
```

Puedes acceder u elemento del arreglo en particular con la *notacion de subindices*:

```rust
let nombres = ["Graydon", "Brian", "Niko"]; // names: [&str; 3]

println!("El segundo nombre es: {}", nombres[1]);
```

Los subindices comienzaen en cero, como en la mayoria de los lenguajes de programacion, entonces el primer nombre es `nombres[0]` y es segundo es `nombres[1]`. El ejemplo anteior imprime: `El segundo nombre es: Brian`. Si intentas usar un subindice que no esta en el arreglo, obtendras un error: el chequeo de los limites en el acceso al arreglo se realiza en tiempo de ejecucion. El accesso errante como ese es la fuente de muchos bugs en los lenguajes de programacion de sistemas.

Puedes encontrar mas documentacion para los `array`s [en la documentacion de la biblioteca estandar][array] (ingles).

[array]: ../std/primitive.array.html

# Slices

Un slice es una referencia a (o una “vista” dentro de) otra estructura de datos. Los slices son utiles para permitir accesso seguro y eficiente a una procion de un arreglo sin involucrar el copiado. Por ejemplo, podrias querer hacer referencia a una sola linea de una archivo que ha sido previamente leido en memoria. Por naturaleza, un slice no es creado directamente, estos son creados para una variable que ya existe. Los slices poseen una logitud, pueden ser mutables o inmutables, en muchas formas se comportan como los arreglos:

```rust
let a = [0, 1, 2, 3, 4];
let middle = &a[1..4]; // Un slice de a: solo los elementos 1, 2, y 3
let complete = &a[..]; // Un slice conteniendo todos los elementos de a.
```

Los slices poseen el tipo `&[T]`. Hablaremos acerca de `T` cuando cubramos [los genericos][generics].

[generics]: generics.html

Puedes encontrar mas documentacion para los slices [en la documentacion de la biblioteca estandar][slice] (ingles)

[slice]: ../std/primitive.slice.html

# `str`

El `str` de Rust es el tipo de cadena de caracteres mas primitivo. Como un [tipo sin tamano][dst], no es muy util en si mismo, pero se vuelve muy util cuando es puesto detras de ua referencia, como [`&str`][strings]. Es por ello que lo dejaremos hasta aqui.

[dst]: unsized-types.html
[strings]: strings.html

Puedes encontrar mas documentacion para str [en la documentacion de la biblioteca estandar][str] (ingles)
[str]: ../std/primitive.str.html

# Tuplas

Una tupla es una lista ordenada de tamano fijo. Como esto:

```rust
let x = (1, "hola");
```

Los parentesis y comas forman esta tupla de longitud dos. He aqui el mismo codigo pero con anotaciones de tipos:

```rust
let x: (i32, &str) = (1, "hola");
```

Como puedes ver, el tipo de una tuple es justo como la tupla, pero con cada posicion teniendo el tipo en lugar del valor. Los lectores cuidadosos tambien notararan que las tuplas son heterogeneas: tenemos un `i32` y a `&str` en esta tupla. En los lenguajes de programacion de sistemas, las cadenas de caracteres son un poco mas complejas que en otros lenguajes. Por ahora solo lee `&str` como un *slice de cadena de caracteres*.  Aprenderemos mas dentro de poco.

Puedes asignar una tipo a otra, si estas tienen los mismo tipos contenidos y [aridad]. La tuples poseen la misma aridad cuando estas poseen el mismo tamano.

[aridad]: glossary.html#arity

```rust
let mut x = (1, 2); // x: (i32, i32)
let y = (2, 3); // y: (i32, i32)

x = y;
```

Puedes acceder a los campos de una tupla a traves de un *let con destructuracion*. He aqui un ejemplo:

```rust
let (x, y, z) = (1, 2, 3);

println!("x es {}", x);
```

Recuerdas [anteriormente][let] cuando dije que el lado izquierdo de una sentencia `let` era mas poderoso que la simple asignacion de un binding? Henos aqui. Podemos poner un patron en el lado izquierdo de un `let`, y si este concuerda con el lado derecho, podemos asignar multiples bindings a variable de una sola vez. En este caso, el `let` “destructura” o “parte” la tupla, ya asigna las partes a los tres bindings a variable.

[let]: variable-bindings.html

Este patron es muy poderoso, y lo veremos repetido con frecuencia en el futuro.

Puedes eliminar la ambiguedad entre una tupla de un solo elemento y un valor encerrado en parentesis usando una coma:

```rust
(0,); // tuple de un solo elementos
(0); // cero encerrado en parentesis
```

## Indexado en tuplas

Puedes tambien acceder los campos de una tupla con la sintaxis de indexado:

```rust
let tupla = (1, 2, 3);

let x = tupla.0;
let y = tupla.1;
let z = tupla.2;

println!("x es {}", x);
```

Al igual que el indexado en arreglos, este comienza en cero, pero a diferencia de el indexado en arreglos, se usan `.`, en lugar de `[]`s.

Puedes encontrar mas documentacion para tuplas [en la documentacion de la biblioteca estandar][tuple] (ingles)
[str]: ../std/primitive.str.html

[tuple]: ../std/primitive.tuple.html

# Funciones

Las funciones teambien tienen un tipo! Estas lucen asi:

```rust
fn foo(x: i32) -> i32 { x }

let x: fn(i32) -> i32 = foo;
```

En este caso, `x` es un ‘apuntador’ a una funcion que recibe un `i32` y retorna un `i32`.
