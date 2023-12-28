# CISC REGISTER

## Structure

#### THE ISA IS BIG ENDIAN

Register CISC architecture uses 1-4 bytes for instructions:
There is 7 registers:
- `FR` - `8 bits` - Flag register with least significant bits representing flags - CF, ZF, OF, SF (not a general-purpose register)
- `R00`, `R01`, `R02`, `R03` - `16 bits each` - with L and H bytes each
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

One stack - the memory stack.

Command sizes according to the 3-bit specifier:

- `000` - 1 byte if halt, 2 bytes otherwise, 4 bytes if SIMD
- `100` - 4 bytes
- `011` - 2 bytes
- `101` - 5 bytes
- `110` - 6 bytes
- `010` - 3 bytes
- `001` - 1 byte

IO:
- multiple serial ports
- 16-bit address space for port numbers
- simple in and out operations

Arithmetic operations:
- signed - look at `OF` to determine whether there was overflow; and at `SF` to determine the sign of the result (regardless of overflow)
- unsigned - look at `CF` to determine whether there was overflow

Only division is a strictly signed operation

## Instructions

# HALT
`00000000` - 1 byte

Disables cpu until power is cycled

# MOV `%REG`, `$IMM`
`10000000` - 4 bytes

Copy the value `$IMM` into `%REG`: `%REG` = `$IMM`

# MOV `%REG1`, `%REG2`
`01100000` - 2 bytes

Copy the value from `%REG2` into `%REG1`: `%REG1` = `%REG2`

YOU COULD DO `MOV %M00H %R00` / `MOV %R00 %M00H` / `MOV %M00L %R00` / `MOV %R00 %M00L`

# MOV `%REG1`, `[%REG2]`
`01100001` - 2 bytes

Copy the value at memory location `[%REG2]` into `%REG1`: `%REG1 = [%REG2]`

# MOV `%REG1`, `[%REG2+$OFFSET]`
`10100000` - 4 bytes

Copy the value at memory location `[%REG2 + $OFFSET]` into `%REG1`: `%REG1 = [%REG2 + $OFFSET]`

# MOV `[%REG1]`, `%REG2`
`01100010` - 2 bytes

Copy the value from `%REG2` into memory location `[%REG1]`: `[%REG1] = %REG2`

# MOV `[%REG1]`, `$IMM`
`10000001` - 4 bytes

Copy the value `$IMM` into memory location `[%REG1]`: `[%REG1] = $IMM`

# MOV `[%REG1+$OFFSET]`, `%REG2`
`10100001` - 5 bytes

Copy the value from `%REG2` into memory location `[%REG1 + $OFFSET]`: `[%REG1 + $OFFSET] = %REG2`

# MOV `[%REG1+$OFFSET]`, `$IMM`
`11000000` - 6 bytes

Copy the value `$IMM` into memory location `[%REG1 + $OFFSET]`: `[%REG1 + $OFFSET] = $IMM`

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
calling a new procedure: `push %BP / mov %BP, %SP / sub %SP, $IMM`.
Notice that you still have to push the other registers onto the memory stack

# LEAVE
`00100000` - 1 byte

Replaces two instructions when returning to the
previous procedure's stack frame: `mov %SP, %BP / pop %BP`

# ADD `%REG1`, `[%REG2]`
`01100011` - 2 bytes

Add the value at `[%REG2]` to `%REG1`

Reset the flag register, then set the `CF` to 1 if the result has an extra carry, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

Note: the implementation isn't at all working as expected, as the `change_flag_result` function takes in the truncated result.

Note: further operations' TODOs say "as if it's addition"; `OF` isn't affected how you'd expect it to with addition for them - we compute (most_significant_bit_a $=$ most_significant_bit_b and most_significant_bit_result $\ne$ most_significant_bit_a). [See implementation](https://github.com/ucu-computer-science/Hardware-Simulator-and-Assembler/blob/master/modules/functions.py#L430)

`%REG1 += [%REG2]`

# ADD `%REG1`, `%REG2`
`01100100` - 2 bytes

Add the value at `%REG2` to `%REG1`

Reset the flag register, then set the `CF` to 1 if the result has an extra carry, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

Note: the implementation isn't at all working as expected, as the `change_flag_result` function takes in the truncated result.

Note: further operations' TODOs say "as if it's addition"; `OF` isn't affected how you'd expect it to with addition for them - we compute (most_significant_bit_a $=$ most_significant_bit_b and most_significant_bit_result $\ne$ most_significant_bit_a). [See implementation](https://github.com/ucu-computer-science/Hardware-Simulator-and-Assembler/blob/master/modules/functions.py#L430)

`%REG1 += %REG2`

# ADD `%REG1`, `[%REG2+$OFFSET]`
`10100010` - 4 bytes

Add the value at `[%REG2+$OFFSET]` to `%REG1`

Reset the flag register, then set the `CF` to 1 if the result has an extra carry, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

Note: the implementation isn't at all working as expected, as the `change_flag_result` function takes in the truncated result.

Note: further operations' TODOs say "as if it's addition"; `OF` isn't affected how you'd expect it to with addition for them - we compute (most_significant_bit_a $=$ most_significant_bit_b and most_significant_bit_result $\ne$ most_significant_bit_a). [See implementation](https://github.com/ucu-computer-science/Hardware-Simulator-and-Assembler/blob/master/modules/functions.py#L430)

`%REG1 += [%REG2+$OFFSET]`

# ADD `[%REG1]`, `%REG2`
`01100101` - 2 bytes

Add the value at `%REG2` to `[%REG1]`

Reset the flag register, then set the `CF` to 1 if the result has an extra carry, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

Note: the implementation isn't at all working as expected, as the `change_flag_result` function takes in the truncated result.

Note: further operations' TODOs say "as if it's addition"; `OF` isn't affected how you'd expect it to with addition for them - we compute (most_significant_bit_a $=$ most_significant_bit_b and most_significant_bit_result $\ne$ most_significant_bit_a). [See implementation](https://github.com/ucu-computer-science/Hardware-Simulator-and-Assembler/blob/master/modules/functions.py#L430)

`[%REG1] += %REG2`

# SUB `%REG1`, `[%REG2]`
`01100110` - 2 bytes

Subtract the value at `[%REG2]` from `%REG1`

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

`%REG1 -= [%REG2]`

# SUB `%REG1`, `%REG2`
`01100111` - 2bytes

Subtract the value at `%REG2` from `%REG1`

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

`%REG1 -= %REG2`

# SUB `%REG1`, `[%REG2+$OFFSET]`
`10100011` - 4 bytes

Subtract the value at `[%REG2+$OFFSET]` from `%REG1`

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

`%REG1 -= [%REG2+$OFFSET]`

# SUB `[%REG1]`, `%REG2`
`01101000` - 2 bytes

Subtract the value at `%REG2` from `[%REG1]`

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

`[%REG1] -= %REG2`

# INC `%REG`
`00011111:` - 2 bytes

Increment `%REG` by one
`%REG += 1`

Reset the flag register, then set the `CF` to 1 if the result has an extra carry, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

# INC `[%REG]`
`00000100` - 2 bytes

Increment `[%REG]` by one
`[%REG] += 1`

Reset the flag register, then set the `CF` to 1 if the result has an extra carry, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

# INC `[%REG+$OFFSET]`
`10000010` - 4 bytes

Increment `[%REG+$OFFSET]` by one
`[%REG+$OFFSET] += 1`

Reset the flag register, then set the `CF` to 1 if the result has an extra carry, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).


# DEC `%REG`
`00000101` - 2 bytes

Decrement `%REG` by one

`%REG -= 1`

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

# DEC `[%REG]`
`00000110` - 2 bytes

Decrement `[%REG]` by one
`[%REG] -= 1`

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

# DEC `[%REG+$OFFSET]`
`10000011` - 4 bytes

Decrement `[%REG+$OFFSET]` by one
`[%REG+$OFFSET] -= 1`

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

# MUL `%REG1`, `%REG2`
`01101001` - 2 bytes

Multiply `%REG1` by `%REG2` and store the result in `%REG1`
`%REG1 *= %REG2`

Reset the flag register, The CF and OF are set when the result cannot fit in the operands size.

# MUL `%REG1`, `[%REG2]`
`01101010` - 2 bytes

Multiply `%REG1` by `[%REG2]` and store the result in `%REG1`
`%REG1 *= [%REG2]`

Reset the flag register, The CF and OF are set when the result cannot fit in the operands size.

# MUL `[%REG1]`, `%REG2`
`01101011` - 2 bytes

Multiply `[%REG1]` by `%REG2` and store the result in `[%REG1]`
`[%REG1] *= %REG2`

Reset the flag register, The CF and OF are set when the result cannot fit in the operands size.

# MUL `%REG`, `$IMM`
`10000100` - 4 bytes

Multiply `%REG1` by `$IMM` and store the result in `%REG1`
`%REG1 *= $IMM`

Reset the flag register, The CF and OF are set when the result cannot fit in the operands size.

# MUL `%REG1`, `[%REG2+$OFFSET]`
`10100100` - 4 bytes

Multiply `%REG1` by `[%REG2+$OFFSET]` and store the result in `%REG1`
`%REG1 *= [%REG2+$OFFSET]`

Reset the flag register, The CF and OF are set when the result cannot fit in the operands size.

# DIV `%REG1`, `%REG2`
`01101100` - 2 bytes

Divide `%REG1` by `%REG2` and store the result in `%REG1`
`%REG1 /= %REG2`

Reset the flag register, and set the `ZF` to 0 if the result is truly 0 (and not rounded).

If zero division is encountered, set the result, the `CF` and `OF` to all-1's and the rest of the flags to 0.

# DIV `%REG1`, `[%REG2]`
`01101101` - 2 bytes

Divide `%REG1` by `[%REG2]` and store the result in `%REG1`
`%REG1 /= [%REG2]`

Reset the flag register, and set the `ZF` to 0 if the result is truly 0 (and not rounded).

If zero division is encountered, set the result, the `CF` and `OF` to all-1's and the rest of the flags to 0.

# DIV `[%REG1]`, `%REG2`
`01101110` - 2 bytes

Divide `[%REG1]` by `%REG2` and store the result in `[%REG1]`
`[%REG1] /= %REG2`

Reset the flag register, and set the `ZF` to 0 if the result is truly 0 (and not rounded).

If zero division is encountered, set the result, the `CF` and `OF` to all-1's and the rest of the flags to 0.

# DIV `%REG`, `$IMM`
`10000101` - 4 bytes

Divide `%REG1` by `$IMM` and store the result in `%REG1`
`%REG1 /= $IMM`

Reset the flag register, and set the `ZF` to 0 if the result is truly 0 (and not rounded).

If zero division is encountered, set the result, the `CF` and `OF` to all-1's and the rest of the flags to 0.

# DIV `%REG1`, `[%REG2+$OFFSET]`
`10100101` - 4 bytes

Divide `%REG1` by `[%REG2+$OFFSET]` and store the result in `%REG1`
`%REG1 /= [%REG2+$OFFSET]`

# AND `%REG1`, `%REG2`
`01101111` - 2 bytes

Binary and `%REG1` and `%REG2`, storing the result into `%REG1`

`%REG1 &= %REG2`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

# AND `%REG1`, `[%REG2]`
`01110000` - 2 bytes

Binary and `%REG1` and `[%REG2]`, storing the result into `%REG1`

`%REG1 &= [%REG2]`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

# OR `%REG1`, `%REG2`
`01110001` - 2 bytes

Binary or `%REG1` and `%REG2`, storing the result into `%REG1`

`%REG1 |= %REG2`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

# OR `%REG1`, `[%REG2]`
`01110010` - 2 bytes

Binary or `%REG1` and `[%REG2]`, storing the result into `%REG1`

`%REG1 |= [%REG2]`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

# XOR `%REG1`, `%REG2`
`01110011` - 2 bytes

Binary xor `%REG1` and `%REG2`, storing the result into `%REG1`

`%REG1 ^= %REG2`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

# XOR `%REG1`, `[%REG2]`
`01110100` - 2 bytes

Binary xor `%REG1` and `[%REG2]`, storing the result into `%REG1`

`%REG1 ^= [%REG2]`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

# NOT `%REG1`
`00000111`  - 2 bytes

Binary not `%REG1`

`%REG1 = ~%REG1`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

# NOT `[%REG]`
`00001000` - 2 bytes

Binary not `[%REG1]`

`[%REG1] = ~[%REG1]`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

 
# LSH `%REG`, `$IMM`
`10000110` - 2 bytes

Shift `%REG` left logically by `$IMM` bits

`%REG = %REG << $IMM`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

# LSH `[%REG]`, `$IMM`
`10000111` - 2 bytes

Shift `[%REG]` left logically by `$IMM` bits

`[%REG] = [%REG] << $IMM`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

# LSH `[%REG+$OFFSET]`, `$IMM`
`11000001` - 6 bytes

Shift `[%REG+$OFFSET]` left logically by `$IMM` bits

`[%REG+$OFFSET] = [%REG+$OFFSET] << $IMM`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0
 
# RSH `%REG`, `$IMM`
`10001000` - 2 bytes

Shift `%REG` right logically by `$IMM` bits

`%REG = %REG >> $IMM`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

# RSH `[%REG]`, `$IMM`
`10001001` - 2 bytes

Shift `[%REG]` right logically by `$IMM` bits

`[%REG] = [%REG] >> $IMM`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

# RSH `[%REG+$OFFSET]`, `$IMM`
`11000010` - 6 bytes

Shift `[%REG+$OFFSET]` right logically by `$IMM` bits

`[%REG+$OFFSET] = [%REG+$OFFSET] >> $IMM`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

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

# CALL `%REG+$OFFSET`
`10001001` - 4 bytes

Jumps to `%REG + $OFFSET`

`%PC = %REG + $OFFSET`

# RET
`00100001` - 1 byte

Pops the value off the memory stack and puts it into `%PC`
 
# CMP `%REG1`, `%REG2`
`01110101` - 2 bytes

DOES NOT MUTATE THE REGISTERS
Compare `%REG1` and `%REG2` by subtractiong them. Sets the flags appropriately

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).
 
# CMP `%REG`, `$IMM`
`10001011` - 4 bytes

DOES NOT MUTATE THE REGISTERS
Compare `%REG1` and `$IMM` by subtractiong them. Sets the flags appropriately

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).
 
# CMP `%REG1`, `[%REG2]`
`10001011` - 2 bytes

DOES NOT MUTATE THE REGISTERS
Compare `%REG1` and `[%REG2]` by subtractiong them. Sets the flags appropriately

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).
 
# CMP `%REG1`, `[%REG2+$OFFSET]`
`10100110` - 4 bytes

DOES NOT MUTATE THE REGISTERS
Compare `%REG1` and `[%REG2 + $OFFSET]` by subtractiong them. Sets the flags appropriately

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).
 
# TEST `%REG1`, `%REG2`
`01110111` - 2 bytes

DOES NOT MUTATE THE REGISTERS
Performs bitwise and on `%REG1` and `%REG2`. Sets the flags accordingly. 

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0
 
# TEST `%REG1`, `[%REG2]`
`01111000` - 2 bytes

DOES NOT MUTATE THE REGISTERS
Performs bitwise and on `%REG1` and `[%REG2]`. Sets the flags accordingly. 

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

# TEST `%REG1`, `[%REG2+$OFFSET]`

`10100111` - 4 bytes
DOES NOT MUTATE THE REGISTERS
Performs bitwise and on `%REG1` and `[%REG2+$OFFSET]`. Sets the flags accordingly. 

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

# JMP `$IMM`
`01000011` - 3 bytes

Sets `%PC` to `$IMM`, effectively jumping to it.

`%PC = $IMM`

 
# JMP `%REG`
`00001010` - 2 bytes
 
Sets `%PC` to `%REG`, effectively jumping to it.

`%PC = %REG`

# JMP `%REG+$OFFSET`
`10001100` - 4 bytes
 
Sets `%PC` to `%REG + $OFFSET`, effectively jumping to it.

`%PC = %REG + $OFFSET`

# JE `$IMM`
`01000100` - 3 bytes

If `ZF == 1`, Sets `%PC` to `$IMM`, effectively jumping to that address
 
# JNE `$IMM`
`01000101` - 3 bytes

If `ZF == 0`, Sets `%PC` to `$IMM`, effectively jumping to that address
 
# JG `$IMM`
`01000110` - 3 bytes

If `ZF == 0 && SF == OF`, Sets `%PC` to `$IMM`, effectively jumping to that address
 
# JGE `$IMM`
`01000111` - 3 bytes

If `Cf == 0`, Sets `%PC` to `$IMM`, effectively jumping to that address
 
# JL `$IMM`
`01001000` - 3 bytes 

If `SF != OF`, Sets `%PC` to `$IMM`, effectively jumping to that address

# JLE `$IMM`
`01001001` - 3 bytes

If `CF == 1 || ZF == 1`, Sets `%PC` to `$IMM`, effectively jumping to that address
 			
# IN `%REG`, `$PORT`
`10001101` - 4 bytes

Transfers data from the device at port `$PORT` to `%REG`

# IN `[%REG]`, `$PORT`
`10001110` - 4 bytes

Transfers data from the device at port `$PORT` to `[%REG]`

# IN `[%REG+$OFFSET]`, `$PORT`
`11000011` - 6 bytes

Transfers data from the device at port `$PORT` to `[%REG + $OFFSET]`
 			
# OUT `$PORT`, `$IMM`
`11001111` - 5 bytes

Puts the value `$IMM` on to the port `$PORT`


# OUT `$PORT`, `%REG`
`10001111` - 4 bytes

Puts the value from `%REG` on to the port `$PORT`

# OUT `$PORT`, `[%REG]`
`10010000` - 4 bytes

Puts the value from `[%REG]` on to the port `$PORT`

# OUT `$PORT`, `[%REG+$OFFSET]`
`11000100` - 6 bytes

Puts the value from `[%REG + $OFFSET]` on to the port `$PORT`

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

Reset the flag register, then set the `CF` to 1 if the result has an extra carry, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

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
