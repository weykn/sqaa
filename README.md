# SUBLEQ abstract assembly

The **SUBLEQ abstract assembly** (*SQAA*) is a structured way to program the **SUBLEQ architecture** — a computer built on a single instruction: *Subtract and Branch if Less than or Equal to Zero*. While this minimalism is elegant, coding directly in raw `subleq` is tedious. **SQAA** adds abstraction layers that make programs easier to write and understand, without altering the simplicity of the underlying machine.

---

## Introduction to the SUBLEQ architecture

**SUBLEQ** stands for **“Subtract and Branch if Less than or Equal to Zero”**.
It is a type of **OISC** (*One-Instruction Set Computer*) architecture, meaning the entire machine operates using just **one instruction**.

#### Instruction format

Each SUBLEQ instruction consists of three operands:

```
subleq A, B, C
```

1. **\[B] = \[B] - \[A]**
   The value at memory location **A** is subtracted from the value at memory location **B**, and the result is stored back into **B**.

2. **If \[B] ≤ 0 → jump to C**
   If the result is less than or equal to zero, execution continues at the instruction located at **C**.
   Otherwise, execution continues at the next instruction.

#### Example

```
subleq 10, 11, 20
```

* Subtracts the value at address `10` from the value at address `11`, stores result at `11`.
* If `[11] ≤ 0`, jumps to instruction at address `20`. Otherwise continues sequentially.

#### Characteristics

* **Simplicity**: Only one instruction exists, making the hardware design minimal.
* **Turing-complete**: Despite its simplicity, any computation can be expressed in SUBLEQ.
* **Self-modifying code**: Since both instructions and data live in the same memory, modifying instructions at runtime is common.
* **Educational value**: Often used to teach fundamental principles of computer architecture, compilers, and minimalism.

#### Relation to other architectures

* Unlike **CISC** (x64, with many complex instructions) and **RISC** (ARM, with a reduced but still rich set of instructions), SUBLEQ strips computation down to the absolute minimum: *one instruction that can do everything*.

---

## Introduction to SUBLEQ Abstract Assembly (*SQAA*)

The **SUBLEQ abstract assembly (SQAA)** is a structured way of programming the SUBLEQ architecture by introducing **progressive abstraction layers**.

At its core, SUBLEQ is a **One-Instruction Set Computer (OISC)**: every operation reduces to the single instruction `subleq A, B, C`. This makes the machine extremely minimal and conceptually elegant, but it also means that programming directly in raw instructions can quickly become **cumbersome**. Even simple tasks like addition or copying require multiple `subleq` steps, making code difficult to write and read.

To make SUBLEQ programming more **human-friendly**, SQAA defines **layers of abstraction** on top of the base instruction. These layers provide conveniences such as named variables, macros for common operations, and higher-level constructs, while still compiling down to plain `subleq`.

The result is an approach that preserves the **simplicity of the architecture** while making it easier to **express, understand, and maintain** SUBLEQ programs.

---

## Compiler variables

Compile-time variables like this are possible due to the program always seeing 0 (memory address) as its start:

* `$` = pointer to next instruction as compile-time variable
* `$t`, `$e` = pointer to a free-to-use zero-cleared temporary memory field as compile-time variable automatically defined at the end of the program image
* `$r` = pointer to the result of a comparison as compile-time variable automatically defined at the end of the program image

---

## Layer 0 abstraction (base, no abstraction)

### **Labels**

A **label** marks a specific **memory location** so you can refer to it by name instead of by its numeric address.
Labels are often placed before data definitions or instructions and are followed by a colon (`:`).

They make your code **readable**, **modular**, and **easier to reference**, since you can use the label name in place of the actual address when loading or jumping to that part of memory.

---

### **The following assembler directives define or reserve data in memory:**

* `db`, *define byte* (8-bit): reserves and initializes 1 byte per value
* `dw`, *define word* (16-bit): reserves and initializes 2 bytes per value
* `dd`, *define double word* (32-bit): reserves and initializes 4 bytes per value
* `dq`, *define quad word* (64-bit): reserves and initializes 8 bytes per value
* `resb`, *reserve bytes*: reserves a specified number of bytes **without initialization**
* `resw`, *reserve words*: reserves a specified number of 16-bit words **without initialization**
* `resd`, *reserve double words*: reserves a specified number of 32-bit double words **without initialization**
* `resq`, *reserve quad words*: reserves a specified number of 64-bit quad words **without initialization**
* `equ`, define a compile-time/constant value, does not use space in memory and can't be modified at runtime

---

### **Example:**

```asm
msg:   db "Hello", 0      ; label 'msg' points to this string  
num:   dw 1234            ; 16-bit value  
ptr:   dd 0x400000        ; 32-bit value  
val:   dq 69999999        ; 64-bit value  
hey:   dw 444, 123, 5422  ; multiple 16-bit values  
buf:   resb 256           ; reserve 256 bytes for a buffer
fav:   equ 67             ; compile-time/constant value
```

**The example program:**

(direct, without labels)

```asm
subleq 18, 30, $    ; [30] = 0 - (-16) = 16
subleq 19, 30, $    ; [30] = 16 - (-8) = 24
subleq 31, 31, $    ; [31] = 0 - 0 = 0
subleq 30, $t, $    ; [$t] = 0 - 24 = -24
subleq $t, 31, $    ; [31] = 0 - (-24) = 24
subleq $t, $t, $    ; [$t] = -24 - (-24) = 0
subleq 0, 0, -1     ; halt (out-of-bounds)

db -16, -8          ; define bytes
```

or with labels:

```asm
subleq [value_1], [pos], $    ; [30] = 0 - (-16) = 16
subleq [value_2], [pos], $    ; [30] = 16 - (-8) = 24
subleq 31, 31, $    `         ; [31] = 0 - 0 = 0
subleq [pos], $t, $           ; [$t] = 0 - 24 = -24
subleq $t, 31, $              ; [31] = 0 - (-24) = 24
subleq $t, $t, $              ; [$t] = -24 - (-24) = 0
subleq 0, 0, -1               ; halt (out-of-bounds)

value_1: db -16
value_2: db -8
pos: equ 30
```

**The program shown performs some simple arithmetic, as represented in the C code below**

```c
int memory[64] = {0};
int temp = 0; // $t

// initialize constant values
memory[18] = -16;
memory[19] = -8;

// program
memory[30] -= memory[18];   // 0 - (-16) = 16
memory[30] -= memory[19];   // 16 - (-8) = 24
memory[31] -= memory[31];   // 0 - 0 = 0
temp -= memory[30];         // 0 - 24 = -24
memory[31] -= temp;         // 0 - (-24) = 24
temp -= temp;               // -24 - (-24) = 0

// memory[30] is 24
// memory[31] is 24
// temp ($t) is 0
```

---

## Layer 1 abstraction

* Function: `sub` (subtract), subleq without jump and values (A, B) swapped, which makes it a simple subtract
* Function: `hlt` (halt), does an out-of-bounds which halts execution
* Define (`equ`): define a compile-time/constant value
* Compile-time math: Inline calculations on compile-time data
* Function: `jmp` (jump), unconditional jump

**Unconditional jump**

Following command will jump to whatever value is in memory field 5

```asm
jmp [5] ; subleq $t, $t, 5
```

**The example program:**

```asm
sub [pos], [value_1] ; subleq 18, 30, $
sub [pos], [value_2] ; subleq 19, 30, $
sub [31], [31]       ; subleq 31, 31, $
sub [$t], [pos]      ; subleq 30, $t, $
sub [31], [$t]       ; subleq $t, 31, $
sub [$t], [$t]       ; subleq $t, $t, $
hlt                  ; subleq 0, 0, -1

; same as before
value_1: db -16
value_2: db -8
pos: equ 30
```

---

## Layer 2 abstraction

* Virtual literal, defined in data section automatically, can be used everywhere
* Function: `add_c` (Compile-time add, superseded by `add`), same as `sub` except it changes the state (positive/negative) of the given virtual literal (cannot be used with runtime data) when placing it in the data section, faster than `add_r` due to compile-time calculations
* Function: `mov` (Move), copies the content from source to destination
* Function: `add_r` (Runtime add, superseded by `add`), same as `sub` except it changes the state (positive/negative) of the given value, slower than `add_c` due to runtime calculations

**Virtual literals example:**

```asm
jmp 5        ; subleq $t, $t, 19
             ; (at position 19) db 5
```

or

```asm
sub [40], 4  ; subleq [55], [40], $
             ; (at position 55) db 4
```

**Runtime add example:**

Field 40 is 6, field 49 is 3

```asm
add_r [40], [49] ; subleq 49, $t, $ (subtract into $t for negative value)
                 ; subleq $t, 40, $ (subtract into target for positive value)
                 ; subleq $t, $t, $ (clear $t)
```
Field 40 is 9 now

**The example program:**

```asm
add_c [30], 16   ; subleq 18, 30, $
add_c [30], 8    ; subleq 19, 30, $

mov [31], [30]   ; subleq 31, 31, $ (clear target)
                 ; subleq 30, $t, $ (subtract into $t for negative value)
                 ; subleq $t, 31, $ (subtract into target for positive value)
                 ; subleq $t, $t, $ (subtract $t by $t to clear $t)

hlt              ; no changes

; db -16, -8 (automatically placed by virtual literal)
```

---

## Layer 3 abstraction

* Function: `add`, uses `add_c` if used on a virtual literal, otherwise uses `add_r`
#

?
