# RISC ACCUMULATOR

## Structure

#### THE ISA IS BIG ENDIAN

There is three addressing modes: register, immediate, memory location
Register name starts with `%`

8-bit instructions, 49 instructions

Registers:
- `FR` - `8 bits` (flag resiter: `CF`(carry), `ZF`(zero), `OF`(overflow), `SF`(sign))
- `SP` - `16 bits` - stack pointer
- `PC` - `16 bits` - program counter
- `IR` - `16 bits` - index register (used for indexing in arrays)
- `ACC` - `8 bits` - the accumulator

Opcode structure:
`| 1 bit immediate sign | 7-bit opcode |`

And immediate constants contain two 8-bit bytes:
`| 8-bit immediate high byte | 8-bit immediate low byte |`

## Instructions

### HALT 
`00000000` - 1 byte

Disables cpu until power is cycled

### LOAD
`00000001` - 1 byte

Loads the memory cell `IR` is pointing to: `%ACC = [%IR]`

### LOAD `%FR`
`00000010` - 2 bytes

Loads the flag register: `%ACC = %FR`

### LOAD `%IR`
`00000011` - 2 bytes

Loads the index register: `%ACC = %IR`

### STORE
`00000100` - 1 byte

Stores `%ACC` into the memory cell `IR` is pointing to: `[%IR] = %ACC`

### STORE `%FR`
`00000101` - 2 bytes

Stores `%ACC` into the flag register: `%FR = %ACC`

### STORE `%IR`
`00000110` - 2 bytes

Stores `ACC` into  the index register: `%IR = %ACC`

### MOV `$imm`
`10000001` - 2 bytes

Puts the value `$imm` in the `acc`: `%ACC = $imm`

### PUSH
`00000111` - 1 byte

Pushes the value of the `%ACC` register on to the top of the memory stack

### POP
`00001010` - 1 byte

Pops the previous value from the memory stack into `%ACC`

### PUSHF
`00001000` - 1 byte

Pushes the value of the `%FR` register on to the top of the memory stack

### POPF
`00001011` - 1 byte

Pops the previous value from the memory stack into `%FR`

### PUSHI
`00001000` - 1 byte

Pushes the value of the `%IR` register on to the top of the memory stack

### POPI
`00001100` - 1 byte

Pops the previous value from the memory stack into `%IR`

### ADD `[mem]`
`00001101` - 2 bytes

Add items from the `acc` register and memory location `mem` (or `acc` register), pushing the result into `acc`

Reset the flag register, then set the `CF` to 1 if the result has an extra carry, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

Note: the implementation isn't at all working as expected, as the `change_flag_result` function takes in the truncated result.

Note: further operations' TODOs say "as if it's addition"; `OF` isn't affected how you'd expect it to with addition for them - we compute (most_significant_bit_a $=$ most_significant_bit_b and most_significant_bit_result $\ne$ most_significant_bit_a). [See implementation](https://github.com/ucu-computer-science/Hardware-Simulator-and-Assembler/blob/master/modules/functions.py#L430)

### SUB `[mem]`
`00001110` - 2 bytes

Subtract items from the `acc` register and memory location `mem` (or `acc` register), pushing the result into `acc`

`%ACC -= [mem]`

Reset the flag register, then set the `CF` to 1 if the result has an extra carry, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

### MUL `[mem]`
`00010001` - 2 bytes

Multiplies value from the location `mem` and value from the register `%ACC`, saving the result in the register `%ACC`

`%ACC *= [mem]`

Reset the flag register, The CF and OF are set when the result cannot fit in the operands size.

### DIV `[mem]`
`00010010` - 2 bytes

Divides value from the register `%ACC` by value at the location `mem`, saving the result in the register `%ACC`

`%ACC /= [mem]`

Reset the flag register, and set the `ZF` to 0 if the result is truly 0 (and not rounded).

If zero division is encountered, set the result, the `CF` and `OF` to all-1's and the rest of the flags to 0.

### INC
`00001111` - 1 byte

Increments `%ACC` by one

`%ACC += 1`

Reset the flag register, then set the `CF` to 1 if the result has an extra carry, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

### DEC
`00010000` - 1 byte

Decrements `%ACC` by one

`%ACC -= 1`

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

### AND `[mem]`
`00010011` - 2 bytes

Computes binary and between word from the `%ACC` register and `mem` location, saving the result into `%ACC`

`%ACC  = %ACC & [mem]`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### OR `[mem]`
`00010100` - 2 bytes

Computes binary or between word from the `%ACC` register and `mem` location, saving the result into `%ACC`

`%ACC  = %ACC | [mem]`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### XOR `[mem]`
`00010101` - 2 bytes

Computes binary xor between word from the `%ACC` register and `mem` location, saving the result into `%ACC`

`%ACC  = %ACC ^ [mem]`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### NOT `[mem]`
`00010110` - 1 byte

Computes binary not between word for the `%ACC` register

`%ACC  = ~%ACC`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### LSH `$num`
`10000010` - 2 bytes

Shifts the value from register `acc` by `num` to the left, saving the result in the register `acc`

`%ACC = %ACC << $num`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### RSH `$num`
`10000011` - 2 bytes

Shifts the value from register `acc` by `num` to the left, saving the result in the register `acc`

`%ACC = %ACC >> $num`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

###  CALL `label`
`10000100` - 2 bytes

Pushes the next instruction's location on to the memory stack, transfers control to the location at `label`

###  CALL `$num`
`10000100` - 2 bytes

Pushes the next instruction's location on to the memory stack, transfers control to the location at `[$num]`

###  CALL
`00010111` - 1 byte

Pushes the next instruction's location on to the memory stack, transfers control to the location at `%ACC`

### RET
`00011000` - 1 byte

Transfers control to the popped instruction location

### CMP `[mem]`
`00011001` - 2 bytes

Compares value of register `%ACC` and value at location `mem` (or with constant `num`), by subtracting the second one from the first one, changing the flags accordingly

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

### CMP `$imm`
`10000101` - 2 bytes

Compares value of register `%ACC` and value `imm`, by subtracting the second one from the first one, changing the flags accordingly

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

### TEST `$num`
`10000110` - 2 bytes

Performs bitwise `and` operation between the `num` and `%ACC`, changing the flags accordingly

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### TEST `[mem]`
`10000110` - 2 bytes

Performs bitwise `and` operation between the value at `mem` and `%ACC`, changing the flags accordingly

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### JMP `$num`
`10000111` - 2 bytes

Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JMP
`00011011` - 1 byte

Sets `%PC` to `%ACC`, effectively jumping to that address

`%PC = %ACC`

### JE `$num`
`10001000` - 2 bytes

If `ZF == 1`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JE
`00011100` - 1 byte

If `ZF == 1`, Sets `%PC` to `%ACC`, effectively jumping to that address

`%PC = %ACC`

### JNE `$num`
`10001001` - 2 bytes

If `ZF == 0`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JNE
`00011101` - 1 byte

If `ZF == 0`, Sets `%PC` to `%ACC`, effectively jumping to that address

`%PC = %ACC`

### JG `$num`
`10001010` - 2 bytes

If `ZF == 0 && SF == OF`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JG
`00011110` - 1 byte

If `ZF == 0 && SF == OF`, Sets `%PC` to `%ACC`, effectively jumping to that address

`%PC = %ACC`

### JGE `$num`
`10001011` - 2 bytes

If `Cf == 0`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JGE
`00011111` - 1 byte

If `CF == 0`, Sets `%PC` to `%ACC`, effectively jumping to that address

`%PC = %ACC`

### JL `$num`
`10001100` - 2 bytes

If `SF != OF`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JL
`00100000` - 1 byte

If `SF != OF`, Sets `%PC` to `%ACC`, effectively jumping to that address

`%PC = %ACC`

### JLE `$num`
`10001101` - 2 bytes

If `CF == 1 || ZF == 1`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JLE
`00100001` - 1 byte

If `CF == 1 || ZF == 1`, Sets `%PC` to `%ACC`, effectively jumping to that address

`%PC = %ACC`

### IN `$num`
`10001110` - 2 bytes

Transfers data from the device at port `num` to `%ACC`

### OUT `$num1, $num2`
`10001111` - 3 bytes

Transfers data from `num2` to the device at port `num1`

### OUT `$num1`
`00100010` - 2 bytes

Transfers data from `%ACC` to the device at port `num1`
