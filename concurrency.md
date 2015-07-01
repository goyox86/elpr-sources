% Concurrencia

La concurrencia y el paralelismo son dos tópicos increíblemente importantes en las ciencias de la computación, también son un tópico caliente en la industria hoy en día. Las computadoras cada vez poseen mas y mas núcleos, aun así, todavia algunos desarrolladores no están preparados para utilizarnos completamente.

La seguridad en el manejo de memoria de Rust también aplica a su historia de concurrencia. Incluso programas concurrentes deben ser seguros en el manejo de memoria, sin tener condiciones de carrera. El sistema de tipos de Rust esta a la altura del desafío, y te dota de poderosas vías para razonar acerca de código concurrente en tiempo de compilación.

Antes de que hablemos de las características de concurrencia que vienen con Rust, es importante entender algo: Rust es de bajo nivel, suficientemente bajo al punto que todas estas facilidades están implementadas en la biblioteca estándar. Esto significa que si no te gusta alguna aspecto de la manera en la que Rust maneja la concurrencia, puedes implementar una forma alternativa de hacerlo. [mio](https://github.com/carllerche/mio) es un vivo ejemplo de este principio en acción.

## Bases: `Send` y `Sync`

La concurrencia es algo sobre lo que es difícil razonar. En Rust tenemos un poderoso, sistema de tipos estáticos que nos ayuda a razonar acerca de nuestro código. Como tal, Rust nos provee de dos traits para ayudarnos a darle sentido a código que pueda posiblemente ser concurrente.

### `Send`

El primer trait del cual hablaremos es [`Send`](../std/marker/trait.Send.html) (ingles). Cuando un tipo `T` implementa `Send`, esto indica al compilador que algo de este tipo puede transferir la pertenencia entre hilos de forma segura.

Eso es importante para hacer cumplir ciertas restricciones. Por ejemplo si tenemos un canal, conectando dos hilos, deberíamos querer transferir algunos datos a el otro hilo  través del canal. Por lo tanto, nos aseguramos de que `Send` haya sido implementado para ese tipo.

De manera opuesta, si estamos envolviendo una biblioteca con FFI que no es threadsafe, no deberíamos querer implementar `Send`, de manera tal que el compilador nos ayude a asegurarnos que esta no pueda abandonar el hilo actual.


### `Sync`


El segundo de estos tras es llamado [`Sync`](../std/marker/trait.Sync.html) (ingles). Cuando un tipo `T` implementa `Sync`, indica al el compilador que algo de este tipo no tiene posibilidad de introducir inseguridad en memoria cuando es usado de manera concurrente por multiples hilos de ejecución.

Por ejemplo, compartir data inmutable con una cuenta de referencias atómica es threadsafe. Rust provee un tipo así, `Arc<T>`, el cual implementa `Sync`, ese por ello que es seguro de compartir entre hilos.

Estos dos traits te permiten usar el sistemas de tipos para hacer garantías fuertes acerca de las propiedades de tu código bajo concurrencia. En primer lugar, antes de demostrar porque, necesitamos aprender como crear un programa concurrente en Rust!

## Hilos

La biblioteca estándar de Rust provee una biblioteca para el manejo de hilos, que te permite ejecutar código rust de forma paralela. He aqui un ejemplo basico del uso de `std::thread`:

```rust
use std::thread;

fn main() {
    thread::spawn(|| {
        println!("Hola desde un hilo!");
    });
}
```

El método `thread::spawn()` acepta un closure como argumento, closure que es ejecutado en un nuevo hilo. Este retorna un handle a el nuevo hilo, el cual puede ser usado para esperar a que el hilo hilo finalice y extraer su resultado:

The `thread::spawn()` method accepts a closure, which is executed in a
new thread. It returns a handle to the thread, that can be used to
wait for the child thread to finish and extract its result:

```rust
use std::thread;

fn main() {
    let handle = thread::spawn(|| {
        "Hola desde un hilo!"
    });

    println!("{}", handle.join().unwrap());
}
```

Muchos lenguajes poseen la habilidad de ejecutar hilos, pero es salvajemente inseguro. Existen libros enteros acerca de como prevenir los errores que ocurren de compartir estado mutable. Rust ayuda con su sistema de tipos, previniendo condiciones de carrera en tiempo de compilación. Hablemos acerca de como efectivamente puedes compartir cosas entre hilos.


## Estado Mutable Compartido Seguro

Debido a el sistema de tipos de Rust,  tenemos un concepto que suena como una mentira: "estado mutable compartido seguro". Muchos programadores concuerdan en que el estado mutable compartido es muy, muy malo.

Alguien dijo alguna vez:

> El estado mutable compartido es la raíz de toda maldad. La mayoría de los lenguajes intentan lidiar con este problema a través de la parte 'mutable', pero Rust lo enfrenta resolviendo la parte 'compartida'.

La misma pertenencia que previene el uso incorrecto de apuntadores también ayuda a eliminar las condiciones de carrera, uno de los peores bugs relacionados con concurrencia.

Como ejemplo, un programa Rust que tendría una condición de carrera en muchos lenguajes. En Rust no compilaría:

```ignore
use std::thread;

fn main() {
    let mut data = vec![1u32, 2, 3];

    for i in 0..3 {
        thread::spawn(move || {
            data[i] += 1;
        });
    }

    thread::sleep_ms(50);
}
```

Lo anterior produce un error:

```text
8:17 error: capture of moved value: `data`
        data[i] += 1;
        ^~~~
```

En este caso, sabemos que nuestro código *deberia* ser seguro, pero Rust no esta seguro. En realidad no es seguro, si tuviéramos una referencia a  `data` en cada hilo, y el hilo toma pertenencia de la referencia, tendríamos tres duenos! Eso esta mal. Podemos arreglar esto a través del uso del tipo `Arc<T>`, un apuntador con conteo atómico de referencias. La parte 'atómico' significa que es seguro compartirlo entre hilos.

`Arc<T>` asume una propiedad mas acerca de su contenido para asegurarse que es seguro compartirlo entre hilos: asume que su contenido es `Sync`. Pero en nuestro caso, queremos mutar el valor. Necesitamos un tipo que pueda asegurarse que solo una persona al la vez pueda mutar lo que este dentro. Para eso, podemos usar el tipo `Mutex<T>`. He aquí la segunda version de nuestro código.  Aun no funciona, pero por una razón diferente:


```ignore
use std::thread;
use std::sync::Mutex;

fn main() {
    let mut data = Mutex::new(vec![1u32, 2, 3]);

    for i in 0..3 {
        let data = data.lock().unwrap();
        thread::spawn(move || {
            data[i] += 1;
        });
    }

    thread::sleep_ms(50);
}
```

He aqui el error:

```text
<anon>:9:9: 9:22 error: the trait `core::marker::Send` is not implemented for the type `std::sync::mutex::MutexGuard<'_, collections::vec::Vec<u32>>` [E0277]
<anon>:11         thread::spawn(move || {
                  ^~~~~~~~~~~~~
<anon>:9:9: 9:22 note: `std::sync::mutex::MutexGuard<'_, collections::vec::Vec<u32>>` cannot be sent between threads safely
<anon>:11         thread::spawn(move || {
                  ^~~~~~~~~~~~~
```

Lo ves, [`Mutex`](../std/sync/struct.Mutex.html) posee un metodo [`lock`](../std/sync/struct.Mutex.html#method.lock) que posee esta firma:

```ignore
fn lock(&self) -> LockResult<MutexGuard<T>>
```

Debido a que `Send` no esta implementado para `MutexGuard<T>`, no podemos transferir el `MutexGuard<T>` entre hilos, lo que se traduce en el error.

Podemos usar `Arc<T>` para corregir el error. He aqui la version funcional:

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let data = Arc::new(Mutex::new(vec![1u32, 2, 3]));

    for i in 0..3 {
        let data = data.clone();
        thread::spawn(move || {
            let mut data = data.lock().unwrap();
            data[i] += 1;
        });
    }

    thread::sleep_ms(50);
}
```

Ahora llamamos `clone()` en nuestro `Arc`, lo cual incremente el contador interno. Este handle es entonces movido dentro del nuevo hilo. Examinemos el cuerpo del hilo mas de cerca:


```rust
# use std::sync::{Arc, Mutex};
# use std::thread;
# fn main() {
#     let data = Arc::new(Mutex::new(vec![1u32, 2, 3]));
#     for i in 0..3 {
#         let data = data.clone();
thread::spawn(move || {
    let mut data = data.lock().unwrap();
    data[i] += 1;
});
#     }
#     thread::sleep_ms(50);
# }
```

Primero, llamamos `lock()`, lo cual obtiene el bloqueo de exclusion mutua. Debido a que esta operación puede fallar, este método retorna un `Result<T, E>`, y debido a que esto es solo un ejemplo, hacemos `unwrap()` en el para obtener una referencia a la data. Código real tendría un manejo de errores mas robusto en este lugar. Somos libres de mutarlo, puesto que tenemos el bloqueo.

Por ultimo, mientras que los hilos esta corriendo, usamos un temporizador corto.
Lastly, while the threads are running, we wait on a short timer. But
this is not ideal: we may have picked a reasonable amount of time to
wait but it's more likely we'll either be waiting longer than
necessary or not long enough, depending on just how much time the
threads actually take to finish computing when the program runs.

A more precise alternative to the timer would be to use one of the
mechanisms provided by the Rust standard library for synchronizing
threads with each other. Let's talk about one of them: channels.

## Channels

Here's a version of our code that uses channels for synchronization, rather
than waiting for a specific time:

```rust
use std::sync::{Arc, Mutex};
use std::thread;
use std::sync::mpsc;

fn main() {
    let data = Arc::new(Mutex::new(0u32));

    let (tx, rx) = mpsc::channel();

    for _ in 0..10 {
        let (data, tx) = (data.clone(), tx.clone());

        thread::spawn(move || {
            let mut data = data.lock().unwrap();
            *data += 1;

            tx.send(());
        });
    }

    for _ in 0..10 {
        rx.recv();
    }
}
```

We use the `mpsc::channel()` method to construct a new channel. We just `send`
a simple `()` down the channel, and then wait for ten of them to come back.

While this channel is just sending a generic signal, we can send any data that
is `Send` over the channel!

```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    for _ in 0..10 {
        let tx = tx.clone();

        thread::spawn(move || {
            let answer = 42u32;

            tx.send(answer);
        });
    }

   rx.recv().ok().expect("Could not receive answer");
}
```

A `u32` is `Send` because we can make a copy. So we create a thread, ask it to calculate
the answer, and then it `send()`s us the answer over the channel.


## Panics

A `panic!` will crash the currently executing thread. You can use Rust's
threads as a simple isolation mechanism:

```rust
use std::thread;

let result = thread::spawn(move || {
    panic!("oops!");
}).join();

assert!(result.is_err());
```

Our `Thread` gives us a `Result` back, which allows us to check if the thread
has panicked or not.
