## Filósofos Cenando

Para nuestro segundo proyecto, echemos un vistazo a un problema clásico de concurrencia. Se llama ‘La cena de los filosofos’. Fue originalmente concebido por Dijkstra en 1965, pero nosotros usaremos una version ligeramente dadaptada de [este paper][paper] por Tony Hoare en 1985. 

[paper]: http://www.usingcsp.com/cspbook.pdf

> En tiempos ancestrales, un filántropo adinerado preparo una universidad para alojar a cinco 
> filósofos eminentes. Cada filosofo tenia una habitación en la cual podía desempeñar su 
> actividad profesional del pensamiento: también había un comedor en común, amoblado con una
> mesa circular, rodeada por cinco sillas, cada una identificada con el nombre del filosofo que > se sentaba en ella. Los filósofos sentaban en sentido anti-horario alrededor de la mesa. A la > izquierda de cada filosofo yacía un tenedor dorado, y en el medio un tazón de espagueti, el 
> cual era constantemente reabastecido. Se esperaba que un filosofo empleara la mayoría de su 
> tiempo pensando; pero cuando se sintieran con hambre, ese dirigiera a el comedor tomara el tenedor que estaba a su izquierda y lo sumieran en el espagueti. Pero tal era naturaleza enredada del espagueti que un segundo tenedor era requerido para llevarlo a la boca. El filosofo por ende tenia que también tomar el tenedor a su derecha. Cuando terminaban debian bajar ambos tenedores, levantarse de la silla y continuar pensando. Por supuesto, un tenedor puede ser usado por un solo filosofo a la vez. Si otro filosofo lo desea, tiene que esperar hasta que el tenedor este disponible nuevamente.

Este problema clásico exhibe algunos elementos de la concurrencia. La razón es que es efectivamente un poco difícil de implementar: una implementación simple puede un deadlock. Por ejemplo, consideremos un algoritmo simple que resolvería este problema:

This classic problem shows off a few different elements of concurrency. The
reason is that it's actually slightly tricky to implement: a simple
implementation can deadlock. For example, let's consider a simple algorithm
that would solve this problem:

1. Un filosofo tomo el tenedor a su izquierda.
2. Después toma el tenedor en a su derecha.
3. Come
4. Devuelve los tenedores.

Ahora, imaginemos esta secuencia de eventos:

1. Filosofo 1 comienza el algoritmo, tomando el tenedor a su izquierda.
2. Filosofo 2 comienza el algoritmo, tomando el tenedor a su izquierda.
3. Filosofo 3 comienza el algoritmo, tomando el tenedor a su izquierda.
4. Filosofo 4 comienza el algoritmo, tomando el tenedor a su izquierda.
5. Filosofo 5 comienza el algoritmo, tomando el tenedor a su izquierda.
6. ... ? Todos los tenedores han sido tomados, pero nadie puede comer!

Existen diferentes formas de resolver este problema. Te guiaremos a   de la solución de este tutorial. Por ahora, comencemos modelando el problema en si mismo. Comenzaremos con los filósofos:

```rust
struct Filosofo {
    nombre: String,
}

impl Filosofo {
    fn new(nombre: &str) -> Filosofo {
        Filosofo {
            nombre: nombre.to_string(),
        }
    }
}

fn main() {
    let f1 = Filosofo::new("Judith Butler");
    let f2 = Filosofo::new("Gilles Deleuze");
    let f3 = Filosofo::new("Karl Marx");
    let f4 = Filosofo::new("Emma Goldman");
    let f5 = Filosofo::new("Michel Foucault");
}
```

Acá, creamos una [`estructura`][struct] (struct) para representar un filosofo. Por ahora el nombre es todo lo que necesitamos. Escojimos el tipo [`String`][string] para el nombre, en vez de `&str`. Generalmente hablando, trabajar con tipo que es dueño de su data es mas fácil que trabajar con uno que use referencias.

[struct]: structs.html
[string]: strings.html

Continuemos:

```rust
# struct Filosofo {
#     nombre: String,
# }
impl Filosofo {
    fn new(nombre: &str) -> Filosofo {
        Filosofo {
            nombre: nombre.to_string(),
        }
    }
}
```

Este bloque `impl` nos permite definir cosas en estructuras `Filosofo`. En este caso estamos definiendo una ‘función asociada’ llamada `new`. La primera linea luce asi: 

```rust
# struct Filosofo {
#     nombre: String,
# }
# impl Filosofo {
fn new(nombre: &str) -> Filosofo {
#         Filosofo {
#             nombre: nombre.to_string(),
#         }
#     }
# }
```

Recibimos un argumento, un `nombre`, de tipo `name`. Esto es una referencia a otra cadena de caracteres. Esta retorna ina instancia de nuestra estructura `Filosofo`. 

```rust
# struct Filosofo {
#     nombre: String,
# }
# impl Filosofo {
#    fn new(nombre: &str) -> Filosofo {
Filosofo {
    nombre: nombre.to_string(),
}
#     }
# }
```

Lo anterior crea un nuevo `Filosofo`, y setea su campo `nombre` a nuestro argumento `nombre`. No solo a el argumento en si mismo, debido a que llamamos `.to_string()` en el. Lo cual crea una copia de la cadena a la que apunta nuestro `&str`, y nos da un nuevo `String`, que es del tipo del campo `nombre` de `Filosofo`.

Porque no aceptar un `String` directamente? Es mas fácil de llamar. Si recibiéramos un `String` pero quien nos llama tuviese un `&str` ellos se verían en la obligación de llamar `.to_string()` de su lado. La desventaja de esta flexibilidad es que _siempre_ hacemos una copia. Para este pequeño programa, esto no es particularmente importante, como sabemos, estaremos usando cadenas cortas de cualquier modo.

Una ultima cosas que habrás notado: solo definimos un `Filosofo`, y no parecemos hacer nada con el. Rust es un lenguaje ‘basado en expresiones’, lo que significa que casi cualquier cosa en Rust es una expresión que retorna un valor. Esto es cierto para las funciones también, la ultima expresión es retornada automáticamente. Debido a que creamos un nuevo `Filosofo` como la ultima expresión de esta función, terminamos retornándolo.

Este nombre `new()`, no es nada especial para Rust, pero es una convención para funciones que crean nuevas instancias de estructuras. Antes que hablemos del porque, echamos un vistazo a `main()` otra vez: 


```rust
# struct Filosofo {
#     nombre: String,
# }
#
# impl Filosofo {
#     fn new(nombre: &str) -> Filosofo {
#         Filosofo {
#             nombre: nombre.to_string(),
#         }
#     }
# }
#
fn main() {
    let f1 = Filosofo::new("Judith Butler");
    let f2 = Filosofo::new("Gilles Deleuze");
    let f3 = Filosofo::new("Karl Marx");
    let f4 = Filosofo::new("Emma Goldman");
    let f5 = Filosofo::new("Michel Foucault");
}
```

Aca, creamos cinco variables con cinco nuevos filósofos. Estos son mis cinco favoritos, pero tu puedes substituirlos con quien tu creas. De no haber definido la función `new()` , luciría así:


```rust
# struct Filosofo {
#     nombre: String,
# }
fn main() {
    let f1 = Filosofo { nombre: "Judith Butler".to_string() };
    let f2 = Filosofo { nombre: "Gilles Deleuze".to_string() };
    let f3 = Filosofo { nombre: "Karl Marx".to_string() };
    let f4 = Filosofo { nombre: "Emma Goldman".to_string() };
    let f5 = Filosofo { nombre: "Michel Foucault".to_string() };
}
```

Un poco mas ruidoso. Usar `new` tiene también posee otras ventajas, pero incluso en este simple caso termina por ser de mejor utilidad.

Ahora que tenemos lo básico en su lugar, hay un numero de maneras en las cuales podemos atacar en problema mas amplio. A mi me gusta comenzar por el final: creemos una forma para que cada filosofo pueda finalizar de comer. Como un paso pequeño, hagamos un método, y luego iteremos a través de todos los filósofos llamándolo:


```rust
struct Filosofo {
    nombre: String,
}

impl Filosofo {
    fn new(nombre: &str) -> Filosofo {
        Filosofo {
            nombre: nombre.to_string(),
        }
    }

    fn comer(&self) {
        println!("{} ha finalizado de comer.", self.nombre);
    }
}

fn main() {
    let filosofos = vec![
        Filosofo::new("Judith Butler"),
        Filosofo::new("Gilles Deleuze"),
        Filosofo::new("Karl Marx"),
        Filosofo::new("Emma Goldman"),
        Filosofo::new("Michel Foucault"),
    ];

    for f in &filosofos {
        f.comer();
    }
}
```

Primero veamos a `main()`. En lugar de tener cinco variables individuales para nuestros filósofos, creamos un `Vec<T>`. `Vec<T>` es llamado también un ‘vector’, y es un arreglo capaz de crecer. Después usamos un ciclo [`for`][for] para iterar a través del vector, obteniendo un referencia a cada filosofo a la vez.

En el cuerpo del bucle, llamamos `f.comer();`, que esta definido como:

```rust,ignore
fn comer(&self) {
    println!("{} ha finalizado de comer.", self.nombre);
}
```

En Rust, los métodos reciben un parámetro explícito `self`.  Es por ello que `comer()` es un método pero `new` es una función asociada:  `new()` no tiene `self`. Para nuestra primera version de `comer()`, solo imprimimos el nombre del filósofo, y mencionamos que ha finalizado de comer. Ejecutar este programa deber generar la siguiente salida:

```text
Judith Butler ha finalizado de comer.
Gilles Deleuze ha finalizado de comer.
Karl Marx ha finalizado de comer.
Emma Goldman ha finalizado de comer.
Michel Foucault ha finalizado de comer.
```

Muy fácil, todos han terminado! No hemos implementado el problema real todavía, así que no hemos terminado!

A continuation, no solo queremos solo finalicen de comer, sino que efectivamente coman. He aquí la siguiente versión: 


```rust
use std::thread;

struct Filosofo {
    nombre: String,
}

impl Filosofo {
    fn new(nombre: &str) -> Filosofo {
        Filosofo {
            nombre: nombre.to_string(),
        }
    }

    fn comer(&self) {
        println!("{} esta comiendo.", self.nombre);

        thread::sleep_ms(1000);

        println!("{} ha finalizado de comer.", self.nombre);
    }
}

fn main() {
    let filosofos = vec![
        Filosofo::new("Judith Butler"),
        Filosofo::new("Gilles Deleuze"),
        Filosofo::new("Karl Marx"),
        Filosofo::new("Emma Goldman"),
        Filosofo::new("Michel Foucault"),
    ];

    for f in &filosofos {
        f.comer();
    }
}
```

Solo unos pocos cambios. Analicemoslos parte por parte.

```rust,ignore
use std::thread;
```

`use` hace disponibles nombres en nuestro ámbito (scope). Comenzaremos a usar el modulo `thread` de la biblioteca estándar, y es por ello que necesitamos hacer `use` en el.


```rust,ignore
    fn comer(&self) {
        println!("{} esta comiendo.", self.nombre);

        thread::sleep_ms(1000);

        println!("{} ha finalizado de comer.", self.nombre);
    }
```

Ahora estamos imprimiendo dos mensajes, con un `sleep_ms()` en el medio. Lo cual simulara el tiempo que tarda un filosofo en comer.

Si ejecutas este programa, deberias ver comer a cada filosofo a la vez:

```text
Judith Butler esta comiendo.
Judith Butler ha finalizado de comer.
Gilles Deleuze esta comiendo.
Gilles Deleuze ha finalizado de comer.
Karl Marx esta comiendo.
Karl Marx ha finalizado de comer.
Emma Goldman esta comiendo.
Emma Goldman ha finalizado de comer.
Michel Foucault esta comiendo.
Michel Foucault ha finalizado de comer.
```

Excelente! Estamos avanzando. Solo hay un problema: no estamos operando de manera concurrente, lo cual es parte central de nuestro problema!

Para hacer a nuestros filósofos comer de manera concurrente, necesitamos hacer un pequeño cambio.

He aqui la siguiente iteración:

```rust
use std::thread;

struct Filosofo {
    nombre: String,
}

impl Filosofo {
    fn new(nombre: &str) -> Filosofo {
        Filosofo {
            nombre: nombre.to_string(),
        }
    }

    fn comer(&self) {
        println!("{} esta comiendo.", self.nombre);

        thread::sleep_ms(1000);

        println!("{} ha finalizado de comer.", self.nombre);
    }
}

fn main() {
    let filosofos = vec![
        Filosofo::new("Judith Butler"),
        Filosofo::new("Gilles Deleuze"),
        Filosofo::new("Karl Marx"),
        Filosofo::new("Emma Goldman"),
        Filosofo::new("Michel Foucault"),
    ];

    let handles: Vec<_> = filosofos.into_iter().map(|f| {
        thread::spawn(move || {
            f.comer();
        })
    }).collect();

    for h in handles {
        h.join().unwrap();
    }
}
```

Todo lo que hemos hecho es cambiar el ciclo en `main()`, y agregado un segundo! Este es el primer cambio:

```rust,ignore
let handles: Vec<_> = filosofos.into_iter().map(|f| {
    thread::spawn(move || {
         f.comer();
    })
}).collect();
```

Aun así son solo cinco lineas, son cinco densas lineas. Analicemos por partes.

```rust,ignore
let handles: Vec<_> =
```

Introducimos un nuevo binding, llamado `handles`. Le hemos dado este nombre porque crearemos algunos nuevos hilos, que resultaran en algunos handles (agarradores, manillas) a esos dichos hilos los cuales nos permitan controlar su operación. Necesitamos anotar el tipo explícitamente, debido a algo que haremos referencia mas adelante. El  `_` es un marcador de posición para un tipo. Estamos diciendo “`handles` es un vector de algo, pero tu, Rust, puedes determinar que es ese algo.”

```rust,ignore
filosofos.into_iter().map(|f| {
```

Tomamos nuestra lista de filósofos y llamamos `into_iter()` en ella. Esto crea un iterador que se adueña (toma pertenencia) de cada filosofo. Necesitamos hacer esto para poder pasar los filósofos a nuestros hilos. Luego tomamos ese iterador y llamamos `map` en el, método que toma un closure como argumento y llama dicho closure en cada uno de los elemento a la vez. 

```rust,ignore
    thread::spawn(move || {
        f.comer();
    })
```

Es aquí donde la concurrencia ocurre. La función `thread::spawn` toma un closure como argumento y ejecuta ese closure en un nuevo hilo. El closure necesita una anotación extra, `move`, para indicar que el closure va a adueñarse de los valores que esta capturando. Principalmente, la variable `f` de la función `map`.

Dentro del hilo, todo lo que hacemos es llamar a `comer();` en `f`.

```rust,ignore
}).collect();
```

Finalmente, tomamos el resultado de todos esas llamadas a `map` y los coleccionamos. `collect()` los convertirá en una colección de alguna tipo, que es el porque anotamos el tipo de retorno: queremos un `Vec<T>`. Los elementos son los valores retornados de las llamadas a `thread::spawn`, que son handles a esos hilos. Whew!

```rust,ignore
for h in handles {
    h.join().unwrap();
}
```

Al final de `main()`, iteramos a través de los handles llamando `join()` en ellos, lo cual bloquea la ejecución hasta que el hilo haya completado su ejecución. Esto asegura que el hilo complete su ejecución antes que el programa termine. 

Si ejecutas este programa, veras que los filósofos comen sin orden!
Tenemos multi-hilos!


```text
Gilles Deleuze esta comiendo.
Gilles Deleuze ha finalizado de comer.
Emma Goldman esta comiendo.
Emma Goldman ha finalizado de comer.
Michel Foucault esta comiendo.
Judith Butler esta comiendo.
Judith Butler ha finalizado de comer.
Karl Marx esta comiendo.
Karl Marx ha finalizado de comer.
Michel Foucault ha finalizado de comer.
```

Pero que acerca de los tenedores no los hemos modelado del todo todavía.

Para hacerlo, creemos un nuevo `struct`:

```rust
use std::sync::Mutex;

struct Mesa {
    tenedores: Vec<Mutex<()>>,
}
```

Esta `Mesa` contiene un vector de `Mutex`es. Un mutex es una forma de controlar concurrencia, solo un hilo puede acceder el contenido a la vez. Esta es la exactamente la propiedad que necesitamos para nuestros tenedores. Usamos una dupla vacía, `()`,  dentro del mutex, debido a que no vamos a usar el valor, solo nos aferraremos a el.

Modifiquemos el programa para hacer uso de `Mesa`:

```rust
use std::thread;
use std::sync::{Mutex, Arc};

struct Filosofo {
    nombre: String,
    izquierda: usize,
    derecha: usize,
}

impl Filosofo {
    fn new(name: &str, izquierda: usize, derecha: usize) -> Filosofo {
        Filosofo {
            nombre: nombre.to_string(),
            izquierda: izquierda,
            derecha: derecha,
        }
    }

    fn comer(&self, mesa: &Mesa) {
        let _izquierda = mesa.tenedores[self.izquierda].lock().unwrap();
        let _derecha = mesa.tenedores[self.derecha].lock().unwrap();

        println!("{} esta comiendo.", self.nombre);

        thread::sleep_ms(1000);

        println!("{} ha finalizado de comer.", self.nombre);
    }
}

struct Mesa {
    tenedores: Vec<Mutex<()>>,
}

fn main() {
    let mesa = Arc::new(Mesa { tenedores: vec![
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
    ]});

    let filosofos = vec![
        Filosofo::new("Judith Butler", 0, 1),
        Filosofo::new("Gilles Deleuze", 1, 2),
        Filosofo::new("Karl Marx", 2, 3),
        Filosofo::new("Emma Goldman", 3, 4),
        Filosofo::new("Michel Foucault", 0, 4),
    ];

    let handles: Vec<_> = filosofos.into_iter().map(|f| {
        let mesa = mesa.clone();

        thread::spawn(move || {
            f.comer(&mesa);
        })
    }).collect();

    for h in handles {
        h.join().unwrap();
    }
}
```

Muchos cambios! Sin embrago, con esta iteración, hemos obtenido un programa funcional.
Veamos los detalles:

```rust,ignore
use std::sync::{Mutex, Arc};
```

Usaremos otra estructura del paquete `std::sync`: `Arc<T>`.

Hablaremos mas acerca de ella cuando la usemos.


```rust,ignore
struct Filosofo {
    nombre: String,
    izquierda: usize,
    derecha: usize,
}
```

Vamos a necesitar agregar dos campos mas a nuestra estrutura `Filosofo`. Cada filosofo tendra dos tenedores: el de la izquierda, y el de la derecha. Usaremos el tipo `usize` para indicarlos, debido a que este es el tipo con el cual se indexan los vectores. Estos dos valores serán indices en los `tenedores` que nuestra `Mesa` posee.


```rust,ignore
fn new(nombre: &str, izquierda: usize, derecha: usize) -> Filosofo {
    Filosofo {
        nombre: nombre.to_string(),
        izquierda: izquierda,
        derecha: derecha,
    }
}
```

Ahora necesitamos construir esos valores `izquierda` y `derecha`, de manera que podamos agregaremos a `new()`.


```rust,ignore
fn comer(&self, mesa: &Mesa) {
    let _izquierda = mesa.tenedores[self.izquierda].lock().unwrap();
    let _derecha = mesa.tenedores[self.derecha].lock().unwrap();

    println!("{} esta comiendo.", self.nombre);

    thread::sleep_ms(1000);

    println!("{} ha finalizado de comer.", self.nombre);
}
```

Tenemos dos nuevas lineas, también hemos agregado un argumento, `mesa`. Accedemos a la lista de tenedores de la `Mesa`, y después usamos `self.izquierda` y `self.derecha` para acceder al tenedor en un indice en particular. Eso nos da acceso al `Mutex` en ese indice, en donde llamamos `lock()`. Si el mutex esta siendo accedido actualmente por alguien mas, nos bloquearemos hasta que este disponible. 

La llamada a `lock()` puede fallar, y si lo hace, queremos terminar abruptamente. En este caso el error que puede ocurrir es que el mutex este [‘envenenado’][poison] (‘poisoned’), que es lo que ocurre cuando el hilo hace pánico mientras el mantiene el bloqueo. Debido a que esto no debería ocurrir, simplemente usamos `unwrap()`.

[poison]: ../std/sync/struct.Mutex.html#poisoning

Otra cosa extraña acerca de esta lineas: hemos nombrado los resultados `_izquierda` and `_derecha`. Que hay con ese sub-guion? Bueno, en realidad no planeamos _usar_ el valor dentro del bloqueo. Solo queremos adquirirlo. A consecuencia de esto Rust nos advertira que nunca usamos el valor. A través del uso del sub-guion le decimos a Rust que es lo que quisimos, de esa manera no generara la advertencia. 

Que acerca de soltar el bloqueo, Bueno, esto ocurrirá cuando `_izquierda` y `_derecha` salgan de ámbito, automáticamente. 

```rust,ignore
    let mesa = Arc::new(Mesa { tenedores: vec![
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
    ]});
```

A continuacion, en `main()`, creamos una nueva `Mesa` y la envolvemos en un `Arc<T>`. ‘arc’ proviene de ‘atomic reference count’ (cuenta de referencias atómica),necesitamos compartir nuestra `Mesa` entre multiples hilos. A media que la compartimos, la cuenta de referencias subirá, y cuando cada hilo termine, ira bajando.

```rust,ignore
let filosofos = vec![
    Filosofo::new("Judith Butler", 0, 1),
    Filosofo::new("Gilles Deleuze", 1, 2),
    Filosofo::new("Karl Marx", 2, 3),
    Filosofo::new("Emma Goldman", 3, 4),
    Filosofo::new("Michel Foucault", 0, 4),
];
```

Necesitamos pasar nuestros valores `izquierda` and `derecha` a los constructores de nuestros `Filosofo`s. Pero hay un detalle mas aquí, y es _muy_ importante. Si observas al patron, es conocítente hasta el final,  Monsieur Foucaultdebe tener `4, 0` como argumentos, pero en vez de esto tiene `0, 4`. Esto es lo que previene deadlocks, en efecto: uno de los filósofos es zurdo! Esa es una forma de resolver el problema, y en mi opinion, es la mas simple.

```rust,ignore
let handles: Vec<_> = filosofos.into_iter().map(|f| {
    let table = mesa.clone();

    thread::spawn(move || {
        f.comer(&mesa);
    })
}).collect();
```

Finalmente, dentro de nuestro ciclo `map()`/`collect()`, llamamos `mesa.clone()`. El método `clone()` en `Arc<T>` es lo que incrementa la cuenta de referencias, y cuando sale de ámbito, decrementa la cuenta. Notaras que podemos introducir una nueva variable  `table`, y esta sobre escribirá (shadow) la anterior. Esto es frecuentemente usado de manera tal de no tener que inventar dos nombres únicos. 

Con todo esto, nuestro programa funciona! Solo dos filosofo pueden comer en un momento dado y en consecuencia tendrás salida que lucirá así:


```text
Gilles Deleuze esta comiendo.
Emma Goldman esta comiendo.
Emma Goldman ha finalizado de comer.
Gilles Deleuze ha finalizado de comer.
Judith Butler esta comiendo.
Karl Marx esta comiendo.
Judith Butler ha finalizado de comer.
Michel Foucault esta comiendo.
Karl Marx is done eating.
Michel Foucault ha finalizado de comer.
```

Felicitaciones! Haz implementado un problema clásico de concurrencia en Rust.


