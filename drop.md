% Drop

Ahora que hemos discutido los traits, hablems de un trait particular proporcionado por la biblioteca estandar de Rust, [`Drop`][drop]. El trait `Drop` provee una forma de ejecutar codigo cuando un valor sale de ambito. Por ejemplo:

[drop]: ../std/ops/trait.Drop.html

```rust
struct TieneDrop;

impl Drop for TieneDrop {
    fn drop(&mut self) {
        println!("Dropeando!");
    }
}

fn main() {
    let x = TieneDrop;

    // hacemos algo

} // x sale de ambito aqui
```

Cuando `x` sale de ambito al final de `main()`, el codigo de `Drop` es ejecutado. `Drop` posee un metodo, tambien denominado `drop()`. Dicho metodo toma una referencia mutable a `self`.

Eso es todo! La mecanica de `Drop` es muy simple, sin embargo, hay algunos detalles. Por ejemplo, los valores son liberados (dropped) en orden opuesto a como fueron declarados. He aqui otro ejemplo:

```rust
struct Explosivo {
    potencia: i32,
}

impl Drop for Explosivo {
    fn drop(&mut self) {
        println!("BOOM multiplicado por {}!!!", self.potencia);
    }
}

fn main() {
    let petardo = Explosivo { potencia: 1 };
    let tnt = Explosivo { potencia: 100 };
}
```

Lo anterior imprimira:

```text
BOOM multiplicado por 100!!!
BOOM multiplicado por 1!!!
```

El TNT se va primero que el pertado, debido que fue creado despues, Ultimo en entrar, primero en salir.

Entoces para que es bueno `Drop`? Generalmente, es usado para limipar cualquier recurso asociado a un `struct`. Por ejemplo, el [tipo `Arc<T>`][arc] es un tipo con conteo de referencias. Cuando `Drop` es llamado, este decrementara el contador de referencias, y si el numero total de referencias es cero, limipara el valor subyacente.

[arc]: ../std/sync/struct.Arc.html
