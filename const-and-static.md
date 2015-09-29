% `const` y `static`

Rust posee una manera de definir constantes con la palabra reservada `const`:

```rust
const N: i32 = 5;
```

A diferencia de los enlaces a variable [`let`][let], debes anotar el tipo de un `const`.

[let]: variable-bindings.html

Las constantes viven por el tiempo de vida completo del programa. Para ser mas específicos, las constantes en Rust no tienen direcciones de memoria fijas. Esto es debido a que estas son insertadas en linea en cada lugar en el que son usadas. Por esta razón, referencias a la misma constante no están garantizadas a apuntar a la misma dirección de memoria.

# `static`

Rust proporciona una facilidad de tipo ‘variable global’ en items estáticos.  Son similares a las constantes, pero los items estáticos no son insertados en linea tras su uso. Esto significa que solo existe una instancia para cada valor, y esta ubicada en una dirección de memoria fija.

He aqui un ejemplo:

```rust
static N: i32 = 5;
```

A diferencia de los enlaces a variable [`let`][let], debes anotar el tipo de un `static`.

Los items `static` viven por el tiempo de vida del programa completo, y en consecuencia cualquier referencia es almacenada en una constante posee  [`un tiempo de vida 'static`][lifetimes]:

```rust
static NOMBRE: &'static str = "Steve";
```

[lifetimes]: lifetimes.html

## Mutabilidad

Puedes introducir mutabilidad con la palabra reservada `mut`:

```rust
static mut N: i32 = 5;
```

Debido que es mutable, un hilo podría estar actualizando `N` mientras que otro este leyéndolo, causando inseguridad en el manejo de memoria. Es por ello que ambos el acceso y la mutación de un `static mut` son [`inseguros`][unsafe], y en consecuencia deben ser efectuados dentro de un bloque `unsafe`:

```rust
# static mut N: i32 = 5;

unsafe {
   N += 1;

   println!("N: {}", N);
}
```

[unsafe]: unsafe.html

Aunado a esto, cualquier tipo almacenado en un `static` debe ser `Sync`, y podría no tener una implementación de [`Drop`][drop].

[drop]: drop.html

# Inicialización

Ambos `const` y `static` tienen requerimientos al proporcionarles un valor. Solo se les puede proporcionar un valor que sea una expresión constante. En otras palabras, no puedes usar el resultado de una llamada a función, algo similarmente complejo o algo determinado en tiempo de ejecución.

# Cual construcción debería usar?

Casi siempre, si puedes escoger entre las dos, elige `const`. Es bastante raro que quieras tener una dirección de memoria asociada con tu constante, también, el usar const permite optimizaciones como propagación de constantes no solo en tu crate si no en los crates subyacentes.
