% Sintaxis Universal de Llamadas a Función

Algunas veces, varias funciones pueden tener el mismo nombre. Considera el siguiente código:

```rust
trait Foo {
    fn f(&self);
}

trait Bar {
    fn f(&self);
}

struct Baz;

impl Foo for Baz {
    fn f(&self) { println!("impl de Foo en Baz"); }
}

impl Bar for Baz {
    fn f(&self) { println!("impl de Bar en Baz"); }
}

let b = Baz;
```

Si intentaramos llamar `b.f()`, obtendriamos un error:

```text
error: multiple applicable methods in scope [E0034]
b.f();
  ^~~
note: candidate #1 is defined in an impl of the trait `main::Foo` for the type
`main::Baz`
    fn f(&self) { println!("Baz’s impl of Foo"); }
    ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
note: candidate #2 is defined in an impl of the trait `main::Bar` for the type
`main::Baz`
    fn f(&self) { println!("Baz’s impl of Bar"); }
    ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

```

Necesitamos una forma de eliminar la ambigüedad. Esta característica es denominada ‘sintaxis universal de llamadas a función’, y luce así:


```rust
# trait Foo {
#     fn f(&self);
# }
# trait Bar {
#     fn f(&self);
# }
# struct Baz;
# impl Foo for Baz {
#     fn f(&self) { println!("impl de Foo en Baz"); }
# }
# impl Bar for Baz {
#     fn f(&self) { println!("impl de Bar en Baz"); }
# }
# let b = Baz;
Foo::f(&b);
Bar::f(&b);
```

Analicémoslo por partes.

```rust,ignore
Foo::
Bar::
```

Dichas mitades de invocacion son los tipos de los traits: `Foo` y `Bar`. Lo anterior es responsable de la eliminación de la ambigüedad entre las dos llamadas: Rust llama la función del trait que nombraste.

```rust,ignore
f(&b)
```

Cuando llamamos un método de la forma `b.f()` usando la [sintaxis de métodos][methodsyntax], Rust automáticamente tomara prestado a `b` si `f()` recibe a `&self`. En este caso, Rust no puede hacerlo. Es por ello que debemos pasar a `&b` de manera explicita.

[methodsyntax]: method-syntax.html

# Usando <>

La forma de SULF de la que acabamos de hablar:

```rust,ignore
Trait::metodo(args);
```

Es una version corta. Existe una version expandida que puede ser necesaria en algunas situaciones:

```rust,ignore
<Tipo as Trait>::metodo(args);
```

La sintaxis `<>::` es una forma de proporcionar una pista de tipos. El tipo va dentro de los `<>`s. En este caso es `Type as Trait` indicando que deseamos llamar a la version `Trait` de `metodo`. La sección `as Trait` es opcional de no ser ambigua. Lo mismo con los `<>`s, de allí proviene version mas corta.

Acá un ejemplo del uso de la forma mas larga.

```rust
trait Foo {
    fn clone(&self);
}

#[derive(Clone)]
struct Bar;

impl Foo for Bar {
    fn clone(&self) {
        println!("Creando un clon de Bar");

        <Bar as Clone>::clone(self);
    }
}
```

Lo anterior llamara el metodo `clone` en el trait `Clone`, en vez de la version `clone` del trait `Foo`.
