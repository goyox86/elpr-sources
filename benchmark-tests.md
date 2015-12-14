% Pruebas de rendimiento

Rust soporta benchmarks, los cuales pueden probar el rendimiento de tu codigo. Pongamos esto en nuestro  `src/lib.rs` (comentarios omitidos):

```rust,ignore
#![feature(test)]

extern crate test;

pub fn suma_dos(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;
    use test::Bencher;

    #[test]
    fn funciona() {
        assert_eq!(4, suma_dos(2));
    }

    #[bench]
    fn bench_suma_dos(b: &mut Bencher) {
        b.iter(|| add_two(2));
    }
}
```

Nota el faeture gate `test`, que habilita esta facilidad inestable.

Hem,os importado el crate `test`, el cual contiene nuestro soporte para pruebas de desempeno. Tenemos tambien una funcion nueva, con el atributo `bench`. A diferencia de las pruebas regulares, los cuales no reciben argumentos, las pruebas de rendimiento reciben un un `&mut Bencher`. Dicho `Bencher` provee un metodo `iter`, que recibe un closure como argumento. Dicho closure contiene el codigo al cual queremos medir el desempeno.

Podemos ejecutar los tests con `cargo bench`:

```bash
$ cargo bench
   Compiling sumador v0.0.1 (file:///home/tu/proyectos/sumador)
     Running target/release/adder-91b3e234d4ed382a

running 2 tests
test tests::funciona ... ignored
test tests::bench_suma_dos ... bench:  1 ns/iter (+/- 0)

test result: ok. 0 passed; 0 failed; 1 ignored; 1 measured
```

Nuestro test regular fue ignorado. Puedes haber notado tambien que `cargo bench` se demora un poco mas que `cargo test`. Esto es debido a que Rust ejecuta las pruebas de rendimiento varias veces, y luego toma el promedio. Debido a que estamos haciendo tan poco en este ejemplo =, tenemos un `1 ns/iter (+/- 0)`, pero este mostraria la varianza de haber alguna.

Consejos en la escritura de pruebas de rendimiento:

* Coloca el codigo de inicializacion fuera del loop `iter`, solo coloca en su interior la parte que deseas medir
* Haz que el codigo "haga lo mismo" en cada iteracion; no acumules o cambies estado
* Haz la funcion exterior idempotente tambien, el ejecutador de pruebas de rendimiento lo correra potencialmente unas cuantes veces
* Haz el ciclo interno `iter` corto y rapido de manera tal que las pruebas de rendimiento sean rapidas y el calibrador pueda ajustar la longitud de ejecucion a una buena resolucion.
* Haz del codigo en ciclo `iter` al;go simple, para ayudar a la identificacion de mejoras de rendimiento (o regresiones)

## Gotcha: optimizaciones

Hay otra parte dificil acerca de escribir pruebas de rendimiento: los benchmarks compilados con optimizaciones activadas pueden ser cambiados de manera dramatica por el optimizador de una manera que la prueba ya no este midiendo lo que esperas. Por ejemplo, el compilador podira reconocer que algun calculo no posea efectos externos removiendolo por completo.

```rust,ignore
#![feature(test)]

extern crate test;
use test::Bencher;

#[bench]
fn bench_xor_1000_enteros(b: &mut Bencher) {
    b.iter(|| {
        (0..1000).fold(0, |viejo, nuevo| viejo ^ nuevo);
    });
}
```

produce los siguientes resultados

```text
running 1 test
test bench_xor_1000_enteros ... bench: 0 ns/iter (+/- 0)

test result: ok. 0 passed; 0 failed; 0 ignored; 1 measured
```

El ejecutador de pruebas de rendimiento ofrece dos formas de evitar lo anterior. Bien sea, el closure que el metodo `iter` recibe puede retornar un valor arbitrario que obligue a el optimizador a considerar el resultado usado y asegurandose que la computacion no sea removida por completo. Esto puede ser logrado para el ejemplo anterior ajustando la llamada `b.iter` a:

```rust
# struct X;
# impl X { fn iter<T, F>(&self, _: F) where F: FnMut() -> T {} } let b = X;
b.iter(|| {
    // note la omision de `;` ( pudimos haber usado un tambien `return` explicito).
    (0..1000).fold(0, |viejo, nuevo| viejo ^ nuevo)
});
```

La otra opcion es llamar a la funcion generica `test::black_box`, que actua como una "caja negra" opaca para el optimizador obligandolo a considerar cualquier argumento usado.

```rust
#![feature(test)]

extern crate test;

# fn main() {
# struct X;
# impl X { fn iter<T, F>(&self, _: F) where F: FnMut() -> T {} } let b = X;
b.iter(|| {
    let n = test::black_box(1000);

    (0..n).fold(0, |a, b| a ^ b)
})
# }
```

Ninguna de las anteriores lee o modifica el valor, y son relamente baratas para valores pequenos, Valores mas grandes pueden ser pasados de manera indirecta para reducir el overhead (e.j. `black_box(&struct_inmenso)`)

Llevar a cabo cualquier de los cambios anteriores produce los siguientes resultados para las pruebas de rendimiento


```text
running 1 test
test bench_xor_1000_enteros ... bench: 131 ns/iter (+/- 3)

test result: ok. 0 passed; 0 failed; 0 ignored; 1 measured
```

Sin embargo, el optimizador pueden aun modificar un  caso de pruebas de una maera indeseada aun mientras se use cualquiera de las tecnicas anteriores.
