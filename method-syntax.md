% Sintaxis de Metodos

Las funciones son geniales, pero si deseas llamar bastantes en alguna data, puede tornarse incomodo. Considera este codigo:

```rust,ignore
baz(bar(foo));
```

Pudieramos leer esto de izquierda a derecha, veriamos entonces ‘baz bar foo’. Pero ese no es el orden en el cual las funciones son llamadas, el orden de hecho, es de adentro hacia afuera: ‘foo bar baz’. No seria excelente si pudieramos hacerlo:

```rust,ignore
foo.bar().baz();
```

Por suerte, como habras podido deducir de la pregunta anteror, por supuesto que podemos! Rust provee la habilidad de usasr la denominada ‘sintaxis de llamada a metodos’ (‘method call syntax’) a traves de la palabra reservada  `impl`.

# Llamadas a metodos

Asi funcionan:

```rust
struct Circulo {
    x: f64,
    y: f64,
    radio: f64,
}

impl Circulo {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radio * self.radio)
    }
}

fn main() {
    let c = Circulo { x: 0.0, y: 0.0, radio: 2.0 };
    println!("{}", c.area());
}
```

Lo anterior imprimira `12.566371`.


Hemos construido una `struct` representando a un circulo. Despues escribimos un bloque `impl` y dentro de el, definimos un metodo, `area`.

Los metodos reciben un primer parametro especial, de el cual existen tres variantes: `self`, `&self`, and `&mut self`. Puedes pensar acerca de este primer parametro como el `foo` en `foo.bar()`. Las tres variantes corresponden a los tres tipos de cosas que `foo` podria ser:  `self` si es solo un valor en la pila, `&self` si es una referencia y and `&mut self` si es una referencia mutable. Debido a que proporcionamos el parametro `&self` a `area`, podemos usarlos justo como cualquier otro parametro. Como conocemos que es un `Circulo`, poedmos acceder a  `radio` como lo hariamos con cualquier otra `struct`.

Deberiamos usar por defecto `&self`, y deberias preferir el prestamo por encima de la toma de pertenencia, asi como recibir referencias inmutables en vez de mutables. He aqui un ejemplo de las tres variantes:

```rust
struct Circulo {
    x: f64,
    y: f64,
    radio: f64,
}

impl Circle {
    fn referencia(&self) {
       println!("recibiendo a self como una referencia!");
    }

    fn referencia_mutable(&mut self) {
       println!("trecibiendo a self como una referencia mutable!");
    }

    fn toma_pertenecia(self) {
       println!("tomando pertenencia de self!");
    }
}
```

# Llamadas a metodo en cadena

Entonces, ahora sabemos como llamar a un metodo, como `foo.bar()`. Pero que hay acerca de nuestro ejemplo original, `foo.bar().baz()`? Esto se denomina ‘encadenamiento de metodos’ (‘method chaining’). Veamos un ejemplo:

```rust
struct Circulo {
    x: f64,
    y: f64,
    radio: f64,
}

impl Circulo {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radio * self.radio)
    }

    fn agrandar(&self, incremento: f64) -> Circulo {
        Circle { x: self.x, y: self.y, radio: self.radio + incremento }
    }
}

fn main() {
    let c = Circulo { x: 0.0, y: 0.0, radio: 2.0 };
    println!("{}", c.area());

    let d = c.agrandar(2.0).area();
    println!("{}", d);
}
```

Observa el valor de retorno:

```rust
# struct Circulo;
# impl Circulo {
fn agrandar(&self, incremento: f64) -> Circulo {
# Circulo } }
```

Solo decimos que retornamos un `Circulo`. Con este metodo, podemos agrandar un nuevo `Circulo` a un tamano arbitrario.

# Funciones asociadas

Puedes tambien definir funciones asociadas que no tomen un parametro `self`. He aqui el patron muy comun en codigo Rust:

```rust
struct Circulo {
    x: f64,
    y: f64,
    radio: f64,
}

impl Circulo {
    fn new(x: f64, y: f64, radio: f64) -> Circulo {
        Circulo {
            x: x,
            y: y,
            radio: radio,
        }
    }
}

fn main() {
    let c = Circulo::new(0.0, 0.0, 2.0);
}
```

Esta ‘funcion asociada’ construye un nuevo `Circulo` para nosotros. Nota que las funciones asociadas son llamadas con la sintaxis `Struct::funcion()`, en lugar de `ref.method()`. Algunos otros lenguajes llaman a las funciones asociadas ‘metodos estaticos’

# El patron Constructor (Builder)

Digamos que queremos que nuestrios usuarios puedan crear `Circulo`s, pero solo les permitiremos dar valores a las propiedades que sean relevantes para ellos. De lo contrario, los atributos `x` y `y` seran `0.0`, y el radio sera `1.0`. Rust no posee sobracarga de metodos, argumentos con nombre, o argumentos variables. Empleamos el patron builder en su  lugar. Dicho patron luce asi:

```rust
struct Circulo {
    x: f64,
    y: f64,
    radio: f64,
}

impl Circulo {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radio * self.radio)
    }
}

struct ConstructorCirculo {
    x: f64,
    y: f64,
    radio: f64,
}

impl ConstructorCirculo {
    fn new() -> ConstructorCirculo {
        ConstructorCirculo { x: 0.0, y: 0.0, radio: 1.0, }
    }

    fn x(&mut self, coordenada: f64) -> &mut ConstructorCirculo {
        self.x = coordenada;
        self
    }

    fn y(&mut self, coordenada: f64) -> &mut ConstructorCirculo {
        self.y = coordenada;
        self
    }

    fn radio(&mut self, radio: f64) -> &mut ConstructorCirculo {
        self.radio = radio;
        self
    }

    fn finalizar(&self) -> Circulo {
        Circulo { x: self.x, y: self.y, radio: self.radio }
    }
}

fn main() {
    let c = ConstructorCirculo::new()
                .x(1.0)
                .y(2.0)
                .radio(2.0)
                .finalizar();

    println!("area: {}", c.area());
    println!("x: {}", c.x);
    println!("y: {}", c.y);
}
```

Lo que hemos hecho es crear otra `struct`, `ConstructorCirculo`. Hemos definido nuestros metodos builder en ella. Tambien hemos definido nuestrio metodo `area()` en `Circulo`. Hemos anadido otro metodo en `ConstructorCirculo`: `finalizar()`. Este metodo crea nuestro `Circulo` desde el builder. Ahora, hemos usado el sistema de tipos para aplicar nuestros intereses: podemos usar los metodos en `ConstructorCirculo` para crear `Circulo`s en la forma que decidamos.
