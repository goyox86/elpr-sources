% Referencias y Préstamo

Esta guia es una de las tres presentando el sistema de pertenencia de Rust. Esta es una de las caracteristicas mas unicas y atractivas de Rust, con la que los desarrolladores Rust deben estar bien familiarizados. La pertenencia es como Rust logra su objetivo mayor, seguridad en el manejo de memoria. Existen unos pocos conceptos distintos, cada uno con su propio capitulo:

* [pertenencia][ownership], el concepto principal
* prestamo, el que lees ahora
* [tiempos de vida][lifetimes], un concepto avanzado del prestamo

Estos tres capitulos estan relacionados, y en orden. Necesitaras leer los tres para entender completamente el sistema de pertenencia.

[ownership]: ownership.html
[lifetimes]: lifetimes.html

# Meta

Antes de entrar en detalle, dos notas importantes acerca del sistema de pertenencia.

Rust tiene foco en seguridad y velocidad. Rust logra esos objetivos a travez de muchas ‘abstracciones de cero costo’, lo que significa que en Rust, las abstracciones cuestan tan poco como sea posible para hacerlas funcionar. El sistema de pertenencia es un ejemplo primordial de una abstracción de cero costo. Todo el análisis del que estaremos hablando en la presente guía es _llevado a cabo en tiempo de compilación_. No pagas ningún costo en tiempo de ejecución por ninguna de estas facilidades.

Sin embargo, este sistema tiene cierto costo: la curva de aprendizaje. Muchos usuarios nuevos Rust experimentan algo que nosotros denominamos ‘pelear con el comprobador de préstamo’ (‘fighting with the borrow checker’), situación en la cual el compilador de Rust se rehusa a compilar un programa el cual el autor piensa valido. Esto ocurre con frecuencia debido a que el modelo mental del programador acerca de como funciona la pertenencia no concuerda con las reglas actuales implementadas en Rust. Probablemente tu experimentes cosas similares al comienzo. Sin embargo, hay buenas noticias: otros desarrolladores Rust experimentados  reportan que una vez que trabajan con las reglas del sistema de pertenencia por un periodo de tiempo, pelean cada vez menos con el comprobador de préstamo.

Con eso en mente, aprendamos acerca de el prestamo.

# Prestamo

Al final de la seccion de [pertenencia][ownership], teniamos una funcion fea que lucia asi:

```rust
fn foo(v1: Vec<i32>, v2: Vec<i32>) -> (Vec<i32>, Vec<i32>, i32) {
    // hacer algo con v1 y v2

    // devolviendo pertenencia, así como el resultado de nuestra función
    (v1, v2, 42)
}

let v1 = vec![1, 2, 3];
let v2 = vec![1, 2, 3];

let (v1, v2, respuesta) = foo(v1, v2);
```

Lo anterior no es Rust idiomatic, sin embargo, debido a que no se beneficia de las ventajas del pretamo. He aqui el primer paso:

```rust
fn foo(v1: &Vec<i32>, v2: &Vec<i32>) -> i32 {
    // hacer algo con v1 y v2

    // retornando la respuesta
    42
}

let v1 = vec![1, 2, 3];
let v2 = vec![1, 2, 3];

let respuesta = foo(&v1, &v2);

// podemos usar a v1 y v2 aqui
```

En lugar de tomar `Vec<i32>`s como nuestros argumentos, tomamos una referencia: `&Vec<i32>`. Y en lugar de pasar `v1` y `v2` directamente, pasamos `&v1` y `&v2`. Llamamos al tipo `&T` una 'referencia', y en vez de tomar pertenecia sobre el recurso, este la toma prestada. Un enlace a variable que hace un prestamo de algo no libera el recurso cuando sale de ambito. Esto significa que despues de la llamada a `foo()`, podemos nuevamente hacer uso de los enlaces a variable originales.

Las referencias son inmutables, justo como los enlaces a variable. Esto se traduce a que dentro de `foo()`, los vectores no pueden ser cambiados:

```rust,ignore
fn foo(v: &Vec<i32>) {
     v.push(5);
}

let v = vec![];

foo(&v);
```

errors with:

```text
error: cannot borrow immutable borrowed content `*v` as mutable
v.push(5);
^
```

Insertar un valor causa una mutacion en el vector, y no tenemos permitido hacerlo.

# referencias &mut

Existe un segundo tipo de referencia: `&mut T`. Una ‘referencia mutable’ te permite mutar el recurso que estas tomando prestado. Por ejemplo:

```rust
let mut x = 5;
{
    let y = &mut x;
    *y += 1;
}
println!("{}", x);
```

Lo anterior imprimira `6`. Hacemos a `y` una referencia mutable a `x`, entonces sumamos uno a lo que sea que `y` apunta. Notaras que `x` tuvo que haber sido marcado como `mut` tambien, de lo contrario, no hubiesemos podido tomar un prestamo mutable a un valor inmutable.

Notaras tambien que hemos agregado un asterisco (`*`) al frente de `y`, tornandolo en `*y`, esto es debido a que `y` es una referencia `&mut`. Tambien necesitaras hacer uso de ellos para acceder a el contenido de una referencia.

De otro modo, las referencias `&mut` son como las referencias. _Existe_ una gran diferencia entre las dos, y como estas interactuan. Podras ver que existe algo que no huele muy bien en el ejemplo anterior, puesto que necesitamos ese ambito extra, con los `{` y `}`. Si los removemos, obtenemos un error:

```text
error: cannot borrow `x` as immutable because it is also borrowed as mutable
    println!("{}", x);
                   ^
note: previous borrow of `x` occurs here; the mutable borrow prevents
subsequent moves, borrows, or modification of `x` until the borrow ends
        let y = &mut x;
                     ^
note: previous borrow ends here
fn main() {

}
^
```

Al parecer, hay reglas.

# Las Reglas

He aqui las reglas acerca del prestamo en Rust:

Primero, cualquier prestamo debe vivir en un ambito no mayor al de el dueno. Segundo, puedes tener uno u otro de estos tipos de prestamo, pero no los dos al mismo tiempo:

* una o mas referencias (`&T`) a un recurso,
* exactamente una referencia mutable (`&mut T`).

Posiblemente notes que esto es muy similar, pero no exactamente igual, a la definicion de una condicion de carrera:

> Hay una ‘condicion de carrera’ cuando dos o mas apuntadores acceden a la misma locacion en memoria al mismo tiempo, en donde al menos uno esta escribiendo, y las operaciones no estan sincronizadas.

Con las referencias, puedes tener cuantas desees, debido a que ninguna de ellas esta escribiendo. Si estas escribiendo, y necesitas dos o mas apuntadores a la misma memoria, puedes tener solo un `&mut` a la vez. Es asi como Rust previene las condiciones de carrera en tiempo de compilacion: obtendremos errores si rompemos las reglas.

Con esto en mente, consideremos nuestro ejemplo otra vez.

## Pensando en ambitos

He aqui el codigo:

```rust,ignore
let mut x = 5;
let y = &mut x;

*y += 1;

println!("{}", x);
```

El codigo anterior genera el siguiente error:

```text
error: cannot borrow `x` as immutable because it is also borrowed as mutable
    println!("{}", x);
                   ^
```

Esto es debido a que hemos vilado las reglas:  tenemos un `&mut T` apuntando a `x`, y en consecuencia no tenemos permitido crear ningun `&T`s. Una cosa u otra. La nota apunta a como pensar acerca de este problema:


```text
note: previous borrow ends here
fn main() {

}
^
```

En otras palabras, el prestamo mutable es mantenido a lo largo de el resto de nuestro ejemplo. Lo que queremos es que nuestro prestamo mutable termine _antes_ que intentemos llamar a `println!` y hagamos un prestamo inmutable. En Rust, el prestamo esta asociado al ambito en el cual el prestamo es valido. Nuestros ambitos lucen asi:


```rust,ignore
let mut x = 5;

let y = &mut x;    // -+ prestamo &mut de x comienza aqui
                   //  |
*y += 1;           //  |
                   //  |
println!("{}", x); // -+ - intento de tomar prestado x aqui
                   // -+ prestamo &mut de x termina aqui
```

Los ambitos entran en conflicto: no podemos crear un `&x` mientras `y` esta en ambito.

Entonces cuando agregamos llaves:

```rust
let mut x = 5;

{
    let y = &mut x; // -+ prestamo &mut de x comienza aqui
    *y += 1;        //  |
}                   // -+ ... y termina aqui

println!("{}", x);  // <- intento de tomar prestado x aqui
```

No hay problema. Nuestro prestamo mutable sale de ambito antes de que creemos un prestamo inmutable. El ambito es clave para ver cuanto dura el prestamo.

## Problemas que el prestamo previene

Porque tenemos estas reglas restrictivas? Bueno, como lo notamos, estas reglas previenen condiciones de carrera. Que tipos de problemas causan las condiciones de carrera? Aca unos pocos.

Why have these restrictive rules? Well, as we noted, these rules prevent data
races. What kinds of issues do data races cause? Here’s a few.

### Invalidacion de Iteradores

Un ejemplo es la ‘invalidacion de iteradores’, que ocuure cuando tratas de mutar una collecion mientras estas iterando sobre ella. El comprobador de prestamos de Rust evita que esto ocurra:

```rust
let mut v = vec![1, 2, 3];

for i in &v {
    println!("{}", i);
}
```

Lo anterior imprime desde uno hasta tres. A medida que iteramos los vectores, solo se nos proporcionan referencias a los elementos. `v` en si mismo es tomado prestado de manera inmutable, lo que se traduce en que no podamos cambiarlo mientras lo iteramos:

```rust,ignore
let mut v = vec![1, 2, 3];

for i in &v {
    println!("{}", i);
    v.push(34);
}
```

He aqui el error:

```text
error: cannot borrow `v` as mutable because it is also borrowed as immutable
    v.push(34);
    ^
note: previous borrow of `v` occurs here; the immutable borrow prevents
subsequent moves or mutable borrows of `v` until the borrow ends
for i in &v {
          ^
note: previous borrow ends here
for i in &v {
    println!(“{}”, i);
    v.push(34);
}
^
```

No podemos modificar `v` debido a que esta tomado prestado por el ciclo.

### uso despues de liberado (use after free)

Las referencias no deben vivir por mas tiempo que el recurso al cual ellas hacen referencia. Rust chequeara los ambitos de tus referencias para asegurarse de que esto sea cierto.

Si Rust no verificara esta proiedad, podiramos accidentalmente usar una referencia invalida. Por ejemplo:

```rust,ignore
let y: &i32;
{
    let x = 5;
    y = &x;
}

println!("{}", y);
```

Obtenemos el siguiente error:

```text
error: `x` does not live long enough
    y = &x;
         ^
note: reference must be valid for the block suffix following statement 0 at
2:16...
let y: &i32;
{
    let x = 5;
    y = &x;
}

note: ...but borrowed value is only valid for the block suffix following
statement 0 at 4:18
    let x = 5;
    y = &x;
}
```

En otras palabras, `y` es valido solo para el ambito en donde `x` existe. Tan pronto como `x` se va, se hace invalido hacerle referencia. Es por ello que el error dice que el prestamo, ‘ no vive lo suficiente’ (‘doesn’t live long enough’) puesto a que no es valido por la cantidad de tiempo correcto.

El mismo problema ocurre cuando la referencia es declarada _antes_ de la variable a la cual hace referencia. Esto es debido a que los recursos dentro del mismo ambito son liberados en orden opuesto al orden en el que fueron declarados:

```rust,ignore
let y: &i32;
let x = 5;
y = &x;

println!("{}", y);
```

Obtenemos este error:

```text
error: `x` does not live long enough
y = &x;
     ^
note: reference must be valid for the block suffix following statement 0 at
2:16...
    let y: &i32;
    let x = 5;
    y = &x;

    println!("{}", y);
}

note: ...but borrowed value is only valid for the block suffix following
statement 1 at 3:14
    let x = 5;
    y = &x;

    println!("{}", y);
}
```

En el ejemplo anterior, `y` es declarada antes que `x`, singificando que `y` vive mas que `x`, lo cual no esta permitido.

In the above example, `y` is declared before `x`, meaning that `y` lives longer
than `x`, which is not allowed.
