% Drop

Ahora que hemos discutido los traits, hablemos de un trait particular proporcionado por la biblioteca estándar de Rust, [`Drop`][drop]. El trait `Drop` provee una forma de ejecutar código cuando un valor sale de ámbito. Por ejemplo:

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

Cuando `x` sale de ámbito al final de `main()`, el código de `Drop` es ejecutado. `Drop` posee un método, también denominado `drop()`. Dicho método toma una referencia mutable a `self`.

Eso es todo! La mecánica de `Drop` es muy simple, sin embargo, hay algunos detalles. Por ejemplo, los valores son liberados (dropped) en orden opuesto a como fueron declarados. He aquí otro ejemplo:

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

El TNT se va primero que el petardo, debido que fue creado después. Ultimo en entrar, primero en salir.

Entonces para que es bueno `Drop`? Generalmente, es usado para limpiar cualquier recurso asociado a un `struct`. Por ejemplo, el [tipo `Arc<T>`][arc] es un tipo con conteo de referencias. Cuando `Drop` es llamado, este decrementará el contador de referencias, y si el numero total de referencias es cero, limpiará el valor subyacente.

[arc]: ../std/sync/struct.Arc.html
