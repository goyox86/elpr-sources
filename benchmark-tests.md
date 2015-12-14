% Pruebas de rendimiento

Rust soporta benchmarks, los cuales pueden probar el rendimiento de tu código. Pongamos esto en nuestro `src/lib.rs` (comentarios omitidos):

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

Nota el feature gate `test`, que habilita esta facilidad inestable.

Hemos importado el crate `test`, que contiene nuestro soporte para pruebas de desempeño. Tenemos también una función nueva, con el atributo `bench`. A diferencia de las pruebas regulares, los cuales no reciben argumentos, las pruebas de rendimiento reciben un un `&mut Bencher`. Dicho `Bencher` provee de un método `iter`, que recibe un closure como argumento. Dicho closure contiene el código al cual queremos medir el desempeño.

Podemos ejecutar los tests con `cargo bench`:

```bash
$ cargo bench
   Compiling sumador v0.0.1 (file:///home/tu/proyectos/sumador)
     Running target/release/sumador-91b3e234d4ed382a

running 2 tests
test tests::funciona ... ignored
test tests::bench_suma_dos ... bench:  1 ns/iter (+/- 0)

test result: ok. 0 passed; 0 failed; 1 ignored; 1 measured
```

Nuestro test regular fue ignorado. Puedes haber notado también que `cargo bench` se demora un poco mas que `cargo test`. Esto es debido a que Rust ejecuta las pruebas de rendimiento unas cuantas veces, tomando luego el promedio. Debido a que estamos haciendo tan poco en este ejemplo, tenemos un `1 ns/iter (+/- 0)`, pero, de existir alguna varianza, hubiera sido mostrada aquí.

Algunos consejos en la escritura de pruebas de rendimiento:

* Coloca el código de inicialización fuera del loop `iter`, solo coloca en su interior la parte que deseas medir
* Haz que el código "haga lo mismo" en cada iteración; no acumules o cambies estado
* Haz la función exterior idempotente también, el ejecutador de pruebas de rendimiento lo correrá potencialmente unas cuantas veces
* Haz el ciclo interno `iter` corto y rápido de manera tal que las pruebas de rendimiento sean rápidas y el calibrador pueda ajustar la duración de ejecución a una buena resolución.
* Haz del código en ciclo `iter` algo simple, para ayudar a la identificación de mejoras de rendimiento (o regresiones)

## Gotcha: optimizaciones

Hay otra parte difícil acerca de escribir pruebas de rendimiento: los benchmarks compilados con optimizaciones activadas pueden ser cambiados de manera dramática por el optimizador de una manera que la prueba deje de medir lo que esperas. Por ejemplo, el compilador podría reconocer que algún calculo no posea efectos externos removiéndolo por completo.

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

El ejecutador de pruebas de rendimiento ofrece dos formas de evitar lo anterior. Bien sea, el closure que el método `iter` recibe puede retornar un valor arbitrario que obligue a el optimizador a considerar el resultado usado asegurandose que la computación no sea removida por completo. Esto puede ser logrado para el ejemplo anterior ajustando la llamada a `b.iter` a:

```rust
# struct X;
# impl X { fn iter<T, F>(&self, _: F) where F: FnMut() -> T {} } let b = X;
b.iter(|| {
    // note la omisión de `;` ( pudimos haber usado un también `return` explicito).
    (0..1000).fold(0, |viejo, nuevo| viejo ^ nuevo)
});
```

La otra opción es llamar a la función genérica `test::black_box`, que actúa como una "caja negra" opaca para el optimizador obligándolo a considerar cualquier argumento usado.

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

Ninguna de las anteriores lee o modifica el valor, y son realmente baratas para valores pequeños, valores mas grandes pueden ser pasados de manera indirecta para reducir el overhead (e.j. `black_box(&struct_inmenso)`)

Efectuar cualquiera de los cambios anteriores produce los siguientes resultados para las pruebas de rendimiento:


```text
running 1 test
test bench_xor_1000_enteros ... bench: 131 ns/iter (+/- 3)

test result: ok. 0 passed; 0 failed; 0 ignored; 1 measured
```

Sin embargo, aun cuando se usen cualquiera de las técnicas anteriores el optimizador todavía podría modificar un caso de pruebas de una manera indeseada.