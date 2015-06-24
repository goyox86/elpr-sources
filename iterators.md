# Iteradores

Hablemos acerca de ciclos.

Recuerdas el ciclo `for` de Rust? He aqui un ejemplo:

```rust
for x in 0..10 {
    println!("{}", x);
}
```

Ahora que sabes mas Rust, podemos hablar en detalle acerca de como festo funciona. Los rangos (el `0..10`) son iteradores. Un iterador es algo en lo que podemos llamar el metodo `.next()` repetitivamente, y este nos proporciona una secuencia de elementos.

Como esto:

```rust
let mut rango = 0..10;

loop {
    match rango.next() {
        Some(x) => {
            println!("{}", x);
        },
        None => { break }
    }
}
```

Creamos un enlace (una variable) a el rango, el cual es nuestro iterador. Luego iteramos a traves de `loop`, con un  `match` interno. Este `match` es usado en el resultado de  `rango.next()`, lo cual nos proporciona una referencia a el siguiente valor en el iterador.  `next` retorna un `Option<i32>`, en este caso, que sera  `Some(i32)` cuando tenemos un valor y  `None` cuando nos quedemos sin valores. Si obtenemos `Some(i32)`, lo imprimimos, y si obtenemos `Some(i32)`, rompemos el ciclo, saliendo de el a traves de  `break`.

Este ejemplo de codigo es basicamente el mismo que nuestra version de un ciclo for `for`. El ciclo  `for` es solo una forma practica de escribir es tra construccion `loop`/`match`/`break`.

Los ciclos  `for` no son la unica cosa que usa iteradores, sin embargo. Escribir tu propio iterador implica implementar el trait  `Iterator`. Si bien hacer eso esta fuera del ambito de esta guia, Rust provee a un numero de iteradores utiles para llevar a cabo varias tareas. Antes de hablar de eso. debemos hablar acerca de un antipatron. Dicho antipatron es usar rangos de esta forma.

Si, acabamos de hablar acerca de cuan cool son los rangos. Pero los rangos son tambein muy primitivos. Por ejemplo, si necesitas iterar a traves del contenido de un vector, podrias estar tentado a escribir esto:

```rust
let nums = vec![1, 2, 3];

for i in 0..nums.len() {
    println!("{}", nums[i]);
}
```

Esto no es estrictamente peor que usar un iterador. Puedes iterar en vectores directamente, entonces escribe esto:

```rust
let nums = vec![1, 2, 3];

for num in &nums {
    println!("{}", num);
}
```

Hay dos razones para esto. Primero, esto expresa mas directamente lo que queremos. Iteramos a traves del vector completo, en vez de iterar a traves de indices, indexando el vector luego. Segundo, esta version es mas eficiente: en la primera version tendremos chequeos de limites extra debido a que usa indexado, `nums[i]`. Pero debido a que con el iterador cedemos una referencia a cada elemento del vector a la vez, ho hay chequeo de limites en el segundo ejemplo. Esto es muy comun en iteradores: podemos ignorar chequeos de limites innecesarios, sabiendo al mismo tiempo que estamos seguros.

Hay otro detalle aqui que no esta el 100% claro debido a como `println!` trabaja. `num` es de tipo `&i32`. Esto es, es una referencia a un `i32`, no un `i32`.  `println!` maneja el dereferenciameinto por nosotros, es por ello que no lo vemos. Este codigo es correcto tambien:

```rust
let nums = vec![1, 2, 3];

for num in &nums {
    println!("{}", *num);
}
```

Ahora estamos dereferenciando a `num` explicitamente. Porque `&nums` nos da referencias? Primeramente, porque nostroes lo solicitamos de manera explicita con `&`. Segundo, si nos diera la data en si misma, tendriamos que ser su dueno, lo cual implicaria la creacion de una copiade la data y darnos esa copia. Con referencias, estamos solo haciendo un prestamo ('borrowing') de una referencia a la data, y por ello solo pasamos una referencia, sin necesidad de transferir la pertenencia.

Entonces, ahora que hemos establecido que los rangos a veces no son lo que queremos, hablemos de lo qeremos.

Hay tres amplias clases de cosas que son relevantes aqui: iteradores, *adaptadores de iteradores* y *consumidores*. He aqui algunas definiciones:

* *iterators* proporcionan una sequencia de valores.
* *adaptadores de iteradores* operan en un iterador, produciendo un nuevo iterador con una secuencia diferente de salida.
* *consumers* operan en un iterador, produciendo un conjunto final de valores.

Hablemos primeramente acerca de los consumidores, debido a que ya hemos visto un iterador, los rangos.

## Consumidores

Un *consumidor* opera en un iterador, retornando algun tipo de valor o valores. El consumidor mas comun es `collect()`. Este codigo no compila, pero muestra la intencion:

```rust,ignore
let uno_hasta_cien = (1..101).collect();
```

Como puedes ver, llamamos `collect()` en nuestro iterador. `collect()` toma tantos valores como el iterador le proporcione, retornando una colleccion de resultados. Entonces, porque esto no compilara? Rust no puede determinar que tipo de cosas quieres collectar, y por ello necesitas hacerle saber. He aqui la version que compila:

```rust
let uno_hasta_cien = (1..101).collect::<Vec<i32>>();
```

Si recuerdas, la sintaxis `::<>` te permite dar una indicio acerca del tipo, en nuestro caso estamos diciendo que queremos un vector de enteros. No siempre es necesario usar el tipo completo. `_` te permitira dar un indicio parcial acerca del tipo:

```rust
let uno_hasta_cien = (1..101).collect::<Vec<_>>();
```

Esto dice "Recolecta en un`Vec<T>`, por favor, pero infiere que es  `T` por mi.". `_` es por esta razon llamado algunas veces "marcador de posicion de tipo".

`collect()` es el consumidor mas comun, pero hay otros tambien. `find()` es uno de ellos:


```rust
let mayores_a_cuarento_y_dos = (0..100)
                               .find(|x| *x > 42);

match mayores_a_cuarento_y_dos {
    Some(_) => println!("Tenemos algunos numeros!"),
    None => println!("No se encontraron numeros :("),
}
```

`find` takes a closure, and works on a reference to each element of an
iterator. This closure returns `true` if the element is the element we're
looking for, and `false` otherwise. Because we might not find a matching
element, `find` returns an `Option` rather than the element itself.

Another important consumer is `fold`. Here's what it looks like:

```rust
let sum = (1..4).fold(0, |sum, x| sum + x);
```

`fold()` is a consumer that looks like this:
`fold(base, |accumulator, element| ...)`. It takes two arguments: the first
is an element called the *base*. The second is a closure that itself takes two
arguments: the first is called the *accumulator*, and the second is an
*element*. Upon each iteration, the closure is called, and the result is the
value of the accumulator on the next iteration. On the first iteration, the
base is the value of the accumulator.

Okay, that's a bit confusing. Let's examine the values of all of these things
in this iterator:

| base | accumulator | element | closure result |
|------|-------------|---------|----------------|
| 0    | 0           | 1       | 1              |
| 0    | 1           | 2       | 3              |
| 0    | 3           | 3       | 6              |

We called `fold()` with these arguments:

```rust
# (1..4)
.fold(0, |sum, x| sum + x);
```

So, `0` is our base, `sum` is our accumulator, and `x` is our element.  On the
first iteration, we set `sum` to `0`, and `x` is the first element of `nums`,
`1`. We then add `sum` and `x`, which gives us `0 + 1 = 1`. On the second
iteration, that value becomes our accumulator, `sum`, and the element is
the second element of the array, `2`. `1 + 2 = 3`, and so that becomes
the value of the accumulator for the last iteration. On that iteration,
`x` is the last element, `3`, and `3 + 3 = 6`, which is our final
result for our sum. `1 + 2 + 3 = 6`, and that's the result we got.

Whew. `fold` can be a bit strange the first few times you see it, but once it
clicks, you can use it all over the place. Any time you have a list of things,
and you want a single result, `fold` is appropriate.

Consumers are important due to one additional property of iterators we haven't
talked about yet: laziness. Let's talk some more about iterators, and you'll
see why consumers matter.

## Iterators

As we've said before, an iterator is something that we can call the
`.next()` method on repeatedly, and it gives us a sequence of things.
Because you need to call the method, this means that iterators
can be *lazy* and not generate all of the values upfront. This code,
for example, does not actually generate the numbers `1-99`, instead
creating a value that merely represents the sequence:

```rust
let nums = 1..100;
```

Since we didn't do anything with the range, it didn't generate the sequence.
Let's add the consumer:

```rust
let nums = (1..100).collect::<Vec<i32>>();
```

Now, `collect()` will require that the range gives it some numbers, and so
it will do the work of generating the sequence.

Ranges are one of two basic iterators that you'll see. The other is `iter()`.
`iter()` can turn a vector into a simple iterator that gives you each element
in turn:

```rust
let nums = vec![1, 2, 3];

for num in nums.iter() {
   println!("{}", num);
}
```

These two basic iterators should serve you well. There are some more
advanced iterators, including ones that are infinite.

That's enough about iterators. Iterator adapters are the last concept
we need to talk about with regards to iterators. Let's get to it!

## Iterator adapters

*Iterator adapters* take an iterator and modify it somehow, producing
a new iterator. The simplest one is called `map`:

```rust,ignore
(1..100).map(|x| x + 1);
```

`map` is called upon another iterator, and produces a new iterator where each
element reference has the closure it's been given as an argument called on it.
So this would give us the numbers from `2-100`. Well, almost! If you
compile the example, you'll get a warning:

```text
warning: unused result which must be used: iterator adaptors are lazy and
         do nothing unless consumed, #[warn(unused_must_use)] on by default
(1..100).map(|x| x + 1);
 ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

Laziness strikes again! That closure will never execute. This example
doesn't print any numbers:

```rust,ignore
(1..100).map(|x| println!("{}", x));
```

If you are trying to execute a closure on an iterator for its side effects,
just use `for` instead.

There are tons of interesting iterator adapters. `take(n)` will return an
iterator over the next `n` elements of the original iterator. Note that this
has no side effect on the original iterator. Let's try it out with our infinite
iterator from before:

```rust
for i in (1..).take(5) {
    println!("{}", i);
}
```

This will print

```text
1
2
3
4
5
```

`filter()` is an adapter that takes a closure as an argument. This closure
returns `true` or `false`. The new iterator `filter()` produces
only the elements that that closure returns `true` for:

```rust
for i in (1..100).filter(|&x| x % 2 == 0) {
    println!("{}", i);
}
```

This will print all of the even numbers between one and a hundred.
(Note that because `filter` doesn't consume the elements that are
being iterated over, it is passed a reference to each element, and
thus the filter predicate uses the `&x` pattern to extract the integer
itself.)

You can chain all three things together: start with an iterator, adapt it
a few times, and then consume the result. Check it out:

```rust
(1..)
    .filter(|&x| x % 2 == 0)
    .filter(|&x| x % 3 == 0)
    .take(5)
    .collect::<Vec<i32>>();
```

This will give you a vector containing `6`, `12`, `18`, `24`, and `30`.

This is just a small taste of what iterators, iterator adapters, and consumers
can help you with. There are a number of really useful iterators, and you can
write your own as well. Iterators provide a safe, efficient way to manipulate
all kinds of lists. They're a little unusual at first, but if you play with
them, you'll get hooked. For a full list of the different iterators and
consumers, check out the [iterator module documentation](../std/iter/index.html).
