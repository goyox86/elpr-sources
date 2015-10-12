% Objetos Trait

Cuando el codigo involucra polimorfismo, es necesario un mecanismo para determinar cual version especifica debe ser ejecutada. Dicho mecanismo es denominado `despacho`. Hay dos formas mayores de despacho: despacho estatico y despacho dinamico. Si bien es cierto que Rust prefiere el despacho estatico, tambien soporta despacho dinamico a traves de un mecanismo llamado ‘objetos trait’.

## Bases

Por el resto de este capitulo, necesitaremos un trait y algunas implementaciones. Creemos uno simple, `Foo`. `Foo` posee un solo metodo que retorna un `String`.

```rust
trait Foo {
    fn metodo(&self) -> String;
}
```

Tambien implementaremos este trait para `u8` y `String`:

```rust
# trait Foo { fn metodo(&self) -> String; }
impl Foo for u8 {
    fn metodo(&self) -> String { format!("u8: {}", *self) }
}

impl Foo for String {
    fn metodo(&self) -> String { format!("string: {}", *self) }
}
```

## Despacho estatico

Podemos usar el trait para llevar a cabo despacho estatico con limites de trait:

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

Rust utiliza ‘monomorfizacion’ para efectuar despacho estatico en este codigo. Lo que significa que creara una version especial de `hacer_algo()` para ambos `u8` y `String`, reemplazando luego los lugares de llamada con invocaciones a dichas funciones especializadas. En otras palabras, Rust genera algo como esto:

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

Lo anterior posee ua gran ventaja: el despacho estatico permite que las llamadas a funcion puedan ser insertadas en linea debido a que el receptor es conocido en tiempo de compilacion, y la insercion en linea es clave para una buena optimizacion. El despacho estatico es rapido, pero viene con una desventaja: ‘codigo inflado’ (‘code bloat’), a consecuencia de las repetidas copias de la misma funcion en el binario, una para cada tipo.

Ademas, los compiladores no son perfectos y pueden “optimizar” codigo haciendolo mas lento. Por ejemplo, funciones insertadas en linea de manera ansiosa inflaran la cache de instrucciones (y el cache gobierna todo a nuestro alrededor). Es por ello que `#[inline]` y `#[inline(always)]` deben ser usadas con cautela, y una razon por la cual usar despacho dinamico es algunas veces mas eficiente.

Sin embargo, el caso comun es que el despacho estatico es el mas eficiente, y uno puede tener una delgada funcion envoltorio despachada estaticamente que efectuando despacho dinamico, pero no vice versa, es decir; las llamadas estaticas
son mas flexibles. Es por esto que la biblioteca estandar intenta ser despachada dinamicamente en donde sea posible.

## Despacho dinamico

Rust proporciona despacho dinamico a traves de una facilidad denominada ‘objetos trait’. Los objetos trait, como `&Foo` o `Box<Foo>`, son valores normales que almacenan un valor de *cualquier* tipo que implementa el trait determinado, en donde el tipo preciso solo puede ser determinado en tiempo de ejecucion.

Un objeto trait puede ser obtenido de un apuntador a un tipo concreto que implemente el trait *convirtiendolo* (e.j. `&x as &Foo`) o aplicandole *coercion* (e.j. usando `&x` como un argumento a una funcion que recibe `&Foo`).

Esas coerciones y conversiones tambien funcionan para apuntadores como `&mut T` a `&mut Foo` y `Box<T>` a `Box<Foo>`, pero eso es todo hasta el momento. Las coerciones y conversiones son identicas.

Esta operacion puede ser vista como el ‘borrado’ del conocimiento del compilador acerca del tipo especifico del apuntador, y es por ello que los objetos trais son a veces referidos como `borrado de tipos`.

Volviendo a el ejemplo anterior, podemos usar el mismo trait para llevar a cabo despacho dinamico con conversion de objetos trait:

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

Una funcion que recibe un objeto trait no es especializada para cada uno de los tipos que implementan `Foo`: solo una copia es generada, resultando algunas veces (pero no siempre) en menos inflado de codigo. Sin embargo, el despacho dinamico viene con el costo de requerir llamadas mas lentas a funciones virtuales, y efectivamente inhibiendo cualquier posibilidad de la insercion en linea y las optimizaciones relacionadas.

### Porque apuntadores?

Rust no coloca cosas detras de apuntadores por defecto, a diferencia de muchos lenguajes administrados, lo que se traduce en que los tipos tengan diferentes tamanos. Conocer el tamano de un valor en tiempo de compilacion es importante para cosas como pasarlo como argumento a una funcion, moverlo en la pila y asignarle (y deasignarle) espacio en el monticulo para su almacenamiento.

Para `Foo`, necesitariamos tener un valor que podria ser mas pequeno que un `String` (24 bytes) o un `u8` (1 byte), asi como cualquier otro tipo que pueda implementar `Foo` en crates dependientes (cualquier numero de bytes). No hay forma de garantizar que este ultimo punto pueda funcionar si los valores son almacenados sin un apuntador, puesto que esos otros tipos pueden ser arbritrariamente grandes.

Colocar el valor detras de un apuntador significa que el tamano del valor no es relevante cuando estemos lanzando un objeto trait por los alrededores, solo el tamano del apuntador en si mismo.

### Representacion

Los metodos del trait pueden ser llamados en un objeto trait a traves de un registro de apuntadores a funcion tradicionalmente llamado ‘vtable’ (creado y administrado por el compilador).

Los objetos trait son simples y complicados al mismo tiempo: su representation y distribucion es bastante directa, pero existen algunos mensajes de error un poco raros y algunos comportamientos sopresivos por descubrir.

Comencemos por lo mas simple, la representacion de un objeto trait en tiempo de ejecucion. El modulo `std::raw` contiene structs con distrubiciones que son igual de complicadas que las de los tipos integrados, [incluyendo los objetos trait][stdraw]:

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

El apuntador data apunta hacia la data (de un tipo desconocido `T`) que guarda el objeto trait, y el vtable pointer apunta a la vtable (table de metodos virtuales’) (‘virtual method table’) correspondiente a la implementacion de `Foo` para `T`

Una vtable es esencialmente una struct de apuntadores a funcion, apuntando al segmento concreto de codigo maquina para cada implementacion de metodo. Una llamada a metodo como `objeto_trait.metodo()` retornara el apuntador correcto desde la vtable y luego hara una llamada dinamica de este. Por ejemplo:

```rust,ignore
struct FooVtable {
    destructor: fn(*mut ()),
    tamano: usize,
    alineacion: usize,
    metodo: fn(*const ()) -> String,
}

// u8:

fn llamar_metodo_en_u8(x: *const ()) -> String {
    // el compilador garantiza que esta funcion solo sea llamada
    // con `x` apuntando a un u8
    let byte: &u8 = unsafe { &*(x as *const u8) };

    byte.metodo()
}

static Foo_para_vtable_u8: FooVtable = FooVtable {
    destructor: /* magia del compilador */,
    tamano: 1,
    alineacion: 1,

    // conversion a a un apuntador a funcion
    metodo: llamar_metodo_en_u8 as fn(*const ()) -> String,
};


// String:

fn llamar_metodo_en_String(x: *const ()) -> String {
    // el compilador garantiza que esta funcion solo sea llamada
    // con `x` apuntando a un String
    let string: &String = unsafe { &*(x as *const String) };

    string.metodo()
}

static Foo_para_vtable_String: FooVtable = FooVtable {
    destructor: /* magia del compilador */,
    // valores para una computadora de 64 bits, dividelos por la mitad para una de 32
    tamano: 24,
    alineacion: 8,

    metodo: llamar_metodo_en_String as fn(*const ()) -> String,
};
```

El campo `destructor` en cada vtable apunta a una funcion que limipiara todos los recursos del tipo de la vtable: para `u8` es trivial, pero para `String` liberara memoria. Esto es necesario para aduenarse de objetos trait como `Box<Foo>`, que necesitan limpiar ambos la allocacion `Box` asi como el tipo interno cuando salgan de ambito. Los campos `tamano` y `alineacion` almacenan el tamano del tipo borrado, y sus requerimientos de alineacion; estos son esencialmente no usados por el momento puesto que la informacion es embebida en el destructor, pero sera usada en el futuro, a medida que los objetos trait son hechos progresivamente mas flexibles.

Supongase que tenemos algunos valores que implementen `Foo`. La forma explicita de construccion y uso de objetos trait `Foo` puede lucir un poco como (ignorando las incongruencias entre tipos: son apuntadores de cualquier manera):

```rust,ignore
let a: String = "foo".to_string();
let x: u8 = 1;

// let b: &Foo = &a;
let b = TraitObject {
    // almacena los datos
    data: &a,
    // almacena los metodos
    vtable: &Foo_para_vtable_String
};

// let y: &Foo = x;
let y = TraitObject {
    // almacena los datos
    data: &x,
    // almacena los metodos
    vtable: &Foo_para_vtable_u8
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

El error dice que `Clone` no es ‘seguro para objetos’ (‘object-safe’). Solo los traits que son seguros para objetos pueden ser usados en la creacion de objetos trait. Un trait es seguro para objetos si ambas condiciones son verdaderas:

* el trait no requiere que `Self: Sized`
* toddos sus metodos son seguros para objetos

Entonces, que hace a un metodo seguro para objetos? Cada metodo debe requerir que `Self: Sized` o todas de las siguientes:

* must not have any type parameters
* must not use `Self`

Uff! Como podemos ver, casi todas estas reglas hablan acerca de `Self`. Una buena intuicion es “exceptuando circunstancias especiales, si tu metodo de trait usa `Self`, no es seguro para objetos.”
