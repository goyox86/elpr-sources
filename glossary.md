% Glosario

No todos los rustaceos poseen una base en programacion de sistemas, o en ciencias de la computacion, es por ello que hemos agregado explicaciones para algunos terminos que podrian no ser familiares.

### Arbol Abstracto de Sintaxis

Cuando un compilador esta traduciendo tu programa, hacer un numero de cosas diferentes. Una de ellas es transformar el texto de tu programa en un ‘arbol abstracto de sintaxis’, o ‘AST’ (por sus siglas en ingles). Dicho arbol es una representacion de la estructura de tu programa. Por ejemplo, `2 + 3` puede ser convertido a un arbol:

```text
  +
 / \
2   3
```

Y `2 + (3 * 4)` luciria asi:

```text
  +
 / \
2   *
   / \
  3   4
```

### Aridad

La aridad se refiere a el numero de argumentos que recibe una funcion u operacion.

```rust
let x = (2, 3);
let y = (4, 6);
let z = (8, 2, 6);
```

En el ejemplo anterior `x` y `y` poseen una aridad de 2. `z` tiene una aridad de 3.

### Limites

Los limites son restricciones en un tipo o [trait][traits]. Por ejemplo, si un limite es colocado en un argumento a una funcion, los tipos proporcionados a dicha funcion deben cumplir con las restrcciones impuestas por el limite.

[traits]: traits.html

### Tipos de Tamano Dinamico TTD (Dynamically Sized Type)

Un tipo sin un tamano u alineacion conocidos de manera estatica. ([mas informacion][link])

[link]: ../nomicon/exotic-sizes.html#dynamically-sized-types-dsts

### Expresion

En ciencias de la computacion, una expresion es una combinacion de valores, constantes, variables operadores y funciones que evaluan a un unico valor. Por ejemplo, `2 + (3 * 4)` es una expresion que retorna el valor 14. Es importante mencionar que las expresiones pueden tener efectos secundarios. Por ejemplo, una funcion incluida en una expresion puede llevar a cabo otras acciones ademas de simplemente retornar un valor.

### Lenguaje de Programacion orientado a Expresiones

En los primeros lenguajes de programacion, las [expresiones][expression] y [sentencias][statement] eran dos categorias sintacticas diferentes: las expresiones poseian un valor y las sentencias llevaban a cabo acciones. Sin embargo, los lenguajes mas modernos hicieron a esta disticncion un poco borrosa, permitiendo a las expresiones llevar a cabo acciones y a las sentencias tener un valor. En un lenguaje de programacion orientado a expresiones, (casi) toda sentencia es una expresion y en consecuencia retorna un valor. Consecuentemente, dicho sentencias expresion pueden en si mismas formar parte de expresiones mas grandes.

[expression]: glossary.html#expression
[statement]: glossary.html#statement

### Sentencia

En ciencias de la computacion, una sentencia es el elemento libre mas pequeno en un lenguaje de programacion que comanda a una computadora a llevar a cabo una accion.
