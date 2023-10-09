# CISC REGISTER

## Structure
Register CISC architecture uses 1-6 bytes for instructions:
There is 7 registers:
- `FR` Flag register with least significant bits representing flags - CF, ZF, OF, SF (not a general-purpose register)
- `R00`, `R01`, `R02`, `R03` with L and H bytes each
- `BP` - base pointer
- `SP`- stack pointer
- `OC` - Program counter (can't be directly affected with arithmetical instructions)

Instructions look like this:
- `| 3-bit style specifier | 5-bit opcode | `
- `| 3-bit style specifier | 5-bit opcode | 3-bit register |`
- `| 3-bit style specifier | 5-bit opcode | 16-bit constant |`
- `| 3-bit style specifier | 5-bit opcode | 3-bit register | 3-bit register |`
- `| 3-bit style specifier | 5-bit opcode | 3-bit register | 16-bit constant |`

Offset instruction:
- `| 3-bit style specifier | 5-bit opcode | 3-bit register | 16-bit constant |`
- `| 3-bit style specifier | 5-bit opcode | 3-bit register | 16-bit constant | 3-bit register |`
- `| 3-bit style specifier | 5-bit opcode | 3-bit register | 3-bit register | 16-bit constant |`

## Instructions

# HALT
`00000000` - 1 byte

Disables cpu until power is cycled

# MOV `%REG`, `$IMM`
`10000000` - 4 bytes

Copy the value `$IMM` into `%REG`: `%REG = `$IMM``

# MOV `%REG1`, `%REG2`
`01100000` - 2 btyes

Copy the value from `%REG2` into `%REG1`: `%REG1 = `%REG2``

# MOV `%REG1`, `[%REG2]`
`01100001` - 2 bytes

Copy the value at memory location `[%REG2]` into `%REG1`: `%REG1 = [%REG2]`

# MOV `%REG1`, `[%REG2+$OFF]`
`10100000` - 4 bytes

Copy the value at memory location `[%REG2 + `$OFF`]` into `%REG1`: `%REG1 = [%REG2 + `$OFF`]`

# MOV `[%REG1]`, `%REG2`
`01100010` - 2 bytes

Copy the value from `%REG2` into memory location `[%REG1]`: `[%REG1]` = `%REG2``

# MOV `[%REG1]`, `$IMM`
`10000001` - 4 bytes

Copy the value `$IMM` into memory location `[%REG1]`: `[%REG1]` = `$IMM``

# MOV `[%REG1+$OFF]`, `%REG2`
`10100001` - 4 bytes

Copy the value from `%REG2` into memory location `[%REG1 + `$OFF`]`: `[%REG1 + `$OFF`]` = `%REG2``

# MOV `[%REG1+$OFF]`, `$IMM`
`11000000` - 6 bytes

Copy the value `$IMM` into memory location `[%REG1 + `$OFF`]`: `[%REG1 + `$OFF`]` = `$IMM``

# PUSH `%REG`
`00000001` - 2 bytes

Pushes the value of `%REG` on the top of the memory stack

# PUSH `$IMM`
`01000000` - 3 bytes

Pushes the value `$IMM` on the top of the memory stack

# POP `%REG`
`00000010` - 2 bytes

Pops the value from the stack and puts it into `%REG`

# ENTER `$IMM`
`01000001` - 3 bytes

Replaces three instructions on moving the stack further down when
calling a new procedure: `push %ebp / mov %ebp, %esp / sub %esp, $num`.
Notice that you still have to push the other registers onto the memory stack

# LEAVE
`00100000` - 1 byte

Replaces two instructions when returning to the
previous procedure's stack frame: `mov %esp, %ebp / pop %ebp`

# ADD `%REG1`, `[%REG2]`
`01100011` - 2 bytes

Add the value at `[%REG2]` to `%REG1`

`%REG1 += [%REG2]`

# ADD `%REG1`, `%REG2`
`01100100` - 2bytes

Add the value at `%REG2` to `%REG1`

`%REG1 += %REG2`

# ADD `%REG1`, `[%REG2+$OFF]`
`10100010` - 4 bytes

Add the value at `[%REG2+$OFF]` to `%REG1`

`%REG1 += [%REG2+$OFF]`

# ADD `[%REG1]`, `%REG2`
`01100101` - 2 bytes

Add the value at `%REG2` to `[%REG1]`

`[%REG1] += %REG2`

# SUB `%REG1`, `[%REG2]`
`01100110` - 2 bytes

Subtract the value at `[%REG2]` from `%REG1`

`%REG1 -= [%REG2]`

# SUB `%REG1`, `%REG2`
`01100111` - 2bytes

Subtract the value at `%REG2` from `%REG1`

`%REG1 -= %REG2`

# SUB `%REG1`, `[%REG2+$OFF]`
`10100010` - 4 bytes

Subtract the value at `[%REG2+$OFF]` from `%REG1`

`%REG1 -= [%REG2+$OFF]`

# SUB `[%REG1]`, `%REG2`
`01101000` - 2 bytes

Subtract the value at `%REG2` from `[%REG1]`

`[%REG1] -= %REG2`

# INC `%REG`
`00000011` - 2 bytes

Increment `%REG` by one
`%REG += 1`

# INC `[%REG]`
`00000100` - 2 bytes

Increment `[%REG]` by one
`[%REG] += 1`

# INC `[%REG+$OFF]`
`10000010` - 4 bytes

Increment `[%REG+$OFF]` by one
`[%REG+$OFF] += 1`


# DEC `%REG`
`00000101` - 2 bytes

Decrement `%REG` by one
`%REG += 1`

# DEC `[%REG]`
`00000110` - 2 bytes

Decrement `[%REG]` by one
`[%REG] += 1`

# DEC `[%REG+$OFF]`
`10000011` - 4 bytes

Decrement `[%REG+$OFF]` by one
`[%REG+$OFF] += 1`


# MUL `%REG1`, `%REG2`
`01101001` - 2 bytes

Multiply `%REG1` by `%REG2` and store the result in `%REG1`
`%REG1 *= %REG2`

# MUL `%REG1`, `[%REG2]`
`01101010` - 2 bytes

Multiply `%REG1` by `[%REG2]` and store the result in `%REG1`
`%REG1 *= [%REG2]`

# MUL `[%REG1]`, `%REG2`
`01101011` - 2 bytes

Multiply `[%REG1]` by `%REG2` and store the result in `[%REG1]`
`[%REG1] *= %REG2`

# MUL `%REG`, `$IMM`
`10000100` - 4 bytes

Multiply `%REG1` by `$IMM` and store the result in `%REG1`
`%REG1 *= $IMM`

# MUL `%REG1`, `[%REG2+$OFF]`
`10100100` - 4 bytes

Multiply `%REG1` by `[%REG2+$OFF]` and store the result in `%REG1`
`%REG1 *= [%REG2+$OFF]`

# DIV `%REG1`, `%REG2`
`01101100` - 2 bytes

Divide `%REG1` by `%REG2` and store the result in `%REG1`
`%REG1 *= %REG2`

# DIV `%REG1`, `[%REG2]`
`01101101` - 2 bytes

Divide `%REG1` by `[%REG2]` and store the result in `%REG1`
`%REG1 *= [%REG2]`

# DIV `[%REG1]`, `%REG2`
`01101110` - 2 bytes

Divide `[%REG1]` by `%REG2` and store the result in `[%REG1]`
`[%REG1] *= %REG2`

# DIV `%REG`, `$IMM`
`10000101` - 4 bytes

Divide `%REG1` by `$IMM` and store the result in `%REG1`
`%REG1 *= $IMM`

# DIV `%REG1`, `[%REG2+$OFF]`
`10100101` - 4 bytes

Divide `%REG1` by `[%REG2+$OFF]` and store the result in `%REG1`
`%REG1 *= [%REG2+$OFF]`

# AND `%REG1`, `%REG2`
`01101111` - 2 bytes

Binary and `%REG1` and `%REG2`, storing the result into `%REG1`

`%REG1 &= %REG2`

# AND `%REG1`, `[%REG2]`
`01110000` - 2 bytes

Binary and `%REG1` and `[%REG2]`, storing the result into `%REG1`

`%REG1 &= [%REG2]`

# OR `%REG1`, `%REG2`
`01110001` - 2 bytes

Binary or `%REG1` and `%REG2`, storing the result into `%REG1`

`%REG1 |= %REG2`

# OR `%REG1`, `[%REG2]`
`01110010` - 2 bytes

Binary or `%REG1` and `[%REG2]`, storing the result into `%REG1`

`%REG1 |= [%REG2]`

# XOR `%REG1`, `%REG2`
`01110011` - 2 bytes

Binary xor `%REG1` and `%REG2`, storing the result into `%REG1`

`%REG1 ^= %REG2`

# XOR `%REG1`, `[%REG2]`
`01110100` - 2 bytes

Binary xor `%REG1` and `[%REG2]`, storing the result into `%REG1`

`%REG1 ^= [%REG2]`

# NOT `%REG1`
`00000111`  - 2 bytes

Binary not `%REG1`

`%REG1 = ~%REG1`

# NOT `[%REG]`
`00001000` - 2 bytes

Binary not `[%REG1]`

`[%REG1] = ~[%REG1]`

 
# LSH `%REG`, `$IMM`
`10000110` - 2 bytes

Shift `%REG` left logically by `$IMM` bits

`%REG = %REG << $IMM`

# LSH `[%REG]`, `$IMM`
`10000111` - 2 bytes

Shift `[%REG]` left logically by `$IMM` bits

`[%REG] = [%REG] << $IMM`

# LSH `[%REG+$OFF]`, `$IMM`
`11000001` - 4 bytes

Shift `[%REG+$OFF]` left logically by `$IMM` bits

`[%REG+$OFF] = [%REG+$OFF] << $IMM`
 
# RSH `%REG`, `$IMM`
`10001000` - 2 bytes

Shift `%REG` right logically by `$IMM` bits

`%REG = %REG << $IMM`

# RSH `[%REG]`, `$IMM`
`10001001` - 2 bytes

Shift `[%REG]` right logically by `$IMM` bits

`[%REG] = [%REG] << $IMM`

# RSH `[%REG+$OFF]`, `$IMM`
`11000010` - 4 bytes

Shift `[%REG+$OFF]` right logically by `$IMM` bits

`[%REG+$OFF] = [%REG+$OFF] << $IMM`

# CALL LABEL
# CALL `[`$IMM`]`
# CALL `[%REG]`
# CALL `[%REG+$OFF]`
 			
# RET
 
# CMP `%REG1`, `%REG2`
 
# CMP `%REG`, `$IMM`
 
# CMP `%REG1`, `[%REG2]`
 
# CMP `%REG1`, `[%REG2+$OFF]`
 
# TEST `%REG1`, `%REG2`
 
# TEST `%REG1`, `[%REG2]`
# TEST `%REG1`, `[%REG2+$OFF]`
# TEST `[%REG1]`, `%REG2`
 
# JMP `[$OFF]`
 
# JMP `[%REG]`
 
# JMP `[%REG+$OFF]`
 
# JE `$OFF`
 
# JNE `$OFF`
 
# JG `$OFF`
 
# JGE `$OFF`
 
# JL `$OFF`
 
# JLE `$OFF`
 			
# IN `$NUM`, `$PORT`
# IN `[%REG]`, `$PORT`
# IN `[%REG+$OFF]`, `$PORT`
 			
# OUT `$PORT`, `$IMM`
# OUT `$PORT`, `%REG`
# OUT `$PORT`, `[%REG]`
# OUT `$PORT`, `[%REG+$OFF]`
# LOAD4 `[%REG]`
# STORE4 `[%REG]`
# ADD4 `[%REG1], [%REG2]`
# SUB4 `[%REG1], [%REG2]`
# MUL4 `[%REG1], [%REG2]`
# DIV4 `[%REG1], [%REG2]`
# CMP4 `[%REG1], [%REG2]`
# TEST4 `[%REG1], [%REG2]`
