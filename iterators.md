% Iteradores

Hablemos de ciclos.

Recuerdas el ciclo `for` de Rust? He aqui un ejemplo:

```rust
for x in 0..10 {
    println!("{}", x);
}
```

Ahora que sabes mas Rust, podemos hablar en detalle acerca de como este código funciona. Los rangos (el `0..10`) son iteradores. Un iterador es algo en lo que podemos llamar el método `.next()` repetitivamente, y el iterador nos proporciona una secuencia de elementos.

Por ejemplo:

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

Creamos un enlace (una variable) a el rango, nuestro iterador. Luego iteramos mediante el ciclo `loop`, con un `match` interno. Dicho `match` usa el resultado de `rango.next()`, que nos proporciona una referencia a el siguiente valor en el iterador. `next` retorna un `Option<i32>`, en este caso, que será `Some(i32)` cuando tenemos un valor y `None` cuando nos quedemos sin valores. Si obtenemos `Some(i32)`, lo imprimimos, y si obtenemos `None`, rompemos el ciclo, saliendo de el a través de `break`.

Este ejemplo de código es básicamente el mismo que nuestra version de un ciclo for `for`. El ciclo  `for` es solo una forma practica de escribir una construcción `loop`/`match`/`break`.

Sin embargo, los ciclos `for` no son la única cosa que usa iteradores. Escribir tu propio iterador implica implementar el trait `Iterator`. Si bien hacerlo esta fuera del ámbito de esta guía, Rust provee a un numero de iteradores útiles para llevar a cabo diversas tareas. Antes de hablar de eso, debemos hablar acerca de un anti-patron. Dicho anti-patron es usar rangos de la manera expuesta anteriormente.

Si, acabamos de hablar acerca de cuan cool son los rangos. Pero son también muy primitivos. Por ejemplo, si necesitamos iterar a través del contenido de un vector, podríamos estar tentados a escribir algo como esto:

```rust
let nums = vec![1, 2, 3];

for i in 0..nums.len() {
    println!("{}", nums[i]);
}
```
Esto no es estrictamente peor que usar un iterador. Se puede iterar en vectores directamente, escribe esto:

```rust
let nums = vec![1, 2, 3];

for num in &nums {
    println!("{}", num);
}
```

Hay dos razones para hacerlo de esta manera. Primero, expresa lo que queremos de manera mas directa. Iteramos a través del vector completo, en vez de iterar a través de indices para luego indexar el vector. Segundo, esta version es mas eficiente: en la primera version tendremos chequeos de limites extra debido a que usa indexado, `nums[i]`. En el segundo ejemplo y debido a que con el iterador cedemos una referencia a cada elemento a la vez, ho hay chequeo de limites. Esto es muy común en iteradores: podemos ignorar chequeos de limites innecesarios, sabiendo al mismo tiempo que estamos seguros.

Hay otro detalle aquí que no esta el 100% claro debido a el funcionamiento `println!`. `num` es de tipo `&i32`. Una referencia a un `i32`, no un `i32`.  `println!` maneja el dereferenciamiento por nosotros, es por ello que no lo vemos. Este código también es correcto:

```rust
let nums = vec![1, 2, 3];

for num in &nums {
    println!("{}", *num);
}
```

Ahora estamos dereferenciando a `num` de forma explicita. Porque `&nums` nos da referencias? Primeramente, porque lo solicitamos de manera explicita con `&`. Segundo, si nos diera la data en si misma, tendríamos que ser dueños de ella, lo cual implicaría la creación de una copia de la data para después darnos esa copia. Con referencias, solo estamos haciendo un préstamo ('borrowing') de una referencia a la data, y por ello solo pasamos una referencia, sin necesidad de transferir la pertenencia.

Entonces, ahora que hemos establecido que los rangos a veces no son lo que queremos, hablemos de lo queremos.

Hay tres amplias clases de cosas que son relevantes: iteradores, *adaptadores de iteradores* y *consumidores*. He aquí algunas definiciones:

* *iteradores* proporcionan una sequencia de valores.
* *adaptadores de iteradores* operan en un iterador, produciendo un nuevo iterador con una secuencia diferente de salida.
* *consumidores* operan en un iterador, produciendo un conjunto final de valores.

Hablemos primeramente acerca de los consumidores, debido a que ya hemos visto un iterador, los rangos.

## Consumidores

Un *consumidor* opera en un iterador, retornando algún tipo de valor o valores. El consumidor mas común es `collect()`. Este código no compila, pero muestra la intención:

```rust,ignore
let uno_hasta_cien = (1..101).collect();
```

Como puedes ver, llamamos `collect()` en el iterador. `collect()` toma tantos valores como el iterador le proporcione, retornando una colección de resultados. Entonces, porque este código no compilara? Rust no puede determinar que tipo de cosas quieres recolectar, y es por ello necesitas hacerle saber. Esta es la version que compila:

```rust
let uno_hasta_cien = (1..101).collect::<Vec<i32>>();
```

Si recuerdas, la sintaxis `::<>` te permite dar una indicio acerca del tipo, en nuestro caso estamos diciendo que queremos un vector de enteros. No siempre es necesario usar el tipo completo. `_` te permitirá dar un indicio parcial acerca del tipo:

```rust
let uno_hasta_cien = (1..101).collect::<Vec<_>>();
```

Esto dice "Recolecta en un`Vec<T>`, por favor, pero infiere que es `T` por mi.". `_` es por esta razón llamado algunas veces "marcador de posición de tipo".

`collect()` es el consumidor mas común, pero hay otros. `find()` es uno de ellos:

```rust
let mayores_a_cuarenta_y_dos = (0..100)
                               .find(|x| *x > 42);

match mayores_a_cuarenta_y_dos {
    Some(_) => println!("Tenemos algunos números!"),
    None => println!("No se encontraron números :("),
}
```

`find` recibe un closure, y trabaja en una referencia a cada elemento de un iterador. Dicho closure retorna `true` si el elemento es el que estamos buscando y `false` de lo contrario. Debido a que podríamos no encontrar un elemento que satisfaga nuestro criterio, `find` retorna un `Option` en lugar de un elemento.

Otro consumidor importante es `fold`. Luce asi:


```rust
let suma = (1..4).fold(0, |suma, x| suma + x);
```

`fold(base, |acumulador, elemento| ...)`. Toma dos argumentos: el primero es un elemento llamado *base*. El segundo es un closure que a su vez toma dos argumentos: el primero es llamado el *acumulador*, y el segundo es un *elemento*. En cada iteración, el closure es llamado, y el resultado es usado como el valor del acumulador en la siguiente iteración. En la primera iteración, la base es el valor del acumulador.

Bien, eso es un poco confuso. Examinemos los valores de todas las cosas en este iterador:


| base | acumulador | elemento | resultado del closure |
|------|------------|----------|-----------------------|
| 0    | 0          | 1        | 1                     |
| 0    | 1          | 2        | 3                     |
| 0    | 3          | 3        | 6                     |


Hemos llamado a `fold()` con estos argumentos:

```rust
# (1..4)
.fold(0, |suma, x| suma + x);
```

Entonces, `0` es nuestra base, `suma` es nuestro acumulador, y `x` es nuestro elemento. En la primera iteración, asignamos `sum` a `0` y `x` es el primer elemento de nuestro rango, `1`. Después sumamos `sum` y `x` lo que nos da `0 + 1 = 1`. En la segunda iteración, ese valor se convierte en el valor de nuestro acumulador, `sum`, y el elemento es el segundo elemento del rango, `2`. `1 + 2 = 3` y de igual manera se convierte en el valor del acumulador para la ultima iteración. En esa iteración, `x` es el ultimo elemento, `3`, y `3 + 3 = 6`, resultado final para nuestro `suma`. `1 + 2 + 3 = 6`, ese es el resultado que obtenemos.

Whew. `fold` puede ser un poco extraño a primera vista, pero una vez hace click, puedes usarlo en todos lados. Cada vez que tengas una lista de cosas, y necesites un único resultado, `fold` es apropiado.

Los consumidores son importantes debido a una propiedad adicional de los iteradores de la que no hemos hablado todavia: pereza (laziness). Hablemos mas acerca de los iteradores y veras porque los consumidores son importantes.

## Iteradores

Como hemos dicho antes, un iterador es algo en lo que podemos llamar el método `.next()` repetidamente, y este nos devuelve una secuencia de elementos. Debido a que necesitamos llamar a el método, los iteradores pueden ser *perezosos* y no generar todos los valores por adelantado. Este código, por ejemplo, no genera los números `1-99`. En su lugar crea un valor que representa la secuencia:

```rust
let nums = 1..100;
```

Debido a que no hicimos nada con el rango, este no genero la secuencia. Agreguemos un consumidor:


```rust
let nums = (1..100).collect::<Vec<i32>>();
```

Ahora `collect()` requerirá que el rango provea algunos números, y en consecuencia tendra que llevar a cabo la labor de generar la secuencia.


Los rangos son una de las dos formas básicas de iteradores que veras. La otra es `iter()`. `iter()` puede transformar un vector en un iterador simple que proporciona un elemento a la vez:

```rust
let nums = vec![1, 2, 3];

for num in nums.iter() {
   println!("{}", num);
}
```

Estos dos iteradores básicos deberian ser de utilidad. Existen iteradores mas avanzados, incluyendo aquellos que son infinitos.

Suficiente acerca de iteradores, los adaptadores de iteradores son el ultimo concepto relacionado a iteradores al que debemos hacer mención. Hagamoslo!

## Adaptadores de iterador

Los *adaptadores de iteradores* toman un iterador y lo modifican de alguna manera, produciendo uno nuevo. El mas simple es llamado `map`:

```rust,ignore
(1..100).map(|x| x + 1);
```

`map` es llamado en otro iterador, `map` produce un iterador nuevo en el que cada referencia a un elemento posee el closure que se ha proporcionado como argumento. El código anterior nos dará los números `2-100`, Bueno, casi! Si compilas el ejemplo, obtendrás una advertencia:

```text
warning: unused result which must be used: iterator adaptors are lazy and
         do nothing unless consumed, #[warn(unused_must_use)] on by default
(1..100).map(|x| x + 1);
 ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

La pereza ataca de nuevo! Ese closure nunca se ejecutara. Este ejemplo no imprime ningún numero:

```rust,ignore
(1..100).map(|x| println!("{}", x));
```

Si estas intentanto ejecutar un closure en un iterador para obtener sus efectos colaterales (side-effects) usa un `for`.

```rust
for i in (1..).take(5) {
    println!("{}", i);
}
```

Esto imprimira:

```text
1
2
3
4
5
```

`filter()` es un adaptador que toma un closure como argumento. Dicho closure retorna `true` o `false`. El nuevo iterador que `filter()` produce solo elementos para los que el closure retorna `true`:

```rust
for i in (1..100).filter(|&x| x % 2 == 0) {
    println!("{}", i);
}
```

Esto imprimirá todos los números pares entre uno y cien. (Nota que debido a que `filter` no consume los elementos que están siendo iterados, a este se le pasa una referencia a cada elemento, debido a ello, el predicado usa el patron `&x` para extraer el entero.)

```rust
(1..)
    .filter(|&x| x % 2 == 0)
    .filter(|&x| x % 3 == 0)
    .take(5)
    .collect::<Vec<i32>>();
```

Lo anterior te  un vector conteniendo `6`, `12`, `18`, `24`, y `30`.


Esta es una pequeña muestra de las cosas en las cuales los iteradores, adaptadores de iteradores, y consumidores pueden ayudarte. Existe una variedad de iteradores realmente útiles, junto al hecho que puedes escribir tus propios. Los iteradores proporcionan una manera segura y eficiente de manipular todo tipo de listas. Son un poco inusuales a primera vista, pero si juegas un poco con ellos, quedaras enganchado. Para una lista de los diferentes iteradores y consumidores echa un vistazo a la [documentacion del modulo iterator](../std/iter/index.html) (ingles).
