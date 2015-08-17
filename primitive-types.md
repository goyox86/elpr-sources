% Tipos Primitivos

Rust posee un conjunto de tipos que son considerados ‘primitivos’. Esto significa que están integrados en el lenguaje. Rust esta estructurado de tal manera que la biblioteca estándar también provee una numero de tipos útiles basados en los primitivos, pero estos son los mas primitivos.

# Booleanos

Rust posee un tipo booleano integrado, denominado `bool`. Tiene dos posibles valores  `true` y `false`:

```rust
let x = true;

let y: bool = false;
```

Un uso común de los booleanos es en [condicionales `if`][if].

[if]: if.html

Puedes encontrar mas documentación para los `bool`eanos [en la documentación de la biblioteca estándar][bool] (ingles).

[bool]: ../std/primitive.bool.html

# `char`

El tipo `char` representa un unico valor escalar Unicode. Puedes crear `char`s con comillas simples: (`'`)


```rust
let x = 'x';
let dos_corazones = '💕';
```

A diferencia de otros lenguajes, esto significa que `char` en Rust no es un solo byte, sino cuatro.

Puedes encontrar mas documentación para los `char`s [en la documentación de la biblioteca estándar][char] (ingles).

[char]: ../std/primitive.char.html

# Tipos numéricos

Rust posee una variedad de tipos numéricos en unas pocas categorías: con signo y sin signo, fijos y variables, de punto flotante y enteros.

Dichos tipos consisten de dos partes: la categoría, y el tamaño. Por ejemplo, `u16` es un tipo sin signo con un tamaño de dieciséis bits. Mas bits te permiten almacenar números mas grandes.

Si un literal de numero no posee nada que cause la inferencia de su tipo, se usan tipos por defecto:

```rust
let x = 42; // x tiene el tipo i32

let y = 1.0; // y tiene el tipo f64
```

He aquí una lista de los diferentes tipos numéricos, con enlaces a su documentación en la biblioteca estándar (ingles):

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

Veamos cada una de las diferentes categorías.

## Con signo y sin signo

Los tipos de enteros vienen en dos variedades: con signo y sin signo. Para entender la diferencia, consideremos un numero con cuatro bits de tamaño. Un numero de cuatro bits con signo te permitiría almacenar los números desde `-8` a `+7`. Los números con signo usan la “representación del complemento a dos”. Un numero de cuatros bits sin signo, debido a que no necesita guardar los valores negativos, puede almacenar valores desde `0` hasta `+15`.

Los tipos sin signo usan una `u` para su categoria, y los tipos con signo usan `i`. La `i` es de `integer` (entero). Entonces, `u8` es un numero de ocho bits sin signo, y un `i8` es un numero de ocho bits con signo.

## Tipos de tamaño fijo

Los tipos de tamaño fijo poseen un numero especifico de bits en su representación. Los tamaños validos son `8`, `16`, `32`, and `64`. Entonces `u32` es un entero sin signo de 32 bits, e `i64` es un entero con signo de 64 bits.

## Tipos de tamaño variable

Rust también provee tipos para los cuales el tamaño depende del tamaño del apuntador en la maquina subyacente. Dichos tipos poseen la categoría ‘size’, y vienen en variantes con y sin signo. Esto resulta en dos tipos `isize` y `usize`.

## Tipos de punto flotante

Rust también posee dos tipos de punto flotante: `f32` y `f64`. Estos corresponden a los números IEEE-754 de simple y doble precision.

# Arreglos

Como muchos lenguajes de programación, Rust posee tipos lista para representar una secuencia de cosas. El mas básico es el *arreglo*, una lista de elementos del mismo tipo y de tamaño fijo. Por defecto los arreglos son inmutables.

```rust
let a = [1, 2, 3]; // a: [i32; 3]
let mut m = [1, 2, 3]; // m: [i32; 3]
```

Los arreglos tienen el tipo `[T; N]`.  Hablaremos de la notación `T` [en la sección de genéricos][generics]. La `N` es una constante en tiempo de compilación, para la longitud del arreglo.

Hay un atajo para la inicialización de cada uno de los elementos del arreglo a el mismo valor. En este ejemplo, cada elemento de `a` sera inicializado a `0`:

```rust
let a = [0; 20]; // a: [i32; 20]
```

Puedes obtener el numero de elementos del arreglo `a` con `a.len()`

```rust
let a = [1, 2, 3];

println!("a tiene {} elementos", a.len());
```

Puedes acceder un elemento del arreglo en particular con la *notación de subindices*:

```rust
let nombres = ["Graydon", "Brian", "Niko"]; // names: [&str; 3]

println!("El segundo nombre es: {}", nombres[1]);
```

Los subindices comienzan en cero, como en la mayoría de los lenguajes de programación, entonces el primer nombre es `nombres[0]` y es segundo es `nombres[1]`. El ejemplo anterior imprime: `El segundo nombre es: Brian`. Si intentas usar un subíndice que no esta en el arreglo, obtendrás un error: el chequeo de los limites en el acceso al arreglo se realiza en tiempo de ejecución. El acceso errante como ese es la fuente de muchos bugs en los lenguajes de programación de sistemas.

Puedes encontrar mas documentación para los `array`s [en la documentación de la biblioteca estándar][array] (ingles).

[array]: ../std/primitive.array.html

# Slices

Un slice es una referencia a (o una “vista” dentro de) otra estructura de datos. Los slices son útiles para permitir acceso seguro y eficiente a una porción de un arreglo sin involucrar el copiado. Por ejemplo, podrías querer hacer referencia a una sola linea de una archivo que ha sido previamente leído en memoria. Por naturaleza, un slice no es creado directamente, estos son creados para una variable que ya existe. Los slices poseen una longitud, pueden ser mutables o inmutables, y en muchas formas se comportan como los arreglos:

```rust
let a = [0, 1, 2, 3, 4];
let middle = &a[1..4]; // Un slice de a: solo los elementos 1, 2, y 3
let complete = &a[..]; // Un slice conteniendo todos los elementos de a.
```

Los slices poseen el tipo `&[T]`. Hablaremos acerca de `T` cuando cubramos [los genericos][generics].

[generics]: generics.html

Puedes encontrar mas documentación para los slices [en la documentación de la biblioteca estándar][slice] (ingles)

[slice]: ../std/primitive.slice.html

# `str`

El `str` de Rust es el tipo de cadena de caracteres mas primitivo. Como un [tipo sin tamaño][dst], y no es muy util en si mismo, pero se vuelve muy util cuando es puesto detrás de una referencia, como [`&str`][strings]. Es por ello que lo dejaremos hasta aquí.

[dst]: unsized-types.html
[strings]: strings.html

Puedes encontrar mas documentación para str [en la documentación de la biblioteca estándar][str] (ingles)
[str]: ../std/primitive.str.html

# Tuplas

Una tupla es una lista ordenada de tamaño fijo. Como esto:

```rust
let x = (1, "hola");
```

Los paréntesis y comas forman esta tupla de longitud dos. He aquí el mismo código pero con anotaciones de tipo:

```rust
let x: (i32, &str) = (1, "hola");
```

Como puedes ver, el tipo de una tupla es justo como la tupla, pero con cada posición teniendo el tipo en lugar del valor. Los lectores cuidadosos también notaran que las tuplas son heterogéneas: tenemos un `i32` y a `&str` en esta tupla. En los lenguajes de programación de sistemas, las cadenas de caracteres son un poco mas complejas que en otros lenguajes. Por ahora solo lee `&str` como un *slice de cadena de caracteres*. Aprenderemos mas dentro de poco.

Puedes asignar una tupla a otra, si estas tienen los mismo tipos contenidos y la misma [aridad]. La tuples poseen la misma aridad cuando estas tienen el mismo tamaño.

[aridad]: glossary.html#arity

```rust
let mut x = (1, 2); // x: (i32, i32)
let y = (2, 3); // y: (i32, i32)

x = y;
```

Puedes acceder a los campos de una tupla a través de un *let con destructuracion*. He aquí un ejemplo:

```rust
let (x, y, z) = (1, 2, 3);

println!("x es {}", x);
```

Recuerdas [anteriormente][let] cuando dije que el lado izquierdo de una sentencia `let` era mas poderoso que la simple asignación de un binding? Henos aquí. Podemos colocar un patron en el lado izquierdo de un `let`, y si este concuerda con el lado derecho, podemos asignar multiples bindings a variable de una sola vez. En este caso, el `let` “destructura” o “parte” la tupla, y asigna las partes a los tres bindings a variable.

[let]: variable-bindings.html

Este patron es muy poderoso, y lo veremos repetido con frecuencia en el futuro.

Puedes eliminar la ambigüedad entre una tupla de un solo elemento y un valor encerrado en paréntesis usando una coma:

```rust
(0,); // tupla de un solo elemento
(0); // cero encerrado en parentesis
```

## Indexado en tuplas

Puedes también acceder los campos de una tupla con la sintaxis de indexado:

```rust
let tupla = (1, 2, 3);

let x = tupla.0;
let y = tupla.1;
let z = tupla.2;

println!("x es {}", x);
```

Al igual que el indexado en arreglos, este comienza en cero, pero a diferencia de el indexado en arreglos, se usan `.`, en lugar de `[]`s.

Puedes encontrar mas documentación para tuplas [en la documentación de la biblioteca estándar][tuple] (ingles)
[str]: ../std/primitive.str.html

[tuple]: ../std/primitive.tuple.html

# Funciones

Las funciones también tienen un tipo! Estas lucen así:

```rust
fn foo(x: i32) -> i32 { x }

let x: fn(i32) -> i32 = foo;
```

En este caso, `x` es un ‘apuntador’ a una función que recibe un `i32` y retorna un `i32`.
