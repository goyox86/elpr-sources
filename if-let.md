% if let

`if let` te permite combinarv `if` y `let` para reducir el costo de algunos tipos de coincidencia de patrones.

Por ejemplo, digamos que tenemos algun tipo de `Option<T>`. Queremos llamar a una funcion en ello de ser un `Some<T>`, pero no hacer nada si es `None`. Seria algo asi:

```rust
# let option = Some(5);
# fn foo(x: i32) { }
match option {
    Some(x) => { foo(x) },
    None => {},
}
```

No tenemos que usar `match` aqui, por ejemplo, podriamos usar `if`:

```rust
# let option = Some(5);
# fn foo(x: i32) { }
if option.is_some() {
    let x = option.unwrap();
    foo(x);
}
```

Ninguna de esas dos opciones es particularmente atractiva. Podemos usar `if let` para hacer lo mismo de una mejor forma:


```rust
# let option = Some(5);
# fn foo(x: i32) { }
if let Some(x) = option {
    foo(x);
}
```

Si un [patron][patterns] concuerda satisfactoriamente, asocia cualquier parte apropiada del valor a los identificadores en el, para luego evaluar la expresion. Si el patron no concuerda, no pasa nada.

Si quisieras hacer algo en el caso de que el patron no concuerde, puedes usar `else`:

```rust
# let option = Some(5);
# fn foo(x: i32) { }
# fn bar() { }
if let Some(x) = option {
    foo(x);
} else {
    bar();
}
```

## `while let`

De manera similar, `while let` puede ser usado cuando desees iterar condicionalmente siempre y cuando el valor concuerde con cierto patron. Convierte codigo como este:

```rust
# let option: Option<i32> = None;
loop {
    match option {
        Some(x) => println!("{}", x),
        _ => break,
    }
}
```

En codigo como este:

```rust
# let option: Option<i32> = None;
while let Some(x) = option {
    println!("{}", x);
}
```

[patterns]: patterns.html
