# RISC REGISTER

## Structure

#### THE ISA IS BIG ENDIAN

There is three addressing modes: register, immediate, memory location
Register name starts with `%`

8-bit instructions, 49 instructions

Registers:
- `R00` - `16 bits` - General-purpose 16-bit word (`R00H` - high byte, `R00L` - low byte)
- `R01` - `16 bits` - General-purpose 16-bit word (`R01H` - high byte, `R01L` - low byte)
- `R02` - `16 bits` - General-purpose 16-bit word (`R02H` - high byte, `R02L` - low byte)
- `R03` - `16 bits` - General-purpose 16-bit word (`R03H` - high byte, `R03L` - low byte)
- `FR`  - `8 bits` -(flag resiter: `CF`(carry), `ZF`(zero), `OF`(overflow), `SF`(sign))
- `SP` -  `16 bits` - stack pointer
- `PC` -  `16 bits` - program counter

Opcode structure:

- for three registers (dest = src1, src2): `| 6-bit opcode | 3-bit register | 3-bit register | 3-bit register | 1 empty bit |`
- for a register and immediate constant `| 6-bit opcode | 3-bit register | 7-bit constant |`
- for immediate constant `| 6-bit opcode | 10-bit constant |`

And load immediate constants instructions look like:

`| 5-bit opcode for high/low load | 3-bit register | 8-bit immediate constant low/high byte |`

## Instructions

### HALT 
`000000` - 1 byte

Disables cpu until power is cycled

### LOAD `%REG1`, `[%REG2]`
`000001` - 1byte

Loads the memory cell `[%REG2]` is pointing to: `%REG1 = [%REG2]`

### STORE `[%REG1]`, `%REG2`
`000010` - 1 byte

Stores `%REG2` into the memory cell `[%REG1]` is pointing to: `[%REG1] = %REG2`

### MOV `%REG, $imm`
`000110` - 2 bytes, used for loading `R00L, R01L, R02L, R03L`

`000111` - 2 bytes, used for loading `R00L, R01L, R02L, R03L`

`001000` - 2 bytes, used for loading `R00H, R01H, R02H, R03H`

`001001` - 2 bytes, used for loading `R00H, R01H, R02H, R03H`

Puts the value `$imm` in `%REG`: `%REG = $imm`

### MOV `%REG1, %REG2`
`000101` - 2 bytes

Puts the value `%REG2` in `%REG1`: `%REG1 = %REG2`

### PUSH `%REG`
`101000` - 1 byte

Pushes the value of the `%REG` register on to the top of the memory stack

### POP `%REG`
`101001` - 1 byte

Pops the previous value from the memory stack into `%REG`

### ADD `%REG1, %REG2, %REG3`
`000011` - 2 bytes

Add items from the `%REG2` register and register `%REG3`, and puts the result into `%REG1`

Reset the flag register, then set the `CF` to 1 if the result has an extra carry, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

Note: the implementation isn't at all working as expected, as the `change_flag_result` function takes in the truncated result.

Note: further operations' TODOs say "as if it's addition"; `OF` isn't affected how you'd expect it to with addition for them - we compute (most_significant_bit_a $=$ most_significant_bit_b and most_significant_bit_result $\ne$ most_significant_bit_a). [See implementation](https://github.com/ucu-computer-science/Hardware-Simulator-and-Assembler/blob/master/modules/functions.py#L430)

### ADDC `%REG1, %REG2, %REG3`
`100100` - 2 bytes

Add items **WITH CARRY** from the `%REG2` register and register `%REG3`, and puts the result into `%REG1`

`%REG1 = %REG2 + %REG3 + CF`

Reset the flag register, then set the `CF` to 1 if the result has an extra carry, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

Note: the implementation isn't at all working as expected, as the `change_flag_result` function takes in the truncated result.

Note: further operations' TODOs say "as if it's addition"; `OF` isn't affected how you'd expect it to with addition for them - we compute (most_significant_bit_a $=$ most_significant_bit_b and most_significant_bit_result $\ne$ most_significant_bit_a). [See implementation](https://github.com/ucu-computer-science/Hardware-Simulator-and-Assembler/blob/master/modules/functions.py#L430)

### SUB `%REG1, %REG2, %REG3`
`000100` -  2 bytes

Subtract from the `%REG2` register  the value in register `%REG3`, and puts the result into `%REG1`

`%REG1 = %REG2 - %REG3`

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

### MUL `%REG1, %REG2, %REG3`
`001010` - 2 bytes

Multiply items from the `%REG2` register and register `%REG3`, and puts the result into `%REG1`

`%REG1 = %REG2 * %REG3`

Reset the flag register, The CF and OF are set when the result cannot fit in the operands size.

### DIV `%REG1, %REG2, %REG3`
`001011` - 2 bytes

Divide the `%REG3` register  by register `%REG2`, and puts the result into `%REG1`

`%REG1 = %REG2 / %REG3`

Reset the flag register, and set the `ZF` to 0 if the result is truly 0 (and not rounded).

If zero division is encountered, set the result, the `CF` and `OF` to all-1's and the rest of the flags to 0.

### AND `%REG1, %REG2, %REG3`
`001100` - 2 bytes

Computes binary and between word from the `%REG2` register and `%REG3` location, saving the result into `%REG1`

`%REG1  = %REG2 & %REG3`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### OR `%REG1, %REG2, %REG3`
`001101` - 2 bytes

Computes binary or between word from the `%REG2` register and `%REG3` location, saving the result into `%REG1`

`%REG1  = %REG2 | %REG3`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### XOR `%REG1, %REG2, %REG3`
`001110` - 2 bytes

Computes binary xor between word from the `%REG2` register and `%REG3` location, saving the result into `%REG1`

`%REG1  = %REG2 ^ %REG3`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### NOT `%REG1, %REG2`
`001111` - 2 bytes

Computes binary not of the `%REG2` register, saving the result into `%REG1`

`%REG1  = ~%REG2`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### LSH `%REG1, %REG2, %REG3`
`010000` - 2 bytes

Shifts `%REG2` left by `%REG3` bits, and saves it to `%REG1`

`%REG1  = %REG2  << %REG3`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### RSH `%REG1, %REG2, %REG3`
`010001` - 2 bytes

Shifts `%REG2` right by `%REG3` bits, and saves it to `%REG1`

`%REG1  = %REG2  >> %REG3`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

###  CALL `label`
`010010` - 2 bytes

Pushes the next instruction's location on to the memory stack, transfers control to the location at `label`

###  CALL `[$num]`
`010010` - 2 bytes

Pushes the next instruction's location on to the memory stack, transfers control to the location at `[$num]`

###  CALL `[%REG]`
`010011` - 2 bytes

Pushes the next instruction's location on to the memory stack, transfers control to the location at `%REg`

### RET
`010100` - 2 bytes

Transfers control to the popped instruction location

### CMP `%REG1, %REG2`
`010101` - 2 bytes

Compares value of register `%REG1` and value `%REG2`, by subtracting the second one from the first one, changing the flags accordingly

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

### CMP `%REG1, $num`
`010110` - 2 bytes

Compares value of register `%REG1` and value `$num`, by subtracting the second one from the first one, changing the flags accordingly

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

### TEST `%REG1, %REG2`
`010111` - 2 bytes

Performs bitwise `and` operation between the `%REG1` and `%REG2`, changing the flags accordingly

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### TEST `%REG1, $num`
`011000` - 2 bytes

Performs bitwise `and` operation between the `%REG1` and the value `$unm`, changing the flags accordingly

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### JMP `$num`
`011001` - 2 bytes

If `ZF == 1`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JMP `%REG`
`011010` - 2 bytes

If `ZF == 1`, Sets `%PC` to `%REG`, effectively jumping to that address

`%PC = %REG`

### JE `$num`
`011010` -  2 bytes

If `ZF == 1`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JNE `$num`
`011011` - 2 bytes

If `ZF == 0`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JG `$num`
`011100` - 2 bytes

If `ZF == 0 && SF == OF`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JGE `$num`
`011101` - 2 btyes

If `Cf == 0`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JL `$num`
`011110` - 2 bytes

If `SF != OF`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JLE `$num`
`011111` - 2 bytes

If `CF == 1 || ZF == 1`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### IN %REG, `$num`
`100000` - 2 bytes
Transfers data from the device at port `num` to `%REG`

### OUT `$num1, $num2`
`100001` - 2 bytes

Transfers data from `num2` to the device at port `num1`

### OUT `$num1, %REG`
`100010` -2 bytes

Transfers data from `%REG` to the device at port `num1`

### NOP
`100011` - 1 byte

Do nothing
