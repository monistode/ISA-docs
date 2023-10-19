# CISC REGISTER

## Structure
Register CISC architecture uses 1-6 bytes for instructions:
There is 7 registers:
- `FR` - `8 bits` - Flag register with least significant bits representing flags - CF, ZF, OF, SF (not a general-purpose register)
- `R00`, `R01`, `R02`, `R03` - `16 bits each` - with L and H bytes each
- `M00`, `M01`, `M02` - `32 bits each` - SIMD registers wirh (`H, L`) being the first / second and third / fourth bytes respectively
- `BP` - `16 bits` - base pointer
- `SP` - `16 bits` - stack pointer
- `PC` - `16 bits` - Program counter (can't be directly affected with arithmetical instructions)

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

Copy the value `$IMM` into `%REG`: `%REG` = `$IMM`

# MOV `%REG1`, `%REG2`
`01100000` - 2 btyes

Copy the value from `%REG2` into `%REG1`: `%REG1` = `%REG2`

YOU COULD DO `MOV %M00H %R00` / `MOV %R00 %M00H` / `MOV %M00L %R00` / `MOV %R00 %M00L`

# MOV `%REG1`, `[%REG2]`
`01100001` - 2 bytes

Copy the value at memory location `[%REG2]` into `%REG1`: `%REG1 = [%REG2]`

# MOV `%REG1`, `[%REG2+$OFF]`
`10100000` - 4 bytes

Copy the value at memory location `[%REG2 + $OFF]` into `%REG1`: `%REG1 = [%REG2 + $OFF]`

# MOV `[%REG1]`, `%REG2`
`01100010` - 2 bytes

Copy the value from `%REG2` into memory location `[%REG1]`: `[%REG1] = %REG2`

# MOV `[%REG1]`, `$IMM`
`10000001` - 4 bytes

Copy the value `$IMM` into memory location `[%REG1]`: `[%REG1] = $IMM`

# MOV `[%REG1+$OFF]`, `%REG2`
`10100001` - 4 bytes

Copy the value from `%REG2` into memory location `[%REG1 + $OFF]`: `[%REG1 + $OFF] = %REG2`

# MOV `[%REG1+$OFF]`, `$IMM`
`11000000` - 6 bytes

Copy the value `$IMM` into memory location `[%REG1 + $OFF]`: `[%REG1 + $OFF] = $IMM`

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
`%REG -= 1`

# DEC `[%REG]`
`00000110` - 2 bytes

Decrement `[%REG]` by one
`[%REG] -= 1`

# DEC `[%REG+$OFF]`
`10000011` - 4 bytes

Decrement `[%REG+$OFF]` by one
`[%REG+$OFF] -= 1`


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
`%REG1 /= %REG2`

# DIV `%REG1`, `[%REG2]`
`01101101` - 2 bytes

Divide `%REG1` by `[%REG2]` and store the result in `%REG1`
`%REG1 /= [%REG2]`

# DIV `[%REG1]`, `%REG2`
`01101110` - 2 bytes

Divide `[%REG1]` by `%REG2` and store the result in `[%REG1]`
`[%REG1] /= %REG2`

# DIV `%REG`, `$IMM`
`10000101` - 4 bytes

Divide `%REG1` by `$IMM` and store the result in `%REG1`
`%REG1 /= $IMM`

# DIV `%REG1`, `[%REG2+$OFF]`
`10100101` - 4 bytes

Divide `%REG1` by `[%REG2+$OFF]` and store the result in `%REG1`
`%REG1 /= [%REG2+$OFF]`

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

`%REG = %REG >> $IMM`

# RSH `[%REG]`, `$IMM`
`10001001` - 2 bytes

Shift `[%REG]` right logically by `$IMM` bits

`[%REG] = [%REG] >> $IMM`

# RSH `[%REG+$OFF]`, `$IMM`
`11000010` - 4 bytes

Shift `[%REG+$OFF]` right logically by `$IMM` bits

`[%REG+$OFF] = [%REG+$OFF] >> $IMM`

# CALL LABEL
`01000010` - 3 bytes

Jumps to label

`%PC = LABEL`

# CALL `$IMM`
`01000010` - 3 bytes

Jumps to `$IMM` address

`%PC = $IMM`

# CALL `%REG`
`00001001` - 2 bytes

Jumps to value at register

`%PC = %REG`

# CALL `%REG+$OFF`
`10001001` - 4 bytes

Jumps to `%REG + $OFF`

`%PC = %REG + $OFF`

# RET
`00100001` - 1 byte

Pops the value off the memory stack and puts it into `%PC`
 
# CMP `%REG1`, `%REG2`
`01110101` - 2 bytes

DOES NOT MUTATE THE REGISTERS
Compare `%REG1` and `%REG2` by subtractiong them. Sets the flags appropriately
 
# CMP `%REG`, `$IMM`
`10001011` - 4 bytes

DOES NOT MUTATE THE REGISTERS
Compare `%REG1` and `$IMM` by subtractiong them. Sets the flags appropriately
 
# CMP `%REG1`, `[%REG2]`
`10001011` - 2 bytes

DOES NOT MUTATE THE REGISTERS
Compare `%REG1` and `[%REG2]` by subtractiong them. Sets the flags appropriately

 
# CMP `%REG1`, `[%REG2+$OFF]`
`10100110` - 4 bytes

DOES NOT MUTATE THE REGISTERS
Compare `%REG1` and `[%REG2 + $OFF]` by subtractiong them. Sets the flags appropriately
 
# TEST `%REG1`, `%REG2`
`01110111` - 2 bytes

DOES NOT MUTATE THE REGISTERS
Performs bitwise and on `%REG1` and `%REG2`. Sets the flags accordingly. 
 
# TEST `%REG1`, `[%REG2]`
`01111000` - 2 bytes

DOES NOT MUTATE THE REGISTERS
Performs bitwise and on `%REG1` and `[%REG2]`. Sets the flags accordingly. 

# TEST `%REG1`, `[%REG2+$OFF]`

`10100111` - 4 bytes
DOES NOT MUTATE THE REGISTERS
Performs bitwise and on `%REG1` and `[%REG2+$OFF]`. Sets the flags accordingly. 

# JMP `[$OFF]`
`01000011` - 2 bytes

Sets `%PC` to `$OFF`, effectively jumping to it.

`%PC = $OFF`

 
# JMP `%REG`
`00001010`
 
Sets `%PC` to `%REH`, effectively jumping to it.

`%PC = %REG`

# JMP `%REG+$OFF`
`10001100` - 4 bytes
 
Sets `%PC` to `%REG + $OFF`, effectively jumping to it.

`%PC = %REG + $OFF`

# JE `$OFF`
`01000100` - 3 bytes

If `ZF == 1`, Sets `%PC` to `$OFF`, effectively jumping to that address
 
# JNE `$OFF`
`01000101` - 2 bytes

If `ZF == 0`, Sets `%PC` to `$OFF`, effectively jumping to that address
 
# JG `$OFF`
`01000110` - 3 bytes

If `ZF == 0 && SF == OF`, Sets `%PC` to `$OFF`, effectively jumping to that address
 
# JGE `$OFF`
`01000111` - 3 bytes

If `Cf == 0`, Sets `%PC` to `$num`, effectively jumping to that address
 
# JL `$OFF`
`01001000` - 3 bytes 

If `SF != OF`, Sets `%PC` to `$OFF`, effectively jumping to that address

# JLE `$OFF`
`01001001` - 3 bytes

If `CF == 1 || ZF == 1`, Sets `%PC` to `$num`, effectively jumping to that address
 			
# IN `%REG`, `$PORT`
`10001101` - 3 bytes

Transfers data from the device at port `$PORT` to `%REG`

# IN `[%REG]`, `$PORT`
`10001110` - 3 bytes

Transfers data from the device at port `$PORT` to `[%REG]`

# IN `[%REG+$OFF]`, `$PORT`
`11000011` - 5 bytes

Transfers data from the device at port `$PORT` to `[%REG + $OFF]`
 			
# OUT `$PORT`, `$IMM`
`10001111` - 4 bytes

Puts the value `$IMM` on to the port `$PORT`


# OUT `$PORT`, `%REG`
`10001111` - 3 bytes

Puts the value from `%REG` on to the port `$PORT`

# OUT `$PORT`, `[%REG]`
`10010000` - 3 bytes

Puts the value from `[%REG]` on to the port `$PORT`

# OUT `$PORT`, `[%REG+$OFF]`
`11000100` - 5 bytes

Puts the value from `[%REG + $OFF]` on to the port `$PORT`

# LOAD4 `%SIMDREG`, `$ADDR`
`00001011` - 4 bytes

Load 4 bytes from memory into `%SIMDREG`, starting at `$ADDR`

# STORE4 `%SIMDREG`, `$ADDR`
`00001100` - 4 bytes

Store 4 bytes from `%SIMDREG` into `[$ADDR]`

# ADD4 `%SIMDREG1, %SIMDREG2`
`01111010` - 2 bytes

Add two SIMD registers (`%SIMDREG1 * %SIMDREG2`), storing the result into `%SIMDREG1`.

`%SIMDREG1 += %SIMDREG2`

# SUB4 `%SIMDREG1, %SIMDREG2`
`01111011` - 2 bytes

Subtract two SIMD registers (`%SIMDREG1 - %SIMDREG2`), storing the result into `%SIMDREG1`.

`%SIMDREG1 -= %SIMDREG2`

# MUL4 `%SIMDREG1, %SIMDREG2`
`01111101` - 2 bytes

Multiply two SIMD registers (`%SIMDREG1 * %SIMDREG2`), storing the result into `%SIMDREG1`.

`%SIMDREG1 *= %SIMDREG2`

# DIV4 `%SIMDREG1, %SIMDREG2`
`01111101` - 2 bytes

Divide two SIMD registers (`%SIMDREG1 / %SIMDREG2`), storing the result into `%SIMDREG1`.

`%SIMDREG1 /= %SIMDREG2`

# NOP
`00100010` - 1 byte

Do nothing for 1 cycle
