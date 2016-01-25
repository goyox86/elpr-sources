% Glosario

No todos los rustaceos poseen una base en programación de sistemas, o en ciencias de la computación, es por ello que hemos agregado explicaciones para algunos términos que podrían no ser familiares.

### Arbol Abstracto de Sintaxis

Cuando un compilador traduce tu programa, lleva a cabo un numero de cosas diferentes. Una de ellas es transformar el texto de tu programa en un ‘árbol abstracto de sintaxis’, o ‘AST’ (por sus siglas en ingles). Dicho árbol es una representación de la estructura de tu programa. Por ejemplo, `2 + 3` puede ser convertido a:

```text
  +
 / \
2   3
```

Y `2 + (3 * 4)` luciría así:

```text
  +
 / \
2   *
   / \
  3   4
```

### Aridad

La aridad se refiere a el numero de argumentos que recibe una función u operación.

```rust
let x = (2, 3);
let y = (4, 6);
let z = (8, 2, 6);
```

En el ejemplo anterior `x` y `y` poseen una aridad de 2. `z` tiene una aridad de 3.

### Limites (Bounds)

Los limites son restricciones en un tipo o [trait][traits]. Por ejemplo, si un limite es colocado en un argumento a una función, los tipos proporcionados a dicha función deben cumplir con las restricciones impuestas por el limite.

[traits]: traits.html

### Tipos de Tamaño Dinámico TTD (Dynamically Sized Type)

Un tipo sin un tamaño o alineación conocidos de manera estática. ([mas informacion][link])

[link]: ../nomicon/exotic-sizes.html#dynamically-sized-types-dsts

### Expresión

En ciencias de la computación, una expresión es una combinación de valores, constantes, variables, operadores y funciones que evalúan a un único valor. Por ejemplo, `2 + (3 * 4)` es una expresión que retorna el valor 14. Es importante mencionar que las expresiones pueden tener efectos secundarios. Por ejemplo, una función incluida en una expresión podría llevar a cabo otras acciones ademas de el simple retorno de un valor.

### Lenguaje de Programación Orientado a Expresiones

En los primeros lenguajes de programación, las [expresiones][expression] y [sentencias][statement] eran dos categorías sintácticas diferentes: las expresiones poseían un valor de retorno y las sentencias llevaban a cabo acciones. Sin embargo, los lenguajes mas modernos hicieron a esta distinción un poco borrosa, permitiendo a las expresiones llevar a cabo acciones y a las sentencias tener un valor. En un lenguaje de programación orientado a expresiones, (casi) toda sentencia es una expresión y por lo tanto, retorna un valor. Consecuentemente, dicho sentencias expresión pueden en si mismas formar parte de expresiones mas grandes.

[expression]: glossary.html#expression
[statement]: glossary.html#statement

### Sentencia

En ciencias de la computación, una sentencia es el elemento libre mas pequeño en un lenguaje de programación que comanda a una computadora a llevar a cabo una acción.
