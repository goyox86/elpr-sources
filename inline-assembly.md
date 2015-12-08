% Ensamblador en linea

Para manipulaciones de extremadamente de bajo nivel y por razones de desempeno, uno podria desear controlar la CPU de manera directa. Para esto Rust soporta el uso de ensamblador en linea a traves de la macro `asm!`. La sintaxis es parecida a la de GCC & Clang:

```ignore
asm!(plantilla ensamblador
   : operandos de salida
   : operandos de entrada
   : clobbers
   : opciones
   );
```

Cualquier uso de `asm` esta protegido por puertas de facilidades (requiere `#![feature(asm)]` en el crate ) y por supuesto requiere de un bloque `unsafe`.

> **Nota**: los ejemplos proporcionados aqui estan escritos en ensablador x86/x86-64 assembly, pero
> todas las plataformas estan soportadas.

## Plantilla ensablador

La `plantilla ensamblador` es el unico parametro mandatorio y debe ser un literal de cadena de caracteres (e.j. `""`)

```rust
#![feature(asm)]

#[cfg(any(target_arch = "x86", target_arch = "x86_64"))]
fn foo() {
    unsafe {
        asm!("NOP");
    }
}

// otras plataformas
#[cfg(not(any(target_arch = "x86", target_arch = "x86_64")))]
fn foo() { /* ... */ }

fn main() {
    // ...
    foo();
    // ...
}
```

(Los `feature(asm)` y `#[cfg]`s seran omitidos de ahora en adelante.)

Los operandos de salida, operandos de entrada, clobbers y opciones son todos opcionales pero debes agregar el numero correcto de `:` si deseas saltarlos:

```rust
# #![feature(asm)]
# #[cfg(any(target_arch = "x86", target_arch = "x86_64"))]
# fn main() { unsafe {
asm!("xor %eax, %eax"
    :
    :
    : "{eax}"
   );
# } }
```

Los espacios en blanco no son significativos:

```rust
# #![feature(asm)]
# #[cfg(any(target_arch = "x86", target_arch = "x86_64"))]
# fn main() { unsafe {
asm!("xor %eax, %eax" ::: "{eax}");
# } }
```

## Operandos

Los operandos de entrada y salida siguen el mismo formato: `restriccion1"(expr1), "restriccion2"(expr2), ..."`. Las expresiones de operandos de salida deben ser valores lvalue mutables, o valores no asignados todavia:

```rust
# #![feature(asm)]
# #[cfg(any(target_arch = "x86", target_arch = "x86_64"))]
fn add(a: i32, b: i32) -> i32 {
    let c: i32;
    unsafe {
        asm!("add $2, $0"
             : "=r"(c)
             : "0"(a), "r"(b)
             );
    }
    c
}
# #[cfg(not(any(target_arch = "x86", target_arch = "x86_64")))]
# fn add(a: i32, b: i32) -> i32 { a + b }

fn main() {
    assert_eq!(add(3, 14159), 14162)
}
```

Sin embargo, Si desearas usar operandos reales en esta posicion, es obligatorio colocar llaves `{}` alrededor del registro que deseas, y tambien estas obligado a colocar el tamano especifico del operando. Esto es muy util para programacion de muy bajo nivel, para la que es importante cual registro es el que sea usa.

```rust
# #![feature(asm)]
# #[cfg(any(target_arch = "x86", target_arch = "x86_64"))]
# unsafe fn read_byte_in(puerto: u16) -> u8 {
let resultado: u8;
asm!("in %dx, %al" : "={al}"(resultado) : "{dx}"(puerto));
resultado
# }
```

## Clobbers

Algunas instrucciones modifican registros los cuales de otra forma hubieran mantenido valores diferentes es por ello que usamos las lista de clobbers para indicar al compilador a que no asuma que ningun valor cargado en dichos registros se mantrendra valido.

```rust
# #![feature(asm)]
# #[cfg(any(target_arch = "x86", target_arch = "x86_64"))]
# fn main() { unsafe {
// Colocando el valor 0x200 en eax
asm!("mov $$0x200, %eax" : /* sin salidas */ : /* sin entradas */ : "{eax}");
# } }
```

Los registros de entrada y salida no necesitan ser listados debido a que esa informacion ya esta comunicada por las restricciones dadas. De lo contrario, cualquier otro registro usado ya sea de manera explicita o implicita debe ser listado.

Si el ensamblador cambia el registro de codigo de condicion `cc` debe ser especificado como uno de los clobbers. Similarmente, si el ensamblador modifica memoria, `memory` debe ser tambien especificado.

## Options

La ultima seccion, `opciones` es especifca de Rust. El formato es una lista de cadenas de caracteres separadas por comma (e.j: `:"foo", "bar", "baz"`). Es usado para especificar informacion extra acerca del ensamblador:

Las opciones actualmente validad son:

1. *volatile* - especifacr esto es analogo a `__asm__ __volatile__ (...)` en gcc/clang.
2. *alignstack* - ciertas instrucciones esperan que la pila este alineada de cierta manera (e.j. SSE) y especificar esto le indica al compilador que inserte su codigo de alinemiento de pila usual.
3. *intel* - uso de la sintaxis de intel en lugar de la AT&T.

```rust
# #![feature(asm)]
# #[cfg(any(target_arch = "x86", target_arch = "x86_64"))]
# fn main() {
let resultado: i32;
unsafe {
   asm!("mov eax, 2" : "={eax}"(resultado) : : : "intel")
}
println!("eax es actualmente {}", resultado);
# }
```

## Mas Informacion

La implementacion actual de la macro `asm!` es un binding directo a las [expresiones ensamblador en linea de LLVM][llvm-docs], asi que asegurate de echarle un vistazo a [su documentacion][llvm-docs] para mayor informacion acerca de clobbers, restricciones, etc.

[llvm-docs]: http://llvm.org/docs/LangRef.html#inline-assembler-expressions
