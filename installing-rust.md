## Instalando Rust

El primer paso para usar Rust es instalarlo! Existen un numero de maneras para instalar Rust, pero la mas fácil es usar el script `rustup`. Si estas en Linux o una Mac, todo lo que necesitas hacer es (sin escribir los `$`s, solo indican el inicio de cada comando):

```bash
$ curl -sf -L https://static.rust-lang.org/rustup.sh | sh
```

Si estas preocupado acerca de la [inseguridad potencial][insecurity] de usar `curl
| sh`, por favor continua leyendo y ve nuestro disclaimer. Sientete libre de usar la version de dos pasos de la instalación y examinar nuestro script:

```bash
$ curl -f -L https://static.rust-lang.org/rustup.sh -O
$ sh rustup.sh
```

[insecurity]: http://curlpipesh.tumblr.com

Si estas en Windows, por favor descarga el [instalador de 32 bits][win32]
o el [instalador de 64 bits][win64] y ejecutalo.

[win32]: https://static.rust-lang.org/dist/rust-1.0.0-i686-pc-windows-gnu.msi
[win64]: https://static.rust-lang.org/dist/rust-1.0.0-x86_64-pc-windows-gnu.msi

## Desintalando

Si decides que ya no quieres mas Rust, estaremos un poco tristes, pero esta bien ningún lenguaje de programación es para todo el mundo. Simplemente ejecuta el script de desinstalación:

```bash
$ sudo /usr/local/lib/rustlib/uninstall.sh
```

Si usaste el instalador de Windows simplemente vuelve a ejecutar el `.msi` y este te dara la opción de desinstalar.

Algunas personas, con mucho derecho, se ven perturbadas cuando les dices que deben `curl | sh`. Básicamente, al hacerlo, estas confiando en que la buena gente que mantiene Rust no va a hackear tu computador y hacer cosas malas. Eso es buen instinto! Si eres una de esas personas por favor echa un vistazo a la documentación acerca de [compilar Rust desde el codigo fuente][from-source], o [la pagina oficial de descargas de binarios][install-page].

[from-source]: https://github.com/rust-lang/rust#building-from-source
[install-page]: http://www.rust-lang.org/install.html

Ah, también debemos mencionar las plataformas soportadas oficialmente:

* Windows (7, 8, Server 2008 R2)
* Linux (2.6.18 o mayor, varias distribuciones), x86 and x86-64
* OSX 10.7 (Lion) o mayor, x86 y x86-64

Nosotros probamos Rust extensivamente en dichas plataformas, y algunas otras pocas, como Android. Estas son las que garantizan funcionar debido a que tienen la mayor cantidad de pruebas.

Finalmente, un comentario acerca de Windows. Rust considera Windows como una plataforma de primera clase en cuanto a release, pero si somos honestos, la experiencia en Windows no es tan integrada como la de Linux/OS X. Estamos trabajando en ello! Si algo no funciona, es un bug. Por favor haznoslo saber. Todos y cada uno de los commits son probados en Windows justo como en cualquier otra plataforma.

Si ya tienes Rust instalado, puedes abrir una consola y escribir:

```bash
$ rustc --version
```

Deberias ver el numero de versión, hash del commit, fecha del commit y la fecha de compilación:

```bash
rustc 1.0.0 (a59de37e9 2015-05-13) (built 2015-05-14)
```

Si lo has visto, Rust ha sido instalado satisfactoriamente! Felicitaciones!

Este instalador también instala una copia de la documentación localmente para que puedas leerla aun estando desconectado. En sistemas UNIX, `/usr/local/share/doc/rust` es la locación.
En Windows, es en el directorio `share/doc`, dentro de donde sea que hayas instalado Rust.

Si no, hay un numero de lugares en los que puedes obtener ayuda. El mas fácil es
[el canal de IRC #rust en irc.mozilla.org][irc], al cual puedes acceder con [Mibbit][mibbit]. Sigue ese enlace y estaras hablando con Rusteros (a tonto apodo que usamos entre nosotros), y te ayudaremos. Otros excelentes recursos son [el foro de usuarios][users], y
[Stack Overflow][stackoverflow].

[irc]: irc://irc.mozilla.org/#rust
[mibbit]: http://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust
[users]: http://users.rust-lang.org/ 
[stackoverflow]: http://stackoverflow.com/questions/tagged/rust

