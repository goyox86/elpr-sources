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

Posiblemente notes que esto es muy similar, pero no exactamente lo mismo, a la definicion de una condicion de carrera:


> There is a ‘data race’ when two or more pointers access the same memory
> location at the same time, where at least one of them is writing, and the
> operations are not synchronized.

With references, you may have as many as you’d like, since none of them are
writing. If you are writing, you need two or more pointers to the same memory,
and you can only have one `&mut` at a time. This is how Rust prevents data
races at compile time: we’ll get errors if we break the rules.

With this in mind, let’s consider our example again.

## Thinking in scopes

Here’s the code:

```rust,ignore
let mut x = 5;
let y = &mut x;

*y += 1;

println!("{}", x);
```

This code gives us this error:

```text
error: cannot borrow `x` as immutable because it is also borrowed as mutable
    println!("{}", x);
                   ^
```

This is because we’ve violated the rules: we have a `&mut T` pointing to `x`,
and so we aren’t allowed to create any `&T`s. One or the other. The note
hints at how to think about this problem:

```text
note: previous borrow ends here
fn main() {

}
^
```

In other words, the mutable borrow is held through the rest of our example. What
we want is for the mutable borrow to end _before_ we try to call `println!` and
make an immutable borrow. In Rust, borrowing is tied to the scope that the
borrow is valid for. And our scopes look like this:

```rust,ignore
let mut x = 5;

let y = &mut x;    // -+ &mut borrow of x starts here
                   //  |
*y += 1;           //  |
                   //  |
println!("{}", x); // -+ - try to borrow x here
                   // -+ &mut borrow of x ends here
```

The scopes conflict: we can’t make an `&x` while `y` is in scope.

So when we add the curly braces:

```rust
let mut x = 5;

{                   
    let y = &mut x; // -+ &mut borrow starts here
    *y += 1;        //  |
}                   // -+ ... and ends here

println!("{}", x);  // <- try to borrow x here
```

There’s no problem. Our mutable borrow goes out of scope before we create an
immutable one. But scope is the key to seeing how long a borrow lasts for.

## Issues borrowing prevents

Why have these restrictive rules? Well, as we noted, these rules prevent data
races. What kinds of issues do data races cause? Here’s a few.

### Iterator invalidation

One example is ‘iterator invalidation’, which happens when you try to mutate a
collection that you’re iterating over. Rust’s borrow checker prevents this from
happening:

```rust
let mut v = vec![1, 2, 3];

for i in &v {
    println!("{}", i);
}
```

This prints out one through three. As we iterate through the vectors, we’re
only given references to the elements. And `v` is itself borrowed as immutable,
which means we can’t change it while we’re iterating:

```rust,ignore
let mut v = vec![1, 2, 3];

for i in &v {
    println!("{}", i);
    v.push(34);
}
```

Here’s the error:

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

We can’t modify `v` because it’s borrowed by the loop.

### use after free

References must not live longer than the resource they refer to. Rust will
check the scopes of your references to ensure that this is true.

If Rust didn’t check this property, we could accidentally use a reference
which was invalid. For example:

```rust,ignore
let y: &i32;
{
    let x = 5;
    y = &x;
}

println!("{}", y);
```

We get this error:

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

In other words, `y` is only valid for the scope where `x` exists. As soon as
`x` goes away, it becomes invalid to refer to it. As such, the error says that
the borrow ‘doesn’t live long enough’ because it’s not valid for the right
amount of time.

The same problem occurs when the reference is declared _before_ the variable it
refers to. This is because resources within the same scope are freed in the
opposite order they were declared:

```rust,ignore
let y: &i32;
let x = 5;
y = &x;

println!("{}", y);
```

We get this error:

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

In the above example, `y` is declared before `x`, meaning that `y` lives longer
than `x`, which is not allowed.
