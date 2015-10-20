% ¡Hola, mundo!

Ahora que has instalado Rust, escribamos tu primer programa. Es tradición que tu primer programa en cualquier lenguaje 
sea uno que imprima el texto “¡Hola, mundo!” a la pantalla. Lo bueno de comenzar con un programa tan simple es que 
verificas no solo que el compilador esta instalado, sino que esta funcionando. Imprimir información a la pantalla es una cosa muy común.

La primera cosa que debemos hacer es crear un archivo en donde poner nuestro código. A mi me gusta crear un directorio 
`proyectos` en mi directorio de usuario y mantener todos mis proyectos allí. A Rust no le importa dónde reside el código.

Esto lleva en cuestión a otro asunto que debemos aclarar: esta guía asumirá que posees una familiaridad básica con la 
linea de comandos. Rust no demanda nada con respecto a tus herramientas de edición o dónde vive tu código. Si prefieres 
un IDE a la interfaz de linea de comandos, probablemente deberías echar un vistazo a [SolidOak][solidoak], o donde sea 
que los plugins estén para tu IDE favorito. Existen un número de extensiones de calidad variada en desarrollo por la 
comunidad. El equipo detrás de Rust también publica [plugins para varios editores][plugins]. Configurar tu editor o IDE 
escapa de los objetivos de éste tutorial, chequea la documentación para tu configuración específica.

[solidoak]: https://github.com/oakes/SolidOak
[plugins]: https://github.com/rust-lang/rust/blob/master/src/etc/CONFIGS.md

Dicho esto, creemos un directorio en nuestro directorio de proyectos.

```bash
$ mkdir ~/proyectos
$ cd ~/proyectos
$ mkdir hola_mundo
$ cd hola_mundo
```

Si estás en Windows y no estás usando PowerShell, el `~` podría no funcionar. Consulta la documentacion específica para 
tu terminal para mayor detalle.

Creemos un nuevo archivo de código fuente. Llamaremos `main.rs` a nuestro archivo. Los archivos Rust terminan con la 
extensión `.rs`. Si estás usando más de una palabra en tu nombre de archivo, usa un subguion: `hola_mundo.rs` en vez de `holamundo.rs`.

Ahora que tienes tu archivo abierto escribe esto en el:

```rust
fn main() {
    println!("¡Hola, mundo!");
}
```

Guarda los cambios en el archivo,  y escribe lo siguiente en la ventana de tu terminal:

```bash
$ rustc main.rs
$ ./main # o main.exe en Windows
¡Hola, mundo!
```

¡Éxito! Ahora veamos que ha pasado en detalle.

```rust
fn main() {

}
```

Estas lineas definen una *función* en Rust. La función `main` es especial: es el principio de todo programa Rust. La 
primera línea dice: "Estoy declarando una función llamada `main` la cual no recibe argumentos y no retorna nada." Si 
existieran argumentos, estos irían dentro de paréntesis (`(` and `)`), y debido a que no estamos retornando nada de 
esta función, podemos omitir el tipo de retorno completamente. Llegaremos a esto más adelante.

También notaras que la función está envuelta en llaves (`{` and `}`). Rust requiere dichas llaves delimitando todos los 
cuerpos de función. Es también considerado buen estilo colocar la llave de apertura en la misma linea de la declaración 
de la función, con un espacio intermedio.

Lo siguiente es esta linea:

```rust
    println!("¡Hola, mundo!");
```

Esta línea hace todo el trabajo en nuestro pequeño programa. Hay un número de detalles que son importantes aquí. El 
primero es que está sangrado con cuatro espacios, no tabulaciones. Por favor, configura tu editor a insertar cuatro 
espacios con la tecla tab. Proveemos algunas [configuraciones de ejemplo para varios editores][configs].

[configs]: https://github.com/rust-lang/rust/tree/master/src/etc/CONFIGS.md

El segundo punto es la parte `println!()`. Esto es llamar a una macro Rust [macro][macro], que es como se hace metaprogramación 
en Rust. Si esto fuese en cambio una función, luciria asi: `println()`. Para nuestros efectos, no necesitamos preocuparnos 
acerca de esta diferencia. Solo saber que algunas veces verás `!`, y que esto significa que estas llamando a una macro 
en vez de una función normal. Rust implementa `println!` como una macro en vez de como una función por buenas razones, 
pero eso es un tópico avanzado. Una última cosa por mencionar: Las macros en Rust son diferentes de las macros en C, 
si las has usado. No estés asustado de usar macros. Llegaremos a los detalles eventualmente, por ahora simplemente debes 
confiar en nosotros.

[macro]: macros.html

A continuación, `"¡Hola, mundo!"` es una cadena de caracteres. Las cadenas de caracteres son un tópico sorprendentemente 
complejo en lenguajes de programación de sistemas, y ésta concretamente es una cadena de carateres ‘asiganda estáticamente’. 
Si te gustaría leer acerca de asignación de memoria, echa un vistazo a [la pila y el montículo][allocation], pero por 
ahora no necesitas hacerlo si no lo deseas. Pasamos esta cadena de caracteres como un argumento a `println!` quien a 
su vez imprime la cadena de caracteres a la pantalla. ¡Fácil!

[allocation]: the-stack-and-the-heap.html

Finalmente, la linea termina con un punto y coma  (`;`). Rust es un lenguaje orientado a expresiones, lo que significa 
que la mayoría de las cosas son expresiones, en vez de sentencias. El `;` se usa para indicar que la expresión ha 
terminado, y que la siguiente está lista para comenzar. La mayoría de las líneas de código en Rust terminan con un `;`.

Finalmente, compilar y ejecutar nuestro programa. Podemos compilar con nuestro compilador `rustc` pasándole el nombre de 
nuestro archivo de código fuente:


```bash
$ rustc main.rs
```

Esto es similar a `gcc` or `clang`, si provienes de C o C++. Rust emitirá un binario ejecutable. Puedes verlo con `ls`:


```bash
$ ls
main  main.rs
```

O en Windows:

```bash
$ dir
main.exe  main.rs
```

Hay dos archivos: nuestro código fuente, con la extensión `.rs`, y el ejecutable (`main.exe` en Windows, `main` en los demás)


```bash
$ ./main  # o main.exe en Windows
```

Lo anterior imprime nuestro texto `¡Hola, mundo!` a nuestra terminal.

Si provienes de un lenguaje dinámico como Ruby, Python o Javascript, probablemente no estés acostumbrado a ver estos 
dos pasos como separados. Rust es un lenguaje compilado, lo cual significa que puedes compilar tu programa, dárselo a 
alguien más, y éste no necesita tener Rust instalado. Si les das a alguien un archivo `.rb` o `.py` o `.js`, este 
necesita tener una implementación de Ruby/Python/JavaScript, pero sólo necesitas un comando para ambos, compilar y 
ejecutar tu programa. Todo es acerca de balance entre ventajas/desventajas en el diseño de lenguajes, y Rust ha elegido.

Felicitaciones, has escrito oficialmente un programa Rust. ¡Eso te convierte en un programador Rust! Bienvenido. 🎊🎉👍

A continuación me gustaría presentarte otra herramienta, Cargo, el cual es usado para escribir programas Rust para el 
mundo real. Solo usar `rustc` esta bien para cosas simples, pero a medida que tu proyecto crece, necesitarás algo que 
te ayude a administrar todas las opciones que este tiene, así como hacer fácil compartir el codigo con otras personas y proyectos.
