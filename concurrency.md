% Concurrencia

La concurrencia y el paralelismo son dos tópicos increíblemente importantes en las ciencias de la computación, también son un tópico caliente en la industria hoy en día. Las computadoras cada vez poseen mas y mas núcleos, aun así, todavía algunos desarrolladores no están preparados para utilizarnos completamente.

La seguridad en el manejo de memoria de Rust también aplica a su historia de concurrencia. Incluso los programas concurrentes deben ser seguros en el manejo de memoria, sin condiciones de carrera. El sistema de tipos de Rust esta a la altura del desafío, y te dota de poderosas vías para razonar acerca de código concurrente en tiempo de compilación.

Antes de que hablemos de las características de concurrencia que vienen con Rust, es importante entender algo: Rust es de bajo nivel, suficientemente bajo al punto que todas estas facilidades están implementadas en la biblioteca estándar. Esto significa que si no te gusta algún aspecto de la manera en la que Rust maneja la concurrencia, puedes implementar una forma alternativa de hacerlo. [mio](https://github.com/carllerche/mio) es un vivo ejemplo de este principio en acción.


## Bases: `Send` y `Sync`

La concurrencia es algo sobre lo que es difícil razonar. En Rust tenemos un poderoso, sistema de tipos estático que nos ayuda a razonar acerca de nuestro código. Como tal, Rust nos provee de dos traits para ayudarnos a darle sentido a código que pueda posiblemente ser concurrente.


### `Send`

El primer trait del cual hablaremos es [`Send`](../std/marker/trait.Send.html) (ingles). Cuando un tipo `T` implementa `Send`, le indica al compilador que algo de este tipo puede transferir la pertenencia entre hilos de forma segura.

Esto es importante para hacer cumplir ciertas restricciones. Por ejemplo si tenemos un canal, conectando dos hilos, deberíamos querer transferir algunos datos a el otro hilo través del canal. Por lo tanto, nos aseguramos de que `Send` haya sido implementado para ese tipo.

De manera opuesta, si estamos envolviendo una biblioteca con FFI que no es threadsafe, no deberíamos querer implementar `Send`, de manera tal que el compilador nos ayude a asegurarnos que esta no pueda abandonar el hilo actual.


### `Sync`


El segundo de estos traits es llamado [`Sync`](../std/marker/trait.Sync.html) (ingles). Cuando un tipo `T` implementa `Sync`, le indica al el compilador que algo de este tipo no tiene posibilidad de introducir inseguridad en memoria cuando es usado de manera concurrente por multiples hilos de ejecución.

Por ejemplo, compartir data inmutable con una cuenta de referencias atómica es threadsafe. Rust provee dicho tipo, `Arc<T>`, el cual implementa `Sync`, y es por ello que es seguro de compartir entre hilos.

Estos dos traits te permiten usar el sistema de tipos para hacer garantías fuertes acerca de las propiedades de tu código bajo concurrencia. En primer lugar, antes de demostrar porque, necesitamos aprender como crear un programa concurrente en Rust!


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

El método `thread::spawn()` acepta un closure como argumento, closure que es ejecutado en un nuevo hilo. `thread::spawn()` retorna un handle a el nuevo hilo, que puede ser usado para esperar a que el hilo finalice y luego extraer su resultado:

```rust
use std::thread;

fn main() {
    let handle = thread::spawn(|| {
        "Hola desde un hilo!"
    });

    println!("{}", handle.join().unwrap());
}
```

Muchos lenguajes poseen la habilidad de ejecutar hilos, pero es salvajemente inseguro. Existen libros enteros acerca de como prevenir los errores que ocurren como consecuencia de compartir estado mutable. Rust ayuda con su sistema de tipos, previniendo condiciones de carrera en tiempo de compilación. Hablemos acerca de como efectivamente puedes compartir cosas entre hilos.


## Estado Mutable Compartido Seguro

Debido a el sistema de tipos de Rust, tenemos un concepto que suena como una mentira: "estado mutable compartido seguro". Muchos programadores concuerdan en que el estado mutable compartido es muy, muy malo.

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

En este caso, sabemos que nuestro código *deberia* ser seguro, pero Rust no esta seguro de ello. En realidad no es seguro, si tuviéramos una referencia a `data` en cada hilo, y el hilo toma pertenencia de la referencia, tendríamos tres dueños! Eso esta mal. Podemos arreglar esto a través del uso del tipo `Arc<T>`, un apuntador con conteo atómico de referencias. La parte 'atómico' significa que es seguro compartirlo entre hilos.

`Arc<T>` asume una propiedad mas acerca de su contenido para asegurarse que es seguro compartirlo entre hilos: asume que su contenido es `Sync`. Pero en nuestro caso, queremos mutar el valor. Necesitamos un tipo que pueda asegurarse que solo una persona a la vez pueda mutar lo que este dentro. Para eso, podemos usar el tipo `Mutex<T>`. He aquí la segunda version de nuestro código.  Aun no funciona, pero por una razón diferente:


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

Ahora llamamos `clone()` en nuestro `Arc`, lo cual incrementa el contador interno. Este handle es entonces movido dentro del nuevo hilo. Examinemos el cuerpo del hilo mas de cerca:


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

Por ultimo, mientras que los hilos se ejecutan, esperamos por la culminación de un temporizador corto. Esto no es ideal: pudimos haber escogido un tiempo razonable para esperar pero lo mas probable es que esperemos mas de lo necesario, o no lo suficiente, dependiendo de cuanto tiempo los hilos toman para terminar la computación cuando el programa corre.

Una alternativa mas precisa a el temporizador seria el uso de uno de los mecanismos proporcionados por la biblioteca estándar de Rust para la sincronización entre hilos. Hablemos de ellos: los canales.

## Canales

He aquí una version nuestro código que usa canales para la sincronización, en lugar de esperar por un tiempo especifico:


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

Hacemos uso del método `mpsc::channel()` para construir un canal nuevo. Enviamos (a través de `send`) un simple `()` a través del canal, y luego esperamos por el regreso diez de ellos.

Mientras que este canal esta solo enviando una senal genérica, podemos enviar cualquier data que sea `Send` a través del canal!


```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    for _ in 0..10 {
        let tx = tx.clone();

        thread::spawn(move || {
            let respuesta = 42u32;

            tx.send(respuesta);
        });
    }

   rx.recv().ok().expect("No se ha podido recibir la respuesta");
}
```

Un `u32` es `Send` porque podemos hacer una copia de el. Entonces creamos un hilo, y le solicitamos que calcule la respuesta, este luego nos envía la respuesta de regreso (usando `send()`) a través del canal.


## Panicos

Un `panic!` causara la finalización abrupta (crash) del hilo de ejecución actual. Puedes usar los hilos de Rust como un mecanismo de aislamiento sencillo:


```rust
use std::thread;

let resultado = thread::spawn(move || {
    panic!("ups!");
}).join();

assert!(resultado.is_err());
```

Nuestro `Thread` nos devuelve un `Result`, el cual nos permite chequear si el hilo ha hecho pánico o no.