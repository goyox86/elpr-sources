% Objetos Trait

Cuando el código involucra polimorfismo, es necesario un mecanismo para determinar que versión especifica debe ser ejecutada. Dicho mecanismo es denominado `despacho`. Hay dos formas mayores de despacho: despacho estático y despacho dinámico. Si bien es cierto que Rust prefiere el despacho estático, también soporta despacho dinámico a través de un mecanismo llamado ‘objetos trait’.

## Bases

Por el resto de este capitulo, necesitaremos un trait y algunas implementaciones. Creemos uno simple, `Foo`. `Foo` posee un solo método que retorna un `String`.

```rust
trait Foo {
    fn metodo(&self) -> String;
}
```

También implementaremos este trait para `u8` y `String`:

```rust
# trait Foo { fn metodo(&self) -> String; }
impl Foo for u8 {
    fn metodo(&self) -> String { format!("u8: {}", *self) }
}

impl Foo for String {
    fn metodo(&self) -> String { format!("string: {}", *self) }
}
```

## Despacho estático

Podemos usar el trait para efectuar despacho estático mediante el uso de limites de trait:

```rust
# trait Foo { fn metodo(&self) -> String; }
# impl Foo for u8 { fn metodo(&self) -> String { format!("u8: {}", *self) } }
# impl Foo for String { fn metodo(&self) -> String { format!("string: {}", *self) } }
fn hacer_algo<T: Foo>(x: T) {
    x.metodo();
}

fn main() {
    let x = 5u8;
    let y = "Hola".to_string();

    hacer_algo(x);
    hacer_algo(y);
}
```

Rust utiliza ‘monomorfizacion’ para el despacho estático en este código. Lo que significa que creara una versión especial de `hacer_algo()` para ambos `u8` y `String`, reemplazando luego los lugares de llamada con invocaciones a estas funciones especializadas. En otras palabras, Rust genera algo así:

```rust
# trait Foo { fn metodo(&self) -> String; }
# impl Foo for u8 { fn metodo(&self) -> String { format!("u8: {}", *self) } }
# impl Foo for String { fn metodo(&self) -> String { format!("string: {}", *self) } }
fn hacer_algo_u8(x: u8) {
    x.metodo();
}

fn hacer_algo_string(x: String) {
    x.metodo();
}

fn main() {
    let x = 5u8;
    let y = "Hola".to_string();

    hacer_algo_u8(x);
    hacer_algo_string(y);
}
```

Lo anterior posee una gran ventaja: el despacho estático permite que las llamadas a función puedan ser insertadas en linea debido a que el receptor es conocido en tiempo de compilación, y la inserción en linea es clave para una buena optimizacion. El despacho estático es rápido, pero viene con una desventaja: ‘código inflado’ (‘code bloat’), a consecuencia de las repetidas copias de la misma función que son insertadas en el binario, una para cada tipo.

Ademas, los compiladores no son perfectos y pueden “optimizar” código haciendolo mas lento. Por ejemplo, funciones insertadas en linea de manera ansiosa inflaran la cache de instrucciones (y el cache gobierna todo a nuestro alrededor). Es por ello que `#[inline]` y `#[inline(always)]` deben ser usadas con cautela, y una razón por la cual usar despacho dinámico es algunas veces mas eficiente.

Sin embargo, el caso común es que el despacho estático sea mas eficiente. Uno puede tener una delgada función envoltorio despachada de manera estática efectuando despacho dinámico, pero no vice versa, es decir; las llamadas estáticas son mas flexibles. Es por esto que la biblioteca estándar intenta ser despachada dinámicamente siempre y cuando sea posible.

## Despacho dinámico

Rust proporciona despacho dinámico a través de una facilidad denominada ‘objetos trait’. Los objetos trait, como `&Foo` o `Box<Foo>`, son valores normales que almacenan un valor de *cualquier* tipo que implementa el trait determinado, en donde el tipo preciso solo puede ser determinado en tiempo de ejecución.

Un objeto trait puede ser obtenido de un apuntador a un tipo concreto que implemente el trait *convirtiéndolo* (e.j. `&x as &Foo`) o aplicándole *coercion* (e.j. usando `&x` como un argumento a una función que recibe `&Foo`).

Esas coerciones y conversiones también funcionan para apuntadores como `&mut T` a `&mut Foo` y `Box<T>` a `Box<Foo>`, pero eso es todo hasta el momento. Las coerciones y conversiones son idénticas.

Esta operación puede ser vista como el ‘borrado’ del conocimiento del compilador acerca del tipo especifico del apuntador, y es por ello que los objetos traits son a veces referidos como `borrado de tipos`.

Volviendo a el ejemplo anterior, podemos usar el mismo trait para llevar a cabo despacho dinámico con conversión de objetos trait:

```rust
# trait Foo { fn metodo(&self) -> String; }
# impl Foo for u8 { fn metodo(&self) -> String { format!("u8: {}", *self) } }
# impl Foo for String { fn metodo(&self) -> String { format!("string: {}", *self) } }

fn hacer_algo(x: &Foo) {
    x.metodo();
}

fn main() {
    let x = 5u8;
    hacer_algo(&x as &Foo);
}
```

o a traves de la coercion:

```rust
# trait Foo { fn metodo(&self) -> String; }
# impl Foo for u8 { fn metodo(&self) -> String { format!("u8: {}", *self) } }
# impl Foo for String { fn metodo(&self) -> String { format!("string: {}", *self) } }

fn hacer_algo(x: &Foo) {
    x.method();
}

fn main() {
    let x = "Hola".to_string();
    hacer_algo(&x);
}
```

Una función que recibe un objeto trait no es especializada para cada uno de los tipos que implementan `Foo`: solo una copia es generada, resultando algunas veces (pero no siempre) en menos inflación de código. Sin embargo, el despacho dinámico viene con el costo de requerir las llamadas mas lentas a funciones virtuales, efectivamente inhibiendo cualquier posibilidad de inserción en linea y las optimizaciones relacionadas.

### Porque apuntadores?

Rust, a diferencia de muchos lenguajes administrados, no coloca cosas detrás de apuntadores por defecto, lo que se traduce en que los tipos tengan diferentes tamaños. Conocer el tamaño de un valor en tiempo de compilación es importante para cosas como: pasarlo como argumento a una función, moverlo en la pila y asignarle (y deasignarle) espacio en el montículo para su almacenamiento.

Para `Foo`, necesitaríamos tener un valor que podría ser mas pequeño que un `String` (24 bytes) o un `u8` (1 byte), así como cualquier otro tipo que pueda implementar `Foo` en crates dependientes (cualquier numero de bytes). No hay forma de garantizar que este ultimo caso pueda funcionar si los valores no son almacenados en un apuntador, puesto que esos otros tipos pueden ser de tamaño arbitrario.

Colocar el valor detrás de un apuntador significa que el tamaño del valor no es relevante cuando estemos lanzando un objeto trait por los alrededores, solo el tamaño del apuntador en si mismo.

### Representación

Los métodos del trait pueden ser llamados en un objeto trait a través de un registro de apuntadores a función tradicionalmente llamado ‘vtable’ (creado y administrado por el compilador).

Los objetos trait son simples y complejos al mismo tiempo: su representation y distribución es bastante directa, pero existen algunos mensajes de error un poco raros y algunos comportamientos sorpresivos por descubrir.

Comencemos por lo mas simple, la representación de un objeto trait en tiempo de ejecución. El modulo `std::raw` contiene structs con distribuciones que son igual de complicadas que las de los tipos integrados, [incluyendo los objetos trait][stdraw]:

```rust
# mod foo {
pub struct TraitObject {
    pub data: *mut (),
    pub vtable: *mut (),
}
# }
```

[stdraw]: ../std/raw/struct.TraitObject.html

Eso es, un trait object como `&Foo` consiste de un apuntador ‘data’ y un apuntador ‘vtable’.

El apuntador data apunta hacia los datos (de un tipo desconocido `T`) que guarda el objeto trait, y el vtable pointer apunta a la vtable (table de metodos virtuales’) (‘virtual method table’) correspondiente a la implementación de `Foo` para `T`.

Una vtable es esencialmente una struct de apuntadores a función, apuntando al segmento concreto de código maquina para cada implementación de metodo. Una llamada a metodo como `objeto_trait.metodo()` retornara el apuntador correcto desde la vtable y luego hará una llamada dinámica de este. Por ejemplo:

```rust,ignore
struct FooVtable {
    destructor: fn(*mut ()),
    tamano: usize,
    alineacion: usize,
    metodo: fn(*const ()) -> String,
}

// u8:

fn llamar_metodo_en_u8(x: *const ()) -> String {
    // el compilador garantiza que esta función solo sea llamada
    // con `x` apuntando a un u8
    let byte: &u8 = unsafe { &*(x as *const u8) };

    byte.metodo()
}

static Foo_vtable_para_u8: FooVtable = FooVtable {
    destructor: /* magia del compilador */,
    tamano: 1,
    alineacion: 1,

    // conversion a a un apuntador a función
    metodo: llamar_metodo_en_u8 as fn(*const ()) -> String,
};


// String:

fn llamar_metodo_en_String(x: *const ()) -> String {
    // el compilador garantiza que esta función solo sea llamada
    // con `x` apuntando a un String
    let string: &String = unsafe { &*(x as *const String) };

    string.metodo()
}

static Foo_vtable_para_String: FooVtable = FooVtable {
    destructor: /* magia del compilador */,
    // valores para una computadora de 64 bits, dividelos por la mitad para una de 32
    tamano: 24,
    alineacion: 8,

    metodo: llamar_metodo_en_String as fn(*const ()) -> String,
};
```

El campo `destructor` en cada vtable apunta a una función que limpiara todos los recursos del tipo de la vtable: para `u8` es trivial, pero para `String` liberara memoria. Esto es necesario para adueñarse de objetos trait como `Box<Foo>`, que necesitan limpiar ambos la asignación `Box` así como el tipo interno cuando salgan de ámbito. Los campos `tamano` y `alineacion` almacenan el tamaño del tipo borrado, y sus requerimientos de alineación; estos son esencialmente no usados por el momento puesto que la información es embebida en el destructor, pero sera usada en el futuro, a medida que los objetos trait sean progresivamente hechos mas flexibles.

Supongamos que tenemos algunos valores que implementen `Foo`. La forma explicita de construcción y uso de objetos trait `Foo` puede lucir un poco como (ignorando las incongruencias entre tipos: son apuntadores de cualquier manera):

```rust,ignore
let a: String = "foo".to_string();
let x: u8 = 1;

// let b: &Foo = &a;
let b = TraitObject {
    // almacena los datos
    data: &a,
    // almacena los metodos
    vtable: &Foo_vtable_para_String
};

// let y: &Foo = x;
let y = TraitObject {
    // almacena los datos
    data: &x,
    // almacena los metodos
    vtable: &Foo_vtable_para_u8
};

// b.metodo();
(b.vtable.metodo)(b.data);

// y.metodo();
(y.vtable.metodo)(y.data);
```

## Seguridad de Objetos

No todo trait puede ser usado para crear un objeto trait. Por ejemplo, los vectores implementan `Clone`, pero si intentamos crear un objeto trait:

```ignore
let v = vec![1, 2, 3];
let o = &v as &Clone;
```

Obtenemos un error:

```text
error: cannot convert to a trait object because trait `core::clone::Clone` is not object-safe [E0038]
let o = &v as &Clone;
        ^~
note: the trait cannot require that `Self : Sized`
let o = &v as &Clone;
        ^~
```

El error dice que `Clone` no es ‘seguro para objetos’ (‘object-safe’). Solo los traits que son seguros para objetos pueden ser usados en la creación de objetos trait. Un trait es seguro para objetos si ambas condiciones son verdaderas:

* el trait no requiere que `Self: Sized`
* todos sus métodos son seguros para objetos

Entonces, que hace a un metodo seguro para objetos? Cada metodo debe requerir que `Self: Sized` o todas de las siguientes:

* no debe tener ninguna parámetro de tipo
* no debe usar `Self`

Uff! Como podemos ver, casi todas estas reglas hablan acerca de `Self`. Una buena intuición seria “exceptuando circunstancias especiales, si tu metodo de trait usa `Self`, no es seguro para objetos.”
