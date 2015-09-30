% Coerciones Deref

La biblioteca estandar proporciona un trait especial [`Deref`][deref]. Es usado normalmente para sobrecargar `*`, el operador de dereferencia:

```rust
uuse std::ops::Deref;

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

Lo anterior es util para escribir tipos personalizados de apuntadores. Sin embargo, hay una facilidad del lenguage relacionada a `Deref`: las ‘coerciones deref’. He aqui la regla: Si tienes un tipo `U`, y este implementa `Deref<Target=T>`, los valores de `&U` haran coercion automatica a un `&T`. A continuacion un ejemplo:

```rust
fn foo(s: &str) {
    // tomar la cadena prestada por un segundo
}

// String implementa Deref<Target=str>
let owned = "Hola".to_string();

// entonces, esto funciona:
foo(&owned);
```

Usando un ampersand en frente del valor toma una referencia a el. Entonces `owned` es un `String`, `&owned` es un `&String`, y debido a `impl Deref<Target=str> para `String`, `&String` hara deref a `&str` que es tomado por foo()`.

Eso es todo. Dicha regla es uno de los unicos lugares en los que Rust hace conversiones automaticas por ti, pero al mismo tiempo agrega mucha flexiblidad. Por ejemplo el tipo `Rc<T>` implementa `Deref<Target=T>`, demanera que esto funciona:

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

Todo lo que hemos hecho es envolver nuestro `String` en un `Rc<T>`. Pero ahora podemos pasar el `Rc<String>` a cualquier lado en el cual tengamos un a `String`. La firma de `foo` no cambio, pero funciona de igual forma con cualquiera de los dos tipos. Este ejemplo tiene dos conversiones: de `Rc<String>` a `String` y luego de `String` a `&str`. Rust hara esto tantas veces como sea posible hasta que los tipos coincidan.

Otra implementacion muy comun proporcionada por la bibliteca estandar es:

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

`Deref` tambien entrara en efecto cuando se llame a un metodo. Considera el siguiente ejemplo:

```rust
struct Foo;

impl Foo {
    fn foo(&self) { println!("Foo"); }
}

let f = &&Foo;

f.foo();
```

Aunque `f` es un `&&Foo` y `Foo` recibe a `&self`, lo anterior funciona. Todo esto puesto que todas estas cosas son lo mismo:

```rust,ignore
f.foo();
(&f).foo();
(&&f).foo();
(&&&&&&&&f).foo();
```

Un valor de tipo `&&&&&&&&&&&&&&&&Foo` puede aun tener llamadas a metodos definidos en `Foo` debido a que el compilador insertara tantas operaciones * sean necesarias para hacerlo funcionar. Y debido a que inserta `*`s, estos hacen uso de `Deref`.
