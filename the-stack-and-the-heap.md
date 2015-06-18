# La Pila y el Montículo

Como un lenguaje de sistemas, Rust opera a un bajo nivel. Si provienes de un lenguaje de alto nivel, hay algunos aspectos de los lenguajes de programación de sistemas con los que puedas no estar familiarizado. El mas importante es el funcionamiento de la memoria, con una pila y el montículo. Si eres familiar a como lenguajes como C usan asignación desde la pila, este capitulo sera un repaso. Si no lo estas, aprenderás acerca de este concepto general, pero con un enfoque Rustero.

# Manejo de memoria

Estos dos términos hacen referencia a el manejo de la memoria. La pila y el montículo son abstracciones que ayudan a determinan cuando asignar y liberar memoria.

He aquí una comparación de alto nivel:

La pila es muy rápida, y es de donde la memoria es asignada por defecto en Rust. Pero la asignación es local a una llamada a función, y es limitada en tamaño. El montículo por otro lado, es mas lento, y es asignado por tu programa. Pero es efectivamente de un tamaño ilimitado, y es globalmente accesible.

# La Pila

Hablemos acerca de este programa Rust:

```rust
fn main() {
    let x = 42;
}
```

Este programa posee una variable (variable binding), `x`. Esta memoria tiene que ser asignada desde algún sitio. Rust asigna desde la pila por defecto, lo que significa que los valores básicos ‘van a la pila’. Pero, que significa esto?

Veamos, cuando una función es llamada, algo de memoria es asignada para sus variables locales y otra información extra. Dicha memoria es llamada registro de activación ‘stack frame’, y para el propósito de este tutorial, ignoraremos la información extra y solo consideraremos las variables locales a las cuales estamos asignando memoria. Así que en este caso, cuando `main()` es ejecutada, asignamos un entero de 32 bits para nuestro registro de activación. Todo esto es manejado automáticamente, como puedes has podido ver, no tuvimos que escribir código Rust especial o alguna otra cosa mas.

Cuando la función termina, su registro de activación es liberado o desasignado. Esto ocurre de manera automática, no tuvimos que hacer nada especial acá.

Eso es todo para este simple programa. Lo clave a entender aquí es que la asignación de memoria desde la pila es muy, muy rápida. Debido a que conocemos por adelantado todas las variables locales, podemos obtener toda la memoria de una sola vez. Y debido a que la desecharemos toda completa, podemos deshacernos de ella muy rápido, también.

La desventaja es que no podemos mantener valores rondando por allí si los necesitamos por un periodo mas largo que el tiempo de vida de una función. Tampoco hemos hablado acerca de que significa ese nombre, ‘pila’. Para hacerlo necesitamos un ejemplo ligeramente mas complejo:

```rust
fn foo() {
    let y = 5;
    let z = 100;
}

fn main() {
    let x = 42;

    foo();
}
```

Este programa tiene tres variables en total: dos en `foo()`, una en `main()`. Al igual que antes, cuando `main()` es llamada, un solo entero es asignado para su registro de activación. Pero antes que demostremos que es lo que pasa cuando `foo()` es llamada, necesitamos visualizar que es lo que esta pasando en memoria. Tu sistema operativo presenta a tu programa una visión muy simple: una lista inmensa de direcciones, desde 0 hasta un numero muy grande, que representa cuanta memoria RAM posee la maquina. Por ejemplo si tienes un gigabyte de RAM, tus direcciones irán desde 0 hasta `1,073,741,824` ese numero viene de 2<sup>30</sup>, el numero de bytes en un gigabyte.

Esta memoria es una especie de arreglo gigante: las direcciones comienzan en cero y se incrementan hasta el numero final. Entonces, he aquí un diagrama de nuestro primer stack frame:


| Dirección | Nombre | Valor |
|-----------|--------|-------|
| 0         | x      | 42    |

Hemos colocado a `x` en la dirección `0`, con el valor `42`

Cuando `foo()` es llamada un nuevo registro de activación es asignado:


| Dirección | Nombre | Valor |
|-----------|--------|-------|
| 2         | z      | 100   |
| 1         | y      | 5     |
| 0         | x      | 42    |

Debido a que `0` fue reservado para la primera frame, `1` y `2` son usados para el registro de activación de `foo()`. La pila crece hacia arriba, a medida que llamamos a mas funciones.

Hay algunas cosas importantes que debemos notar aquí. Los números 0, 1 y 2 existen solo para propósitos ilustrativos, y no poseen relación con los números que una computadora realmente usaría. En particular, la serie de direcciones estan separadas por algún numero de bytes, y esa separación puede incluso exceder el tamaño del valor que esta siendo almacenado.

Después que `foo()` termina, su registro de activación es liberado:

| Dirección | Nombre | Valor |
|-----------|--------|-------|
| 0         | x      | 42    |

Y luego después de que `main()` finaliza, este ultimo valor se va. Fácil!

Es llamada una pila (‘stack’) debido a que funciona como una pila de platos: el primer plato que colocas es el ultimo plato que sacaras. Las pilas son algunas veces llamadas ‘colas ultimo que entra, primero que sale’ (‘last in, first out queues’), por estas razones el ultimo valor que pusiste en la pila será el primero que obtendrás de ella.

Probemos un ejemplo de tres niveles:

```rust
fn bar() {
    let i = 6;
}

fn foo() {
    let a = 5;
    let b = 100;
    let c = 1;

    bar();
}

fn main() {
    let x = 42;

    foo();
}
```

Bien, en primera instancia, llamamos a `main()`:

| Dirección | Nombre | Valor |
|-----------|------|---------|
| 0         | x    | 42      |

Acto seguido, `main()` llama a `foo()`:

| Dirección | Nombre | Valor |
|-----------|--------|-------|
| 3         | c      | 1     |
| 2         | b      | 100   |
| 1         | a      | 5     |
| 0         | x      | 42    |

Luego `foo()` llama a `bar()`:

| Dirección | Nombre | Valor |
|-----------|--------|-------|
| 4         | i      | 6     |
| 3         | c      | 1     |
| 2         | b      | 100   |
| 1         | a      | 5     |
| 0         | x      | 42    |

Uff! Nuestra pila esta creciendo.

Después que `bar()` termina, su registro de activación es liberado, dejando solo a `foo()` y `main()`:


| Dirección | Nombre | Valor |
|-----------|--------|-------|
| 3         | c      | 1     |
| 2         | b      | 100   |
| 1         | a      | 5     |
| 0         | x      | 42    |

Después `foo()` termina, dejando solo a `main()`

| Dirección | Nombre | Valor |
|-----------|--------|-------|
| 0         | x      | 42    |

Hemos terminado entonces. Comenzamos a entender? Es como apilar platos: agregas al tope y sacas de el.

# El Montículo

Ahora, todo esto trabaja bien, pero no todo puede funcionar como esto. Algunas veces, necesitas pasar memoria entre diferentes funciones, o mantener memoria viva por un tiempo mayor que la ejecución de una función. Para esto usamos el montículo.

En Rust, puedes asignar memoria desde el montículo con el [tipo `Box<T>`][box] (caja).

He aqui un ejemplo:

```rust
fn main() {
    let x = Box::new(5);
    let y = 42;
}
```

[box]: ../std/boxed/index.html

Acá, lo que sucede cuando `main()` es llamada:

| Dirección | Nombre | Valor |
|-----------|--------|-------|
| 1         | y      | 42    |
| 0         | x      | ??????|

Asignamos espacio para dos variables en la pila. `y` es `42`, como siempre ha sido, pero que acerca de `x`? Bueno, `x` es un `Box<i32>`, y las cajas (boxes) asignan memoria desde el montículo. El valor de la caja en cuestión es una estructura que posee un apuntador a ‘el montículo’. Cuando comienza la ejecución de la función, y `Box::new()` es llamada, esta asigna algo de memoria para el montículo y coloca `5` allí. La memoria ahora luce así:

| Dirección       | Nombre | Valor          |
|-----------------|--------|----------------|
| 2<sup>30</sup>  |        | 5              |
| ...             | ...    | ...            |
| 1               | y      | 42             |
| 0               | x      | 2<sup>30</sup> |

Tenemos 2<sup>30</sup> en nuestra computadora hipotética con 1GB de RAM. Y debido a que nuestra pila crece desde cero, la forma mas fácil para asignar memoria es desde el otro extremo. Entonces nuestro primer valor esta en el lugar mas alto en la memoria. Y el valor de la estructura en `x` tiene un  [apuntador plano][rawpointer] (raw pointer) a el lugar que hemos asignado en el montículo, entonces el valor de `x` es 2<sup>30</sup>, la dirección de la memoria que hemos solicitado.

[rawpointer]: raw-pointers.html

No hemos hablado mucho acerca de que significa en realidad asignar y liberar memoria en estos contextos. Entrar en el profundo detalle de ello esta fuera del alcance de este tutorial, lo importante a resaltar es que el montículo no es una simple pila que crece desde el lado opuesto. Tendremos un ejemplo de esto mas adelante en el libro, pero debido a que el montículo puede ser asignado y liberado en cualquier orden, puede terminar con ‘vacios’. He aquí un diagrama de la distribución de la memoria de un programa que ha estado corriendo por alguna tiempo:

| Dirección            | Nombre | Valor               |
|----------------------|--------|---------------------|
| 2<sup>30</sup>       |        | 5                   |
| (2<sup>30</sup>) - 1 |        |                     |
| (2<sup>30</sup>) - 2 |        |                     |
| (2<sup>30</sup>) - 3 |        | 42                  |
| ...                  | ...    | ...                 |
| 3                    | y      | (2<sup>30</sup>) - 3|
| 2                    | y      | 42                  |
| 1                    | y      | 42                  |
| 0                    | x      | 2<sup>30</sup>      |

En este caso, hemos asignado cuatro cosas en el montículo, pero hemos liberado dos de ellas. Hay un vacío entre 2<sup>30</sup> y (2<sup>30</sup>) - 3 que no esta siendo usado actualmente. El detalle especifico acerca de como y porque esto sucede depende de la estrategia usada para manejar el montículo. Diferentes programas pueden usar diferentes ‘asignadores de memoria’ (‘memory allocators’), que son bibliotecas encargadas de manejar la asignación de memoria por ti. Los programas en Rust usan [jemalloc][jemalloc] para dicho propósito.

[jemalloc]: http://www.canonware.com/jemalloc/

De cualquier modo, de vuelta a nuestro ejemplo. Debido a que esta memoria esta en el montículo, puede permanecer viva mas tiempo que la función que crea la caja (box). En este caso, sin embargo, esto no sucede. [^moving] cuando la función termina, necesitamos liberar el registro de activación de `main()`. `Box<T>`, sin embargo, tienen un truco bajo la manga: [Drop][drop]. La implementación de `Drop` para `Box` libera la memoria que ha sido asignada cuando la caja es creada. Grandioso! Así que cuando `x` se va (sale de contexto), primero libera la memoria asignada desde el montículo:

| Dirección | Nombre | Valor |
|-----------|--------|-------|
| 1         | y      | 42    |
| 0         | x      | ??????|

[drop]: drop.html

[^moving]: Podemos hacer que la memoria permanezca viva mas tiempo transfiriendo la pertenencia (ownership), algunas veces llamado ‘moviendo fuera de la caja’ (‘moving out of the box’). Ejemplos mas complejos serán cubiertos mas adelante.

Luego el registro de activación se va, liberando toda nuestra memoria.

# Argumentos y prestamo (borrowing)

Hemos llevado a cabo algunos ejemplos básicos con la pila y el montículo, pero que hay acerca de los argumentos a funciones y el préstamo (borrowing). He aquí un pequeño programa Rust:

```rust
fn foo(i: &i32) {
    let z = 42;
}

fn main() {
    let x = 5;
    let y = &x;

    foo(y);
}
```

Cuando entramos a `main()`, la memoria luce de la siguiente manera:

| Dirección | Nombre | Valor |
|-----------|--------|-------|q
| 1         | y      | 0     |
| 0         | x      | 5     |

`x` es un simple `5`, y `y` es una referencia a `x`. Entonces, el valor de `y` es la dirección de memoria en la que `x` vive, que en este caso es `0`.

Que sucede cuando llamamos a `foo()` pasando a `y` como argumento?

| Dirección | Nombre | Valor |
|-----------|--------|-------|
| 3         | z      | 42    |
| 2         | i      | 0     |
| 1         | y      | 0     |
| 0         | x      | 5     |

Los registros de activación no son solo para variables locales, son también para argumentos. En este caso, necesitamos tener ambos `i`, nuestro argumento, y `z` nuestra variable local. `i` es una copia del argumento, `y`. Debido a que el valor de `y` es `0` entonces ese es el valor de `i`.

Esta es una razón por la cual tomar prestada una variable no libera ninguna memoria: el valor de la referencia es solo un apuntador a una dirección de memoria. Si nos deshiciéramos de la memoria subyacente, las cosas no funcionarían del todo bien.

# Un ejemplo complejo

Bien, vayamos a través de este programa complejo paso-a-paso:

```rust
fn foo(x: &i32) {
    let y = 10;
    let z = &y;

    baz(z);
    bar(x, z);
}

fn bar(a: &i32, b: &i32) {
    let c = 5;
    let d = Box::new(5);
    let e = &d;

    baz(e);
}

fn baz(f: &i32) {
    let g = 100;
}

fn main() {
    let h = 3;
    let i = Box::new(20);
    let j = &h;

    foo(j);
}
```

Primero, llamamos a `main()`:

| Dirección       | Nombre | Valor         |
|-----------------|--------|---------------|
| 2<sup>30</sup>  |        | 20            |
| ...             | ...    | ...           |
| 2               | j      | 0             |
| 1               | i      | 2<sup>30</sup>|
| 0               | h      | 3             |

Asignamos memoria para `j`, `i`, y `h`. `i` esta en el montículo, es por ello que su valor apunta hacia el.

A continuation, al final de `main()`, `foo()` es llamada:

| Dirección       | Nombre | Valor          |
|-----------------|--------|----------------|
| 2<sup>30</sup>  |        | 20             |
| ...             | ...    | ...            |
| 5               | z      | 4              |
| 4               | y      | 10             |
| 3               | x      | 0              |
| 2               | j      | 0              |
| 1               | i      | 2<sup>30</sup> |
| 0               | h      | 3              |

Es asignado espacio para `x`, `y`, y `z`. El argumento `x` tiene el mismo valor que `j`, debido a que eso fue lo que le proporcionamos a la función. Es un apuntador a la dirección `0`, puesto que `j` apunta a `h`.

Seguidamente, `foo()` llama a `baz()`, pasándole `z`:

| Dirección       | Nombre | Valor          |
|-----------------|--------|----------------|
| 2<sup>30</sup>  |        | 20             |
| ...             | ...    | ...            |
| 7               | g      | 100            |
| 6               | f      | 4              |
| 5               | z      | 4              |
| 4               | y      | 10             |
| 3               | x      | 0              |
| 2               | j      | 0              |
| 1               | i      | 2<sup>30</sup> |
| 0               | h      | 3              |

Hemos asignado memoria para `f` y `g`. `baz()` es muy corta, así que cuando termina, nos deshacemos de su registro de activación:

| Dirección       | Nombre | Valor          |
|-----------------|--------|----------------|
| 2<sup>30</sup>  |        | 20             |
| ...             | ...    | ...            |
| 5               | z      | 4              |
| 4               | y      | 10             |
| 3               | x      | 0              |
| 2               | j      | 0              |
| 1               | i      | 2<sup>30</sup> |
| 0               | h      | 3              |

Después, `foo()` llama a `bar()` con `x` y `z`:

| Dirección            | Nombre | Valor                |
|----------------------|--------|----------------------|
|  2<sup>30</sup>      |        | 20                   |
| (2<sup>30</sup>) - 1 |        | 5                    |
| ...                  | ...    | ...                  |
| 10                   | e      | 9                    |
| 9                    | d      | (2<sup>30</sup>) - 1 |
| 8                    | c      | 5                    |
| 7                    | b      | 4                    |
| 6                    | a      | 0                    |
| 5                    | z      | 4                    |
| 4                    | y      | 10                   |
| 3                    | x      | 0                    |
| 2                    | j      | 0                    |
| 1                    | i      | 2<sup>30</sup>       |
| 0                    | h      | 3                    |

Terminamos asignando otro valor en el montículo, así que tenemos que restar uno a 2<sup>30</sup>. Es mas fácil escribir eso que `1,073,741,823`. En cualquier caso, seteamos las variables como es usual.

Al final de `bar()`, esta llama a `baz()`:

| Dirección            | Nombre | Valor                |
|----------------------|--------|----------------------|
|  2<sup>30</sup>      |        | 20                   |
| (2<sup>30</sup>) - 1 |        | 5                    |
| ...                  | ...    | ...                  |
| 12                   | g      | 100                  |
| 11                   | f      | 9                    |
| 10                   | e      | 9                    |
| 9                    | d      | (2<sup>30</sup>) - 1 |
| 8                    | c      | 5                    |
| 7                    | b      | 4                    |
| 6                    | a      | 0                    |
| 5                    | z      | 4                    |
| 4                    | y      | 10                   |
| 3                    | x      | 0                    |
| 2                    | j      | 0                    |
| 1                    | i      | 2<sup>30</sup>       |
| 0                    | h      | 3                    |

Con esto, estamos en nuestro punto mas profundo! Wow! Felicitaciones por haber seguido todo esto y haber llegado tan lejos.

Luego `baz()`termina, nos deshacemos de `f` y `g`:

| Dirección            | Nombre | Valor                |
|----------------------|--------|----------------------|
|  2<sup>30</sup>      |        | 20                   |
| (2<sup>30</sup>) - 1 |        | 5                    |
| ...                  | ...    | ...                  |
| 10                   | e      | 9                    |
| 9                    | d      | (2<sup>30</sup>) - 1 |
| 8                    | c      | 5                    |
| 7                    | b      | 4                    |
| 6                    | a      | 0                    |
| 5                    | z      | 4                    |
| 4                    | y      | 10                   |
| 3                    | x      | 0                    |
| 2                    | j      | 0                    |
| 1                    | i      | 2<sup>30</sup>       |
| 0                    | h      | 3                    |

A continuación, retornamos de `bar()`. `d` en este caso es un `Box<T>`, entonces también libera a lo que apunta: (2<sup>30</sup>) - 1.

| Dirección            | Nombre | Valor          |
|----------------------|--------|----------------|
|  2<sup>30</sup>      |        | 20             |
| ...                  | ...    | ...            |
| 5                    | z      | 4              |
| 4                    | y      | 10             |
| 3                    | x      | 0              |
| 2                    | j      | 0              |
| 1                    | i      | 2<sup>30</sup> |
| 0                    | h      | 3              |

Después de esto, `foo()` retorna:

| Dirección       | Nombre | Valor          |
|-----------------|--------|----------------|
|  2<sup>30</sup> |        | 20             |
| ...             | ...    | ...            |
| 2               | j      | 0              |
| 1               | i      | 2<sup>30</sup> |
| 0               | h      | 3              |

Entonces, finalmente `main()` retorna, lo cual limpia el resto. Cuando `i` es liberada (a través de `Drop`) esta limpiara también lo ultimo en el montículo.

# Que hacen otros lenguajes?

La mayoría de los lenguajes con un recolector de basura asignan por defecto desde el montículo por. Esto significa que todos los valores están dentro de cajas (boxed). Existen un numero de razones por la cuales esto es hecho de esta manera, pero estan fuera del alcance de este tutorial. También, existen algunas optimizaciones que hacen que esto no sea 100% verdad todo el tiempo. En vez de confiar en la pila y `Drop` para limpiar la memoria, el recolector de basura es el encargado de administrar el montículo.

# Cual usar?

Si la pila es mas rápida y mas fácil de usar, porque necesitamos el montículo? Una gran razon es que la asignación desde la pila significa que solo tienes semántica LIFO para reclamar almacenamiento. La asignación desde el montículo es estrictamente mas general, permitiendo que el almacenamiento pueda ser tomado y retornado a el pool en orden arbitrario, pero con un costo en complejidad.

Generalmente, deberías preferir asignación desde la pila, es por ello que Rust asigna desde la pila por defecto. El modelo LIFO de la pila es mas simple, a nivel fundamental. Esto tiene dos grandes impactos: Eficiencia en tiempo de ejecución e impacto semántico.

## Runtime Efficiency.

Administrar la memoria para la pila es trivial: La maquina simplemente incrementa un solo valor, el llamado "apuntador a la pila" (“stack pointer”). La administración de memoria para el montículo no lo es: La memoria asignada desde el montículo es liberada en puntos arbitrarios, y cada bloque de memoria asignada desde el montículo pude ser de un tamaño arbitrario, el administrador de memoria generalmente debe trabajar mucho mas duro para identificar memoria que pueda ser recusada.

Si quisieras sumergirte mas en este topico en mayor detalle, [este paper][wilson] es una muy buena introducción.

[wilson]: http://www.cs.northwestern.edu/~pdinda/icsclass/doc/dsa.pdf

## Impacto semantico

La asignación desde la pila impacta a Rust como lenguaje, y con ello el modelo mental del desarrollador. La semántica LIFO es lo que conduce como el lenguaje Rust maneja el manejo automático de memoria. Incluso la liberación de una caja asignada desde el montículo con un único dueño puede ser manejada por la semántica LIFO, tal y como se ha discutido en este capitulo. La flexibilidad (e.j. expresividad) de la semántica no LIFO significa que en general el compilador no puede de manera automática y en tiempo de compilación donde la memoria debería ser liberada; tiene que apoyarse en protocolos dinamicos, potencialmente externos a el lenguaje, para efectuar liberación de memoria (conteo de referencias, como el usado en `Rc<T>` y `Arc<T>`, es un ejemplo de ello).

Cuando llevado al extremo, el mayor poder expresivo de la asignación desde el montículo viene a costo de bien sea soporte significativo en tiempo de ejecución  (e.j. en forma de un recolector de basura) o esfuerzo significativo por parte del programador (en la forma de llamadas manuales explícitas que requieren verificación no proporcionada por el compilador de Rust).
