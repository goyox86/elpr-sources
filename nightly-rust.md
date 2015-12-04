% Rust Nocturno

Rust proporciona tres canales de distribucion: nocturno (nightly), beta y estable (stable). Las facilidades inestables estan solo disponibles en Rust nocturno. Para mayor detalle acerca de este proceso, vease ‘[Estabilidad como un entregable][stability]’.

[stability]: http://blog.rust-lang.org/2014/10/30/Stability.html

Para instalar Rust nocturno, puedes usar `rustup.sh`:

```bash
$ curl -s https://static.rust-lang.org/rustup.sh | sh -s -- --channel=nightly
```

Si tienes alguna duda acerca de la [inseguridad potencial] del uso de `curl | sh`, por favor, continua leyendo y encontraras nuestro disclaimer. Sientete tambien libre de usar una version de dos pasos de la instalacion examinando nuestro script de instalacion:

```bash
$ curl -f -L https://static.rust-lang.org/rustup.sh -O
$ sh rustup.sh --channel=nightly
```

[insecurity]: http://curlpipesh.tumblr.com

Si estas en Windows, por favor descarga bien sea el [instalador de 32 bits][win32] o el [instalador de 64 bits][win64] y ejecutalo.

[win32]: https://static.rust-lang.org/dist/rust-nightly-i686-pc-windows-gnu.msi
[win64]: https://static.rust-lang.org/dist/rust-nightly-x86_64-pc-windows-gnu.msi

## Desinstalando

Si has decidido que ya no quieres mas a Rust, estaremos un pco triste, pero esta bien. No todos los leguajes de programacion son para todo el mundo. Simplemente ejecuta el script de desinstalacion:

```bash
$ sudo /usr/local/lib/rustlib/uninstall.sh
```

Si usaste el instalador de Windows, simplemente ejecuta el `.msi` otra vez y este te dara la opcion de desinstalacion.

Algunas personas, y de alguna forma con derecho, se sienten perturbados cuando les decimos que hagan `curl | sh`. Basicamente, cuando haces esto, estas confiando que la buena gente que mantiene Rust no van a hackear tu computadora y hacer cosas malas. Eso es un buen instinto! Si eres una de esas personas, por favor echa un vistazo a la documentacion acerca de la [instalacion de Rust desde fuentes][from-source]. o las [descargas de los ejecutables oficiales][install-page].

[from-source]: https://github.com/rust-lang/rust#building-from-source
[install-page]: https://www.rust-lang.org/install.html

Ah, debemos mencionar las plataforma oficialmente soportadas:

* Windows (7, 8, Server 2008 R2)
* Linux (2.6.18 o mayor, varias distribuciones), x86 and x86-64
* OSX 10.7 (Lion) o mayor, x86 and x86-64

Nosotors, el equipo de Rust probamos Rust de manera extensiva en esas plataformas, y otras pocas mas, como Android. Pero estas son las mas propensas en funcionar correctamente, debido a que son las que poseen mas pruebas.

Finalmente, un comentario caerca de Windows. Rust considera a Windows como una plataforma de primera clase en lo que se refiere a la liberacion, pero si somos honestos, la experiencia en Windows no estan integrada como las experiencia en Linux/OS X. Estamos trabajando en ello! Si algo no funciona, es un bug. Por favor haznos saber si eso pasa. Cada commmit es probado en Windows justo como cualquier otra plataforma.

Si has instalado Rust, puedes abrir una terminal, y escribir:

```bash
$ rustc --version
```

Deberias ver el numero de version, el hash de commit, fecha del commit y fecha de compilacion:

```bash
rustc 1.0.0-nightly (f11f3e7ba 2015-01-04) (built 2015-01-06)
```

Si lo ves, Rust ha sido instalado satisfactoriamente! Felicidades!

Este instalador tambien instala una copia local de la documentacion, de manera que puedas leerla offline. En sistemas UNIX, la locacion es `/usr/local/share/doc/rust`. En windows, se ubica en el directorio `share/doc`, en donde sea que hayas isnataldo Rust.

Si no, hay una variedad de lugares en los que puedes solicitar ayuda. El mas facil es [el canal #rust en irc.mozilla.org][irc], el cual puedes acceder via [Mibbit][mibbit]. Cliquea ese enlace, y estaras chateando con Rutaceos (un tonto apodo con el que nos referimos a nosotros), y podremos ayudarte. Otros muy buenos recursos incluyen [el foro de usuarios, y ][users], y [Stack Overflow][stackoverflow].

[irc]: irc://irc.mozilla.org/#rust
[mibbit]: http://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust
[users]: https://users.rust-lang.org/
[stackoverflow]: http://stackoverflow.com/questions/tagged/rust
