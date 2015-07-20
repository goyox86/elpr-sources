% Borrow y AsRef

Lost traits [`Borrow`][borrow] and [`AsRef`][asref] son muy similares, pero diferentes. He aquí un breve repaso acerca de lo que significan dichos traits:

[borrow]: ../std/borrow/trait.Borrow.html (ingles)
[asref]: ../std/convert/trait.AsRef.html (ingles)

# Borrow

El trait `Borrow` se usa cuando estamos escribiendo un estructura de datos, y deseamos usar un tipo owned o un tipo borrowed como sinónimo por alguna razón.

Por ejemplo, [`HashMap`][hashmap] posee un [metodo `get`][get] que usa `Borrow`:

```rust,ignore
fn get<Q: ?Sized>(&self, k: &Q) -> Option<&V>
    where K: Borrow<Q>,
          Q: Hash + Eq
```

[hashmap]: ../std/collections/struct.HashMap.html
[get]: ../std/collections/struct.HashMap.html#method.get

Esta firma es complicada. El parametro `K` es en lo que estamos interesados ahora. Se refiere a un parametro del `HashMap` en si mismo:

```rust,ignore
struct HashMap<K, V, S = RandomState> {
```

El parametro `K` es el tipo de la *clave* que usa el `HashMap`. Al ver de nuevo la firma de `get()`, solo podemos usar `get()` cuando la clave implementa `Borrow<Q>`. De esta manera, podemos crear un `HashMap` que use claves  `String` pero al momento de buscar use `&str`s:

```rust
use std::collections::HashMap;

let mut map = HashMap::new();
map.insert("Foo".to_string(), 42);

assert_eq!(map.get("Foo"), Some(&42));
```

Esto es debido a que la biblioteca estándar posee una `impl Borrow<str> for String`.

Para la mayoría de los tipos, cuando desees tomar un tipo ya sea owned o borrowed, un &T es suficiente. Pero un area en la cual `Borrow` es efectivo es cuando hay mas de un tipo de valor borrowed. Esto es especialmente cierto en referencias y slices: puedes tener ambos un `&T` un `&mut T`. Si queremos aceptar ambos tipos `Borrow` es lo que necesitamos:

```rust
use std::borrow::Borrow;
use std::fmt::Display;

fn foo<T: Borrow<i32> + Display>(a: T) {
    println!("a es un borrowed: {}", a);
}

let mut i = 5;

foo(&i);
foo(&mut i);
```

This will print out `a es un borrowed: 5` dos veces.

# AsRef

El trait `AsRef` es un trait de conversion. Es usado para convertir algún valor a una referencia en código generico. Como esto:

```rust
let s = "Hola".to_string();

fn foo<T: AsRef<str>>(s: T) {
    let slice = s.as_ref();
}
```

# Cual debería usar?

Podemos ver como ambos son una especie de lo mismo: ambos lidian con las versiones owned y borrowed de algún tipo. Sin embargo, son diferentes.

Escoge `Borrow` cuando quieras abstraerte por encima de diferentes tipos de borrowing, o cuando estas construyendo una estructura de datos que trata los valores owned y borrowed de formas equivalentes, como el hashing y la comparación.

Usa `AsRef` cuando desees convertir algo directamente en una referencia, y estés escribiendo código genérico.
