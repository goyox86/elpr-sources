% Ensamblador en linea

Para manipulaciones de muy bajo nivel y por razones de desempeño, uno podría desear controlar la CPU de manera directa. Para esto Rust soporta el uso de ensamblador en linea a través de la macro `asm!`. La sintaxis es parecida a la de GCC & Clang:

```ignore
asm!(plantilla ensamblador
   : operandos de salida
   : operandos de entrada
   : clobbers
   : opciones
   );
```

Cualquier uso de `asm` esta protegido por puertas de feature (requiere `#![feature(asm)]` en el crate) y por supuesto requiere de un bloque `unsafe`.

> **Nota**: los ejemplos proporcionados aquí están escritos en ensamblador x86/x86-64, pero
> todas las plataformas están soportadas.

## Plantilla ensamblador

La `plantilla ensamblador` es el único parámetro mandatorio y debe ser un literal de cadena de caracteres (e.j. `""`)

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

(Los `feature(asm)` y `#[cfg]`s son omitidos de ahora en adelante.)

Los operandos de salida, operandos de entrada, clobbers y opciones son todos opcionales pero debes agregar el numero correcto de `:` en caso de que desees saltarlos:

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

Los operandos de entrada y salida siguen el mismo formato: `restriccion1"(expr1), "restriccion2"(expr2), ..."`. Las expresiones de operandos de salida deben ser valores lvalue mutables, o valores aun no asignados:

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

Sin embargo, Si desearas usar operandos reales en esta posición, seria obligatorio colocar llaves `{}` alrededor del registro que deseas, y también estarías obligado a colocar el tamaño especifico del operando. Lo anterior es muy util para programación de muy bajo nivel, para la que es importante cual registro es el que sea usa.

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

Algunas instrucciones modifican registros los cuales de otra forma hubieran mantenido valores diferentes es por ello que usamos las lista de clobbers para indicar al compilador a que no asuma que ningún valor cargado en dichos registros se mantendrá valido.

```rust
# #![feature(asm)]
# #[cfg(any(target_arch = "x86", target_arch = "x86_64"))]
# fn main() { unsafe {
// Colocando el valor 0x200 en eax
asm!("mov $$0x200, %eax" : /* sin salidas */ : /* sin entradas */ : "{eax}");
# } }
```

Los registros de entrada y salida no necesitan ser listados puesto a que esa información ya esta comunicada por las determinadas restricciones. De lo contrario, cualquier otro registro usado ya sea de manera explicita o implícita debe ser listado.

Si el ensamblador cambia el registro de código de condición `cc` debe ser especificado como uno de los clobbers. Similarmente, si el ensamblador modifica memoria, `memory` debe ser también especificado.

## Opciones

La ultima sección, `opciones` es específica de Rust. El formato es una lista de cadenas de caracteres separadas por comma (e.j: `:"foo", "bar", "baz"`). Es usado para especificar información extra acerca del ensamblador:

Las opciones validas actualmente son:

1. *volatile* - especificar esto es analogo a `__asm__ __volatile__ (...)` en gcc/clang.
2. *alignstack* - ciertas instrucciones esperan que la pila este alineada de cierta manera (e.j. SSE), especificar esto le indica al compilador que inserte su código de alineamiento de pila usual.
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

## Mas Información

La implementación actual de la macro `asm!` es un binding directo a las [expresiones ensamblador en linea de LLVM][llvm-docs], así que asegurate de echarle un vistazo a [su documentación][llvm-docs] para mayor información acerca de clobbers, restricciones, etc.

[llvm-docs]: http://llvm.org/docs/LangRef.html#inline-assembler-expressions
