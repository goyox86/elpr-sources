% Sintaxis de Metodos

Las funciones son geniales, pero si deseas llamar bastantes en alguna data, puede tornarse incomodo. Considera este código:

```rust,ignore
baz(bar(foo));
```

Pudiéramos leer esto de izquierda a derecha, veríamos entonces ‘baz bar foo’. Pero ese no es el orden en el cual las funciones son llamadas, el orden de hecho, es de adentro hacia afuera: ‘foo bar baz’. No seria excelente si pudiéramos hacerlo:

```rust,ignore
foo.bar().baz();
```

Por suerte, como habrás podido deducir de la pregunta anterior, por supuesto que podemos! Rust provee la habilidad de usar la denominada ‘sintaxis de llamada a métodos’ (‘method call syntax’) a través de la palabra reservada  `impl`.

# Llamadas a metodos

Así funcionan:

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

Hemos construido una `struct` representando a un circulo. Después escribimos un bloque `impl` y dentro de el, definimos un método, `area`.

Los métodos reciben un primer parámetro especial, del cual existen tres variantes: `self`, `&self`, and `&mut self`. Puedes pensar acerca de este primer parámetro como el `foo` en `foo.bar()`. Las tres variantes corresponden a los tres tipos de cosas que `foo` podría ser: `self` si es solo un valor en la pila, `&self` si es una referencia y `&mut self` si es una referencia mutable. Debido a que proporcionamos el parámetro `&self` a `area`, podemos usarlo justo como cualquier otro. Como conocemos que es un `Circulo`, podemos acceder a `radio` como lo haríamos con cualquier otra `struct`.

Deberíamos usar por defecto `&self`, así como preferir también el préstamo por encima de la toma de pertenencia y recibir referencias inmutables en vez de mutables, en lo posible. He aquí un ejemplo de las tres variantes:

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

Entonces, ahora sabemos como llamar a un método, como `foo.bar()`. Pero que hay acerca de nuestro ejemplo original, `foo.bar().baz()`? Se denomina ‘encadenamiento de métodos’ (‘method chaining’). Veamos un ejemplo:

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
        Circulo { x: self.x, y: self.y, radio: self.radio + incremento }
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

Solo decimos que retornamos un `Circulo`. Con este método, podemos agrandar un nuevo `Circulo` a un tamaño arbitrario.

# Funciones asociadas

Puedes también definir funciones asociadas que no tomen el parámetro `self`. He aquí un patron muy común en código Rust:

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

Esta ‘función asociada’ construye un nuevo `Circulo`. Nota que las funciones asociadas son llamadas con la sintaxis `Struct::funcion()`, en lugar de `ref.metodo()`. Otros lenguajes llaman a las funciones asociadas ‘métodos estáticos’

# El patron Constructor (Builder)

Digamos que queremos que nuestros usuarios puedan crear `Circulo`s, pero solo les permitiremos dar valores a las propiedades que sean relevantes para ellos. De lo contrario, los atributos `x` y `y` serán `0.0`, y el radio sera `1.0`. Rust no posee sobrecarga de métodos, argumentos con nombre o argumentos variables. Se emplea el patron builder en su lugar. Dicho patron luce asi:

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

Lo que hemos hecho es crear otra `struct`, `ConstructorCirculo`. Hemos definido nuestros métodos constructores en ella. También hemos definido nuestro método `area()` en `Circulo`. Hemos agregado otro método en `ConstructorCirculo`: `finalizar()`. Este método crea nuestro `Circulo` desde el constructor. Ahora, hemos usado el sistema de tipos para hacer valer nuestros intereses: podemos usar los métodos en `ConstructorCirculo` para crear `Circulo`s en la forma que decidamos.
