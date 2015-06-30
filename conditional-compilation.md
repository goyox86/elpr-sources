% Compilación Condicional

Rust posee un atributo especial, `#[cfg]`, que te permite compilar código basado en una opción proporcionada al compilador. Tiene dos formas:

```rust
#[cfg(foo)]
# fn foo() {}

#[cfg(bar = "baz")]
# fn bar() {}
```

También posee algunos helpers:

```rust
#[cfg(any(unix, windows))]
# fn foo() {}

#[cfg(all(unix, target_pointer_width = "32"))]
# fn bar() {}

#[cfg(not(foo))]
# fn not_foo() {}
```

Los cuales pueden ser anidados de manera arbitraria:

```rust
#[cfg(any(not(unix), all(target_os="macos", target_arch = "powerpc")))]
# fn foo() {}
```

Para activar o desactivar estos switches, si estas usando Cargo, se configuran en la [sección `[features]`][features] de tu `Cargo.toml`

[features]: http://doc.crates.io/manifest.html#the-%5Bfeatures%5D-section

```toml
[features]
# No features por defecto
default = []

# El feature “secure-password” depende en el paquete bcrypt.
secure-password = ["bcrypt"]
```

Cuando hacemos esto, Cargo pasa una opción a `rustc`:

```text
--cfg feature="${feature_name}"
```

La suma de esas opciones `cfg` determinara cuales son activadas, y en consecuencia, cual código sera compilado. Tomemos este código:

```rust
#[cfg(feature = "foo")]
mod foo {
}
```

Si lo compilamos con `cargo build --features "foo"`, Cargo enviara la opción `--cfg feature="foo"` a `rustc`, y la salida tendrára el `mod foo`. Si compilamos con un `cargo build` normal, ninguna opción extra sera proporcionada, y debido a esto, ningún modulo `foo` existirá.

# cfg_attr

También puedes configurar otro atributo basado en una variable `cfg` con `cfg_attr`:

```rust
#[cfg_attr(a, b)]
# fn foo() {}
```

Sera lo mismo que `#[b]` si `a` es configurado por un atributo `cfg`, y nada de cualquier otra manera.

# cfg!

La [extension de sintaxis][compilerplugins] te permite también usar este tipo de opciones en cualquier otro punto de tu código:

```rust
if cfg!(target_os = "macos") || cfg!(target_os = "ios") {
    println!("Think Different!");
}
```

[compilerplugins]: compiler-plugins.html

Estos serán reemplazado por `true` o `false` en tiempo de compilación, dependiendo en las opciones de la configuración.
