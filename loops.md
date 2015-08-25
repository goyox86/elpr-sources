% Ciclos

Rust actualmente provee tres enfoques para realizar actividad iterativa. `loop`, `while` y `for`. Cada uno de dichos enfoques tiene sus propios usos.

## loop

El ciclo infinito `loop` es el ciclo mas simple disponible en Rust. A traves del uso de la palabra reservada `loop`, Rust proporciona una forma de iterar indefinidamente hasta que alguna sentencia de terminación sea alcanzada. El `loop` infinito de Rust luce así:


```rust,ignore
loop {
    println!("Itera por siempre!");
}
```

## while

Rust también tiene un ciclo `while`. Luce de esta manera:

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

Los ciclos `while` son la elección correcta cuando no estas seguro acerca de cuantas veces necesitas iterar.

De necesitar un ciclo infinito, podrías sentirte tentado a escribir algo como esto:

```rust,ignore
while true {
```

Sin embargo, `loop` es por lejos, el mejor para este caso:

```rust,ignore
loop {
```

El análisis de flujo de control de Rust trata esta construcción de manera diferente que a un `while true`, debido a que sabemos que iteraremos por siempre. En general, mientras mas información le proporcionemos al compilador, este podría desempeñarse mejor en relación a la seguridad y generación de código, es por ello que siempre deberías preferir `loop` cuando tengas planeado iterar de manera indefinida.


## for

El ciclo `for` es usado para iterar un numero particular de veces. Los ciclos `for` de Rust, sin embargo, trabajan de manera diferente a los de otros lenguajes de programación de sistemas. El `for` de Rust no luce como este ciclo `for` al “estilo C”


```c
for (x = 0; x < 10; x++) {
    printf( "%d\n", x );
}
```

En su lugar, el ciclo `for` de Rust luce así:

```rust
for x in 0..10 {
    println!("{}", x); // x: i32
}
```

En términos ligeramente mas abstractos:

```ignore
for var in expresion {
    código
}
```

La expresión es un [iterador][iterator].  El iterador retorna una serie de elementos. Cada elemento es una iteración del ciclo. Ese valor es a su vez asignado a el nombre `var`, el cual es valido en el cuerpo del ciclo. Una vez que el ciclo ha terminado, el siguiente valor es obtenido del iterador, y se itera una vez mas. Cuando no hay mas valores, el ciclo `for` termina.

[iterator]: iterators.html

En nuestro ejemplo, `0..10` es una expresión que toma una posición de inicio y una de fin, y devuelve un iterador por sobre esos valores. El limite superior es exclusivo, entonces, nuestro loop imprimirá de `0` hasta `9`, no `10`.

Rut no posee el ciclo `for` al estilo de C, a proposito. Controlar manualmente cada elemento del ciclo es complicado y propenso a errores, incluso para programadores C experimentados.

### Enumerate

Cuando necesitas llevar registro de cuantas veces has iterado, puedes usar la función `.enumerate()`.

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

No olvides colocar los paréntesis alrededor del rango.

#### En iteradores:

```rust
# let lineas = "hola\nmundo".lines();
for (numero_linea, linea) in lineas.enumerate() {
    println!("{}: {}", numero_linea, linea);
}
```

Outputs:

```text
0: Contenido de la linea uno
1: Contenido de la linea dos
2: Contenido de la linea tres
3: Contenido de la linea cuatro
```

## Finalizando la iteración de manera temprana

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

Necesitamos mantener un binding a variable `mut`, `completado`, para saber cuando debíamos salir del ciclo. Rust posee dos palabras clave para ayudarnos a modificar el proceso de iteración: `break` y `continue`.

En este caso, podemos escribir el ciclo de una mejor forma con `break`:

```rust
let mut x = 5;

loop {
    x += x - 3;

    println!("{}", x);

    if x % 5 == 0 { break; }
}
```

Ahora iteramos de manera indefinida con `loop` y usamos `break` para romper el ciclo de manera temprana. Usar un `return` explícito también sirve para la terminación temprana del ciclo.

`continue` es similar, pero en lugar de terminar el ciclo, nos hace ir a la siguiente iteración. Lo siguiente imprimirá los numeros impares:

```rust
for x in 0..10 {
    if x % 2 == 0 { continue; }

    println!("{}", x);
}
```

## Etiquetas loop

También podrías encontrar situaciones en las cuales tengas ciclos anidados y necesites especificar a cual de los ciclos pertenecen tus sentencias `break` o `continue`. Como en la mayoría de los lenguajes, por defecto un `break` o `continue` aplica a el ciclo mas interno. En el caso de que desearas aplicar un `break` o `continue` para alguno de los ciclos externos, puedes usar etiquetas para especificar a cual ciclo aplica la sentencia `break` o `continue`. Lo siguiente solo imprimirá cuando ambos `x` y `y` sean impares:


```rust
'exterior: for x in 0..10 {
    'interior: for y in 0..10 {
        if x % 2 == 0 { continue 'exterior; } // continua el ciclo por encima de x
        if y % 2 == 0 { continue 'interior; } // continua el ciclo por encima de y
        println!("x: {}, y: {}", x, y);
    }
}
```
