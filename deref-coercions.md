% Coerciones Deref

La biblioteca estándar proporciona un trait especial [`Deref`][deref]. Es usado normalmente para sobrecargar `*`, el operador de dereferencia:

```rust
use std::ops::Deref;

struct EjemploDeref<T> {
    valor: T,
}

impl<T> Deref for EjemploDeref<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.valor
    }
}

fn main() {
    let x = EjemploDeref { valor: 'a' };
    assert_eq!('a', *x);
}
```

[deref]: ../std/ops/trait.Deref.html

Lo anterior es útil para escribir tipos personalizados de apuntadores. Sin embargo, hay una facilidad del lenguaje relacionada a `Deref`: las ‘coerciones deref’. He aquí la regla: Si tienes un tipo `U`, y este implementa `Deref<Target=T>`, los valores de `&U` harán coercion Automatica a un `&T`. A continuación un ejemplo:

```rust
fn foo(s: &str) {
    // tomar la cadena prestada por un segundo
}

// String implementa Deref<Target=str>
let owned = "Hola".to_string();

// entonces, esto funciona:
foo(&owned);
```

Usando un ampersand en frente del valor tomamos una referencia a el. Entonces `owned` es un `String`, `&owned` es un `&String`, y debido a `impl Deref<Target=str> para `String`, `&String` hará deref a `&str` que es tomado por foo()`.

Eso es todo. Dicha regla es uno de los únicos lugares en los que Rust hace conversiones automáticas por nosotros, pero al mismo tiempo agrega mucha flexibilidad. Por ejemplo el tipo `Rc<T>` implementa `Deref<Target=T>`, de manera que esto funciona:

```rust
use std::rc::Rc;

fn foo(s: &str) {
    // tomar la cadena prestada por un segundo
}

// String implementa Deref<Target=str>
let owned = "Hello".to_string();
let contado = Rc::new(owned);

// entonces, esto funciona:
foo(&contado);
```

Todo lo que hemos hecho es envolver nuestro `String` en un `Rc<T>`. Pero ahora podemos pasar el `Rc<String>` a cualquier lugar en el cual tengamos un a `String`. La firma de `foo` no cambio, pero funciona de igual forma con cualquiera de los dos tipos. Este ejemplo tiene dos conversiones: de `Rc<String>` a `String` y luego de `String` a `&str`. Rust hará esto tantas veces como sea posible hasta que los tipos coincidan.

Otra implementación muy común proporcionada por la biblioteca estándar es:

```rust
fn foo(s: &[i32]) {
    // tomar el pedazo prestado por un segundo
}

// Vec<T> implementa Deref<Target=[T]>
let owned = vec![1, 2, 3];

foo(&owned);
```

Los vectores pueden hacer `Deref` a un slice.

## Deref y llamadas a metodo

`Deref` también entrara en efecto cuando se llame a un método. Considera el siguiente ejemplo:

```rust
struct Foo;

impl Foo {
    fn foo(&self) { println!("Foo"); }
}

let f = &&Foo;

f.foo();
```

Aunque `f` es un `&&Foo` y `Foo` recibe a `&self`, lo anterior funciona. Todo esto como consecuencia de que todas estas cosas son lo mismo:

```rust,ignore
f.foo();
(&f).foo();
(&&f).foo();
(&&&&&&&&f).foo();
```

Un valor de tipo `&&&&&&&&&&&&&&&&Foo` puede aun tener llamadas a métodos definidos en `Foo` debido a que el compilador insertara tantas operaciones * sean necesarias para hacerlo funcionar. Y debido a que inserta `*`s, se hace uso de `Deref`.
