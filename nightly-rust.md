% Rust Nocturno

Rust proporciona tres canales de distribución: nocturno (nightly), beta y estable (stable). Las facilidades inestables están solo disponibles en Rust nocturno. Para mayor detalle acerca de este proceso, véase ‘[Estabilidad como un entregable][stability]’.

[stability]: http://blog.rust-lang.org/2014/10/30/Stability.html

Para instalar Rust nocturno, puedes usar `rustup.sh`:

```bash
$ curl -s https://static.rust-lang.org/rustup.sh | sh -s -- --channel=nightly
```

Si tienes alguna duda acerca de la [inseguridad potencial] del uso de `curl | sh`, por favor, continua leyendo y encontraras nuestro disclaimer. Siéntete también en libertad de usar una version de dos pasos de la instalación examinando nuestro script:

```bash
$ curl -f -L https://static.rust-lang.org/rustup.sh -O
$ sh rustup.sh --channel=nightly
```

[insecurity]: http://curlpipesh.tumblr.com

Si estas en Windows, por favor descarga bien sea el [instalador de 32 bits][win32] o el [instalador de 64 bits][win64] y ejecutalo.

[win32]: https://static.rust-lang.org/dist/rust-nightly-i686-pc-windows-gnu.msi
[win64]: https://static.rust-lang.org/dist/rust-nightly-x86_64-pc-windows-gnu.msi

## Desinstalando

Si has decidido que ya no quieres mas a Rust, estaremos un poco triste, pero esta bien. No todos los lenguajes de programación son para todo el mundo. Simplemente ejecuta el script de desinstalación:

```bash
$ sudo /usr/local/lib/rustlib/uninstall.sh
```

Si usaste el instalador de Windows, simplemente ejecuta el `.msi` de nuevo y este te dará la opción de desinstalación.

Algunas personas, y de alguna forma con derecho, se sienten perturbados cuando les decimos que hagan `curl | sh`. Básicamente, cuando haces esto, estas confiando que la buena gente que mantiene Rust no van a hackear tu computadora y hacer cosas malas. Eso es =buen instinto! Si eres una de esas personas, por favor echa un vistazo a la documentación acerca de la [instalación de Rust desde fuentes][from-source]. O las [descargas de los ejecutables oficiales][install-page].

[from-source]: https://github.com/rust-lang/rust#building-from-source
[install-page]: https://www.rust-lang.org/install.html

Ah, debemos mencionar las plataformas oficialmente soportadas:

* Windows (7, 8, Server 2008 R2)
* Linux (2.6.18 o mayor, varias distribuciones), x86 and x86-64
* OSX 10.7 (Lion) o mayor, x86 and x86-64

Nosotros, el equipo de Rust probamos Rust de manera extensiva en esas plataformas, y otras pocas mas, como Android. Pero estas son las mas propensas en funcionar correctamente, debido a que son las que poseen mas pruebas.

Finalmente, un comentario acerca de Windows. Rust considera a Windows como una plataforma de primera clase en lo que se refiere al release, pero si somos honestos, la experiencia en Windows no esta tan integrada como las experiencias en Linux y OS X. Estamos trabajando en ello! Si algo no funciona, es un bug. Por favor haznos saber si eso pasa. Cada commmit es probado en Windows justo como cualquier otra plataforma.

Si has instalado Rust, puedes abrir una terminal, y escribir:

```bash
$ rustc --version
```

Deberías ver el numero de version, el hash de commit, fecha del commit y fecha de compilación:

```bash
rustc 1.0.0-nightly (f11f3e7ba 2015-01-04) (built 2015-01-06)
```

Si lo ves, Rust ha sido instalado satisfactoriamente! Felicidades!

Este instalador también instala una copia local de la documentación, de manera que puedas leerla offline. En sistemas UNIX, la locación es `/usr/local/share/doc/rust`. En windows, se ubica en el directorio `share/doc`, en donde sea que hayas instalado Rust.

Si no, hay una variedad de lugares en los que puedes solicitar ayuda. El mas fácil es [el canal #rust en irc.mozilla.org][irc], el cual puedes acceder via [Mibbit][mibbit]. Cliquea ese enlace, y estarás chateando con Rutaceos (un tonto apodo con el que nos referimos a nosotros), y podremos ayudarte. Otros muy buenos recursos incluyen [el foro de usuarios][users], y [Stack Overflow][stackoverflow].

[irc]: irc://irc.mozilla.org/#rust
[mibbit]: http://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust
[users]: https://users.rust-lang.org/
[stackoverflow]: http://stackoverflow.com/questions/tagged/rust
