% Ciclos

Rust actualmente provee tres enfoques para realizar actividad iterativa. Son: `loop`, `while` y `for`. Cada uno de esto enfoques tiene su propio juego de usos.

## loop

El ciclo infinito `loop` es el cilo mas simple disponible en Rust. A traves del uso de la palabra reservada `loop`, Rust proporciona una forma de iterar indefinidamente hasta que alguna sentencia de terminacion sea alcanzada. El `loop` infinito de Rust luce asi:


```rust,ignore
loop {
    println!("Itera por siempre!");
}
```

## while

Rust tambien tiene un ciclo `while`. Luce asi:

```rust
let mut x = 5; // mut x: i32
let mut completado = false; // mut completado: bool

while !completado {
    x += x - 3;

    println!("{}", x);

    if x % 5 == 0 {
        completado = true;
    }
}
```

Los ciclos `while` son la eleccion correcta cuando no estas seguro acerca de cuentas veces necesitas iterar.

De necesitar un ciclo infinito, podrias estar tentado a escribir lo siguiente:

```rust,ignore
while true {
```

Sin embargo, `loop` es por lejos mejor en este caso:

```rust,ignore
loop {
```

El analisis de flujo de control trata esta construccion de manera diferente que a un `while true`, debido a que sabemos que iteraremos siempre. En general, mientras mas informacion le proporcionemos al compilador, este podra desempenarse mejor en relacion a seguridad y generacion de codigo, es por ello que siempre deberias preferir `loop` cuando tengas planeado iterar de manera indefinida.


## for

El ciclo `for` es usado para iterar un numero particular de veces. Los ciclos `for` de Rust, sin embergo, trabajan de manera diferente a los de otros lenguajes de programacion de sistemas. El `for` de Rust no luce como este ciclo `for` con “estilo-C”


```c
for (x = 0; x < 10; x++) {
    printf( "%d\n", x );
}
```

En su lugar, el ciclo `for` de Rust luce asi:

```rust
for x in 0..10 {
    println!("{}", x); // x: i32
}
```

En terminos ligeramente mas abstractos:

```ignore
for var in expresion {
    codigo
}
```

La expresion es un [iterador][iterator].  El iterador retorna una serie de elementos. Cada elemento es una iteracion del ciclo. Ese valor es a su vez asignado a el nombre `var`, el cual es valido en el cuerpo del ciclo. Una vez que el ciclo ha terminado, el siguiente valor es obtenido del iterador, e iteramos una vez mas. Cuando no hay mas valores, el ciclo `for` ha terminado.

[iterator]: iterators.html

En nuestro ejemplo, `0..10` es una expresion que toma una posicion de inico y una de fin, y devuelve un iterador por sobre esos vallores. El limite superior es eclusivo, entonces, nuestro loop imprimira de `0` hasta `9`, no `10`.

Rut no posee el ciclo `for` al esticlo C a proposito. Controlar manualmente cada elemento del ciclo es compricado y propenso a errores, incluso para programadores C experimentados.

### Enumerate

Cuando necesitas llevar registro de cuantas veces has iterado, pudes usar la funcion `.enumerate()`.

#### En rangos:

```rust
for (i,j) in (5..10).enumerate() {
    println!("i = {} y j = {}", i, j);
}
```

Salida:

```text
i = 0 y j = 5
i = 1 y j = 6
i = 2 y j = 7
i = 3 y j = 8
i = 4 y j = 9
```

No olvides colocar los parentesis alredor del rango.

#### En iteradores:

```rust
# let lineas = "hola\nmundo".lines();
for (numerolinea, linea) in lineas.enumerate() {
    println!("{}: {}", numerolinea, linea);
}
```

Outputs:

```text
0: Contenido de la linea uno
1: Contenido de la linea dos
2: Contenido de la linea tres
3: Contenido de la linea cuatro
```

## Finalizando la itracion de manera temprana

Echemos un vistazo a ese ciclo `while` que vimos con anterioridad:


```rust
let mut x = 5;
let mut completado = false;

while !completado {
    x += x - 3;

    println!("{}", x);

    if x % 5 == 0 {
        completado = true;
    }
}
```

Necesitabamos mantener un binding a variable `mut`, `completado`, para saber cuando debiamos salir del ciclo. Rust posee dos palabras clave para ayudarnos a modificar la iteracion: `break` y `continue`.

En este caso, podemos escribir el ciclo de una mejor forma con un `break`:

```rust
let mut x = 5;

loop {
    x += x - 3;

    println!("{}", x);

    if x % 5 == 0 { break; }
}
```

Ahora iteramos de manera indefinida con `loop` y usamos `break` para romper el ciclo de manera temprana. Usar un `return` explicito tambien servira para terminar el ciclo de manera temprana.

`continue` es similar, pero en lugar de terminar el ciclo, nos hace ir a la siguiente iteracion. Lo siguiente imprimira los numnero impares:

```rust
for x in 0..10 {
    if x % 2 == 0 { continue; }

    println!("{}", x);
}
```

## Etiquetas loop

Tambien podrias encontras situaciones en las cuales tengas ciclos anidados y necesites especificar a cual de los ciclos pertenecen tus sentencias `break` o `continue`. Como en la mayoria de los lenguajes, por defecto un `break` o `continue` aplicara a el ciclo mas interno. En el caso de que desearas aplicar un `break` o `continue` para alguno de los ciclos externos, puedes usar etiquetas para especificar a cual ciclo aplica la sentencia `break` o `continue`. Lo siguiente solo imprimira cuando ambos `x` y `y` sean impares:


```rust
'exterior: for x in 0..10 {
    'interior: for y in 0..10 {
        if x % 2 == 0 { continue 'exterior; } // continua el ciclo por encima de x
        if y % 2 == 0 { continue 'interior; } // continua el ciclo por encima de y
        println!("x: {}, y: {}", x, y);
    }
}
```
