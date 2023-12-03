# RISC ACCUMULATOR

## Structure

#### THE ISA IS BIG ENDIAN

There is three addressing modes: register, immediate, memory location
Register name starts with `%`

8-bit instructions, 81 instructions

Registers:
- `FR` - `8 bits` (flag resiter: `CF`(carry), `ZF`(zero), `OF`(overflow), `SF`(sign))
- `SP` - `16 bits` - stack pointer
- `PC` - `16 bits` - program counter
- `IR` - `16 bits` - index register (used for indexing in arrays)
- `ACC` - `16 bits` - the accumulator

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

Loads the memory cell `ACC` is pointing to: `%ACC = [%ACC]`

### LOAD $IMM
`11110000` - 3 bytes

Loads the memory cell `ACC` is pointing to: `%ACC = $IMM`

### LOADF
`00000010` - 1 byte

Loads the flag register: `%ACC = %FR`

### LOAD1
`01110000` - 1 byte

Loads the value at first index register: `%ACC = ]%IR1]`

### LOAD2
`01110001` - 1 byte

Loads the value at second index register: `%ACC = [%IR2]`

### LOADI1
`00100011` - 1 byte

Loads the index register: `%ACC = %IR1`

### LOADI2
`00100100` - 1 byte

Loads the index register: `%ACC = %IR2`

### STORE1
`00000100` - 1 byte

Stores `%ACC` into the memory cell `IR1` is pointing to: `[%IR1] = %ACC`

### STORE1 $IMM
`11110001` - 3 bytes

Stores `%IMM` into the memory cell `IR1` is pointing to: `[%IR1] = $IMM`

### STORE2
`00100111` - 1 byte

Stores `%ACC` into the memory cell `IR2` is pointing to: `[%IR2] = %ACC`

### STORE2 $IMM
`11110010` - 3 bytes

Stores `%IMM` into the memory cell `IR2` is pointing to: `[%IR2] = $IMM`

### STOREF
`00000101` - 1 byte

Stores `%ACC` into the flag register: `%FR = %ACC`

### STOREI1
`00100101` - 1 byte

Stores `ACC` into  the index register: `%IR1 = %ACC`

### STOREI2
`00100110` - 1 byte

Stores `ACC` into  the index register: `%IR2 = %ACC`

### TI12
`01001100` - 1 byte

Copies `%IR1` to `%IR2`

### TI21
`01001101` - 1 byte

Copies `%IR2` to `%IR1`

### MOV `$imm`
`10000001` - 3 bytes

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

### PUSHI1
`01000100` - 1 byte

Pushes the value of the `%IR1` register on to the top of the memory stack

### POPI1
`00001100` - 1 byte

Pops the previous value from the memory stack into `%IR1`

### PUSHI2
`00101000` - 1 byte

Pushes the value of the `%IR2` register on to the top of the memory stack

### POPI2
`00101001` - 1 byte

Pops the previous value from the memory stack into `%IR2`

### ADD `[mem]`
`00001101` - 3 bytes

Add items from the `acc` register and memory location `mem` (or `acc` register), pushing the result into `acc`

Reset the flag register, then set the `CF` to 1 if the result has an extra carry, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

Note: the implementation isn't at all working as expected, as the `change_flag_result` function takes in the truncated result.

Note: further operations' TODOs say "as if it's addition"; `OF` isn't affected how you'd expect it to with addition for them - we compute (most_significant_bit_a $=$ most_significant_bit_b and most_significant_bit_result $\ne$ most_significant_bit_a). [See implementation](https://github.com/ucu-computer-science/Hardware-Simulator-and-Assembler/blob/master/modules/functions.py#L430)

### ADDI1
`00101010` - 1 byte

Add items from the `acc` register and memory location `[%IR1]` (or `acc` register), pushing the result into `acc`

### ADDI2
`00101011` - 1 byte

Add items from the `acc` register and memory location `[%IR2]` (or `acc` register), pushing the result into `acc`

### SUB `[mem]`
`00001110` - 3 bytes

Subtract items from the `acc` register and memory location `mem` (or `acc` register), pushing the result into `acc`

`%ACC -= [mem]`

Reset the flag register, then set the `CF` to 1 if the result has an extra carry, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

### SUBI1
`00101100` - 1 byte

Subtract items from the `acc` register and memory location `[%IR1]` (or `acc` register), pushing the result into `acc`

### SUBI2
`00101101` - 1 byte

Subtract items from the `acc` register and memory location `[%IR2]` (or `acc` register), pushing the result into `acc`

### MUL `[mem]`
`00010001` - 3 bytes

Multiplies value from the location `mem` and value from the register `%ACC`, saving the result in the register `%ACC`

`%ACC *= [mem]`

Reset the flag register, The CF and OF are set when the result cannot fit in the operands size.

### MULI1
`00101110` - 1 byte

Multiplies value from the location `[%IR1]` and value from the register `%ACC`, saving the result in the register `%ACC`

### MULI2
`00101111` - 1 byte

Multiplies value from the location `[%IR2]` and value from the register `%ACC`, saving the result in the register `%ACC`

### DIV `[mem]`
`00010010` - 3 bytes

Divides value from the register `%ACC` by value at the location `mem`, saving the result in the register `%ACC`

`%ACC /= [mem]`

Reset the flag register, and set the `ZF` to 0 if the result is truly 0 (and not rounded).

If zero division is encountered, set the result, the `CF` and `OF` to all-1's and the rest of the flags to 0.

### DIVI1
`00110000` - 1 byte

Divides value from the register `%ACC` by value at the location `[%IR1]`, saving the result in the register `%ACC`

### DIVI2
`00110001` - 1 byte

Divides value from the register `%ACC` by value at the location `[%IR2]`, saving the result in the register `%ACC`

### INC
`00001111` - 1 byte

Increments `%ACC` by one

`%ACC += 1`

Reset the flag register, then set the `CF` to 1 if the result has an extra carry, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

### IN1 
`00110010` - 1 byte

Increments `%IR1` by one

### IN12
`00110011` - 1 byte

Increments `%IR2` by one

### DEC
`00010000` - 1 byte

Decrements `%ACC` by one

`%ACC -= 1`

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

### DE1 
`00110100` - 1 byte

Decrements `%IR1` by one

### DE2
`00110101` - 1 byte

Decrements `%IR2` by one

### AND `[mem]`
`00010011` - 3 bytes

Computes binary and between word from the `%ACC` register and `mem` location, saving the result into `%ACC`

`%ACC  = %ACC & [mem]`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### ANDI1 
`00110110` - 1 byte

Computes binary and between word from the `%ACC` register and `[%IR2]` location, saving the result into `%ACC`

### ANDI2
`00110111` - 1 byte

Computes binary and between word from the `%ACC` register and `[%IR2]` location, saving the result into `%ACC`

### OR `[mem]`
`00010100` - 3 bytes

Computes binary or between word from the `%ACC` register and `mem` location, saving the result into `%ACC`

`%ACC  = %ACC | [mem]`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### ORI1 
`00111000` - 1 byte

Computes binary or between word from the `%ACC` register and `[%IR2]` location, saving the result into `%ACC`

### ORI2
`00111001` - 1 byte

Computes binary or between word from the `%ACC` register and `[%IR2]` location, saving the result into `%ACC`

### XOR `[mem]`
`00010101` - 3 bytes

Computes binary xor between word from the `%ACC` register and `mem` location, saving the result into `%ACC`

`%ACC  = %ACC ^ [mem]`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### XORI1 
`00111010` - 1 byte

Computes binary xor between word from the `%ACC` register and `[%IR2]` location, saving the result into `%ACC`

### XORI2
`00111011` - 1 byte

Computes binary xor between word from the `%ACC` register and `[%IR2]` location, saving the result into `%ACC`

### NOT `[mem]`
`00010110` - 1 byte

Computes binary not between word for the `%ACC` register

`%ACC  = ~[mem]`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### NOTI1 
`00111100` - 1 byte

Computes binary not between word from the `%ACC` register and `[%IR2]` location, saving the result into `%ACC`

### NOTI2
`01001001` - 1 byte

Computes binary not between word from the `%ACC` register and `[%IR2]` location, saving the result into `%ACC`

### LSH `$num`
`10000010` - 3 bytes

Shifts the value from register `acc` by `num` to the left, saving the result in the register `acc`

`%ACC = %ACC << $num`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### RSH `$num`
`10000011` - 3 bytes

Shifts the value from register `acc` by `num` to the left, saving the result in the register `acc`

`%ACC = %ACC >> $num`

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

###  CALL `label`
`10000100` - 3 bytes

Pushes the next instruction's location on to the memory stack, transfers control to the location at `label`

###  CALL `$num`
`10000100` - 3 bytes

Pushes the next instruction's location on to the memory stack, transfers control to the location at `[$num]`

###  CALL
`00010111` - 1 byte

Pushes the next instruction's location on to the memory stack, transfers control to the location at `%ACC`

### RET
`00011000` - 1 byte

Transfers control to the popped instruction location

### CMP `[mem]`
`00011001` - 3 bytes

Compares value of register `%ACC` and value at location `mem` (or with constant `num`), by subtracting the second one from the first one, changing the flags accordingly

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

### CMP `$imm`
`10000101` - 3 bytes

Compares value of register `%ACC` and value `imm`, by subtracting the second one from the first one, changing the flags accordingly

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

### CMPI1 
`00111101` - 1 byte

Compares value of register `%ACC` and value at location `[%IR1]` (or with constant `num`), by subtracting the second one from the first one, changing the flags accordingly

### CMPI2
`00111110` - 1 byte

Compares value of register `%ACC` and value at location `[%IR2]` (or with constant `num`), by subtracting the second one from the first one, changing the flags accordingly

### TEST `$num`
`10000110` - 3 bytes

Performs bitwise `and` operation between the `num` and `%ACC`, changing the flags accordingly

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### TEST `[mem]`
`10011111` - 3 bytes

Performs bitwise `and` operation between the value at `mem` and `%ACC`, changing the flags accordingly

Resets `FR`. Sets all flags as if addition, except `OF` - it's 0

### TESTI1 
`00111111` - 1 byte

Performs bitwise `and` operation between the value at `[%IR1]` and `%ACC`, changing the flags accordingly

### TESTI2
`01000000` - 1 byte

Performs bitwise `and` operation between the value at `[%IR2]` and `%ACC`, changing the flags accordingly

### JMP `$num`
`10000111` - 3 bytes

Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JMP
`00011011` - 1 byte

Sets `%PC` to `%ACC`, effectively jumping to that address

`%PC = %ACC`

### JE `$num`
`10001000` - 3 bytes

If `ZF == 1`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JE
`00011100` - 1 byte

If `ZF == 1`, Sets `%PC` to `%ACC`, effectively jumping to that address

`%PC = %ACC`

### JNE `$num`
`10001001` - 3 bytes

If `ZF == 0`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JNE
`00011101` - 1 byte

If `ZF == 0`, Sets `%PC` to `%ACC`, effectively jumping to that address

`%PC = %ACC`

### JG `$num`
`10001010` - 3 bytes

If `ZF == 0 && SF == OF`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JG
`00011110` - 1 byte

If `ZF == 0 && SF == OF`, Sets `%PC` to `%ACC`, effectively jumping to that address

`%PC = %ACC`

### JGE `$num`
`10001011` - 3 bytes

If `Cf == 0`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JGE
`00011111` - 1 byte

If `CF == 0`, Sets `%PC` to `%ACC`, effectively jumping to that address

`%PC = %ACC`

### JL `$num`
`10001100` - 3 bytes

If `SF != OF`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JL
`00100000` - 1 byte

If `SF != OF`, Sets `%PC` to `%ACC`, effectively jumping to that address

`%PC = %ACC`

### JLE `$num`
`10001101` - 3 bytes

If `CF == 1 || ZF == 1`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JLE
`00100001` - 1 byte

If `CF == 1 || ZF == 1`, Sets `%PC` to `%ACC`, effectively jumping to that address

`%PC = %ACC`

### IN `$num`
`10001110` - 3 bytes

Transfers data from the device at port `num` to `%ACC`

### OUT `$num1`
`00100010` - 3 bytes

Transfers data from `%ACC` to the device at port `num1`
