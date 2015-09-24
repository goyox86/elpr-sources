% if let

`if let` te permite combinar `if` y `let` para reducir el costo de algunos tipos de coincidencia de patrones.

Por ejemplo, digamos que tenemos algún tipo de `Option<T>`. Queremos llamar a una función sobre ello en caso de ser un `Some<T>`, pero no hacer nada si es `None`. Seria algo así:

```rust
# let option = Some(5);
# fn foo(x: i32) { }
match option {
    Some(x) => { foo(x) },
    None => {},
}
```

No tenemos que usar `match` aquí, por ejemplo, podríamos usar `if`:

```rust
# let option = Some(5);
# fn foo(x: i32) { }
if option.is_some() {
    let x = option.unwrap();
    foo(x);
}
```

Ninguna de esas dos opciones es particularmente atractiva. Podemos usar `if let` para hacer lo mismo, pero de mejor forma:


```rust
# let option = Some(5);
# fn foo(x: i32) { }
if let Some(x) = option {
    foo(x);
}
```

Si un [patrón][patterns] concuerda satisfactoriamente, asocia cualquier parte apropiada del valor a los identificadores en el patrón, para luego evaluar la expresión. Si el patrón no concuerda, no pasa nada.

Si quisieras hacer algo en el caso de que el patrón no concuerde, puedes usar `else`:

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

De manera similar, `while let` puede ser usado cuando desees iterar condicionalmente mientras el valor concuerde con cierto patrón. Convierte código como este:

```rust
# let option: Option<i32> = None;
loop {
    match option {
        Some(x) => println!("{}", x),
        _ => break,
    }
}
```

En código como este:

```rust
# let option: Option<i32> = None;
while let Some(x) = option {
    println!("{}", x);
}
```

[patterns]: patterns.html
