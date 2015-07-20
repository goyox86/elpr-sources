% Canales de Distribución

El proyecto Rust usa un concepto denominado ‘canales de distribución’ para administrar los releases.

Es importante entender este proceso para así poder decidir cual version de Rust debe usar tu proyecto.

# Visión general

Hay tres canales para los releases de Rust:

* Nocturno (nightly)
* Beta
* Estable (stable)

Los releases nocturnos son creados una vez al día. Cada seis semanas, el ultimo release nocturno es promovido a 'Beta'. En este punto, solo recibirá parches que arreglen errores serios. Seis semanas después el beta es promovido a 'Estable', y se convierte en el nuevo release de `1.x`.

Este proceso ocurre en paralelo. Cada seis semanas, en el mismo día el nocturno sube a beta, el beta es promovido a estable. Cuando `1.x` es liberado, al mismo tiempo, `1.(x + 1)-beta` es liberado, y el nocturno se vuelve la primera version de `1.(x + 2)-nightly`.

# Escogiendo una versión

Generalmente hablando, a menos que tengas una razón especifica, deberías usar el canal de distribución estable. Esos releases tienen como objetivo una audiencia general.

Sin embargo, dependiendo de tu interés en Rust, podrías escoger el nocturno. El balance es el siguiente: en el canal nocturno, puedes hacer uso de nuevas características de Rust. Sin embargo, las características inestables están sujetas al cambio, y es por ello que cualquier nocturno tiene la posibilidad de romper tu código. Si usas el release estable, no tienes acceso a características experimentales, pero el siguiente release de Rust no causara problemas significativos a causa de cambios.

# Ayudando a el ecosistema a traves de CI

Que hay acerca de beta? Nosotros recomendamos a todos los usuarios Rust que usan el canal de distribución estable a que también prueben en sus sistemas de integración continua con el canal beta. Esto ayudara a advertir al equipo en caso de que exista una regresión accidental.

Adicionalmente, probar contra el nocturno puede detectar regresiones mucho mas temprano, es por ello que si no te importa tener un tercer build, apreciamos que pruebes contra todos los canales.