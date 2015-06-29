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

`find` recibe un closure, y trabaja en una referencia a cada elemento de un iterador. Este closure retorna `true` si el elemento es el que estamos buscando y `false` de lo contrario. Debido a que podriamos no encontrar un elemento que satisfaga nuestro criterio, `find` retorna un `Option` en lugar de un elemento.

Otro consumidor importante es `fold`. Luce de esta manera:


```rust
let suma = (1..4).fold(0, |suma, x| suma + x);
```

`fold(base, |accumulator, element| ...)`. Toma dos argumentos: el primero es un elemento llamado *base*. El segundo es un closure que a su vez toma dos argumentos: el primero es llamado el *acumulador*, y el segundo es un *elemento*. En cada iteracion, el closure es llamado, y el resultado es usado como el valor del acumulador en la siguiente iteracion. En la primera iteracion, la base es el valor del acumulador.

Bien, eso  es un poco confuso. Examinemos los valores de todas esas cosas en este iterador:


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

Entonces, `0` es nuestra base, `suma` es nuestro acumulador, y `x` es nuestro elemento. En la primera iteracion, asignamos `sum` a `0` y `x` es el primer elemento de nuestro rango, `1`. Despues sumamos `sum` y `x` lo que nos da `0 + 1 = 1`. En la segunda iteracion, ese valor se convierte en el valor de nuestro acumulador, `sum`, y el elemento es el segundo elemento del rango, `2`. `1 + 2 = 3` y de igual manera se convierte en el valor del acumulador para la ultima iteracion. En esa iteracion, `x` es el ultimmo elemento, `3`, y `3 + 3 = 6`, el cual es nurstro resultado final para nuestro `suma`. `1 + 2 + 3 = 6`, y ese es el rsultado que obtenemos.

When. `fold` puede ser un poco extrano las primeras veces que lo ves, pero una vez hace click, puedes usarlo en todos lados. Cada vez que tengas una lista de cosas, y necesites un unico reultado, `fold` es apropiado.

Los consumidores son importantes debido a una propiedad adicional de los iteradores de la que no hemos hablado todavia: pereza (laziness). Hablemos mas acerca de los iteradores y veras porque los consumidores son importantes.

## Iteradores

Como hemos dicho antes, un iterador es algo en lo que podemos llamar el metodo `.next()` repetidamente, y este nos devuelve una secuencia de elementos. Debido a que necesitamos llamar a el metodo, esto se traduce en que los iteradores pueden ser *perezosos* y no generar todos los valores por adelantado. Este codigo, por ejemplo, no genera los numeros `1-99`. En su lugar crea un valor que representa la secuencia:

```rust
let nums = 1..100;
```

Debido a que no hicimos nada con el rango, este no genero la secuencia. Agreguemos un consumidor:


```rust
let nums = (1..100).collect::<Vec<i32>>();
```

Ahora `collect()` requerira que el rango provea algunos numeros, y en consecuencia este tendra que llevar a cabo la labor de generar la secuencia.

Now, `collect()` will require that the range gives it some numbers, and so
it will do the work of generating the sequence.

Los rangos son una de las dos formas basicas de iteradores que veras. La otra es `iter()`. `iter()` puede transformar un vector en un iterador simple que te da un elemento a la vez:


```rust
let nums = vec![1, 2, 3];

for num in nums.iter() {
   println!("{}", num);
}
```

Estos dos iteratores basicos deberian servirte bien. Existen iteradores mas avanzados, incluyendo aquellos que son infinitos.

Eso es suficiente acerca de los iteradores, los adaptadores de iteradores son el ultimo concepto a el cual debemos hacer mencion en relacion a los iteradores. Hagamoslo!

## Adaptadores de iterador

Los *adaptadores de iteradores* toman un iterador y lo modifican de alguna manera, produciendo un nuevo iterador. El mas simple es llamado `map`:

```rust,ignore
(1..100).map(|x| x + 1);
```

`map` es llamado en otro iterador, `map` produce un iterador nuevo en el que cada referencia a un elemento posee el closure que se ha proporcido como argumento. Entonces esto nos dara los numeros `2-100`, Bueno, casi! Si compilas el ejemplo, obtendras una advertencia:

```text
warning: unused result which must be used: iterator adaptors are lazy and
         do nothing unless consumed, #[warn(unused_must_use)] on by default
(1..100).map(|x| x + 1);
 ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

La pereza ataca de nuevo! Ese closure nunca se ejecutara. Este ejemplo no imprime ningun numero:

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

Esto imprimira todos los numeros pares entre uno y cien. (Nota que debido a que `filter` no consume los elementos que estan siendo iterados, a este se le pasa una referencia a cada elemento, es por ello que el predicado usa el patron `&x` para extraer el entero en si mismo.)

```rust
(1..)
    .filter(|&x| x % 2 == 0)
    .filter(|&x| x % 3 == 0)
    .take(5)
    .collect::<Vec<i32>>();
```

Lo anterior te dara un vector conteniendo `6`, `12`, `18`, `24`, y `30`.


Esta es una pequena muestra de en lo que los iteradores, adaptadores de iteradores, y consumidores pueden ayudarte. Existen una variedad de iteradores realmente utiles, al igual, puedes escribir tus propios. Los iteradores proporcionan una manera segura y eficiente de manipular todo tipo de listas. Son un poco inusuales a primera vista, pero si juegas un poco con ellos, quedaras enganchado. Para una lista de los diferentes iteradores y consumidores echa un vistazo a la [documentacion del modulo iterator](../std/iter/index.html) (ingles).
