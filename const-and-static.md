% `const` y `static`

Rust posee una manera de definir constantes con la palabra reservada `const`:

```rust
const N: i32 = 5;
```

A diferencia de los enlaces a variable [`let`][let], debes anotar el tipo de un `const`.

[let]: variable-bindings.html

Las constantes viven por el tiempo de vida completo del programa. Para ser mas especificos, las constantes en Rust no tienen direcciones de memoria fijas. Esto es debido a que estas son insertadas en linea en cada lugar en el que son usadas. Por esta razon, referencias a la misma constante no estan garantizadas a apuntar a la misma direccion de memoria.

# `static`

Rust proporciona una facilidad de tipo ‘variable global’ en items estaticos.  Son similares a las constantes, pero los items estaticos no son inertados en linea tras su uso. Esto significa que solo existe una instancia para cada valor, y esta ubicada en una direccion de memoria fija.

He aqui un ejemplo:

```rust
static N: i32 = 5;
```

A diferencia de los enlaces a variable [`let`][let] , debes anotar el tipo de un `static`.

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

Debido que es mutable, un hilo podria estar actualizando `N` mientras que otro este leyendolo, causando inseguridad en el manejo de memoria. Es por ello que ambos el acceso y la mutacion de un `static mut` es [`inseguro`][unsafe], y enconsecuencia debe ser efectuado dentro de un bloque `unsafe`:

```rust
# static mut N: i32 = 5;

unsafe {
   N += 1;

   println!("N: {}", N);
}
```

[unsafe]: unsafe.html

Aunado a esto, cualquier tipo almacenado en un `static` debe ser `Sync`, y podria no tener una implementacion de [`Drop`][drop].

[drop]: drop.html

# Inicializacion

Amobs `const` y `static` tienen requerimientos para proporcionarles un valor. Solo se les puede proporcionar el valor que sea una expresion constante. En otras palabras, no puedes usar el resultado de una llamada a funcion, algo similarmente complejo o en tiempo de ejecucion.

# Cual construccion deberia usar?

Casi siempre, si puedes escojer entre las dos, elige `const`. Es bastatnte raro que quieras tener una direccion de memoria asociada con tu constante, tambien, el usar const permite optimizaciones como propagacion de constantes no solo en tu crate si no en los crates subyacentes.
