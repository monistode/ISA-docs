# RISC ACCUMULATOR

## Structure

There is three addressing modes: register, immediate, memory location
Register name starts with `%`

8-bit instructions, 49 instructions

Registers:
- `R00` - General-purpose 16-bit word (`R00H` - high byte, `R00L` - low byte)
- `R01` - General-purpose 16-bit word (`R01H` - high byte, `R01L` - low byte)
- `R02` - General-purpose 16-bit word (`R02H` - high byte, `R02L` - low byte)
- `R03` - General-purpose 16-bit word (`R03H` - high byte, `R03L` - low byte)
- `FR` (flag resiter: `CF`(carry), `ZF`(zero), `OF`(overflow), `SF`(sign))
- `SP` - stack pointer
- `PC` - program counter
- `LR` - link register (stores call return address)

Opcode structure:

- for three registers (dest = src1, src2): `| 6-bit opcode | 3-bit register | 3-bit register | 3-bit register | 1 empty bit |`
- for a register and immediate constant `| 6-bit opcode | 3-bit register | 7-bit constant |`
- for immediate constant `| 6-bit opcode | 10-bit constant |`

And load immediate constants instructions look like:

`| 5-bit opcode for high/low load | 3-bit register | 8-bit immediate constant low/high byte |`

## Instructions

### HALT 
Disables cpu until power is cycled

### LOAD `%REG1`, `[%REG2]`
Loads the memory cell `[%REG2]` is pointing to: `%REG1 = [%REG2]`

### STORE `[%REG1]`, `%REG2`
Stores `%REG2` into the memory cell `[%REG1]` is pointing to: `[%REG1] = %REG2`


# TODO: FINISH ALL THIS `@pmozil`, PLEASE
### MOV `$imm`
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

Pops the previous value from the memory stack into `%IR`

### PUSHI
`00001000` - 1 byte

Pushes the value of the `%IR` register on to the top of the memory stack

### POPI
`00001100` - 1 byte

Pops the previous value from the memory stack into `%FR`

### ADD `[mem]`
`00001101` - 2 bytes

Add items from the `acc` register and memory location `mem` (or `acc` register), pushing the result into `acc`

`%ACC += [mem]`

### SUB `[mem]`
`00001110` - 2 bytes

Subtract items from the `acc` register and memory location `mem` (or `acc` register), pushing the result into `acc`

`%ACC -= [mem]`

### MUL `[mem]`
`00010001` - 2 bytes

Multiplies value from the location `mem` and value from the register `%ACC`, saving the result in the register `%ACC`

`%ACC *= [mem]`

### DIV `[mem]`
`00010010` - 2 bytes

Divides value from the register `%ACC` by value at the location `mem`, saving the result in the register `%ACC`

`%ACC /= [mem]`

### INC
`00001111` - 1 byte

Increments `%ACC` by one

`%ACC += 1`

### DEC
`00010000` - 1 byte

Decrements `%ACC` by one

`%ACC -= 1`

### AND `[mem]`
`00010011` - 2 bytes

Computes binary and between word from the `%ACC` register and `mem` location, saving the result into `%ACC`

`%ACC  = %ACC & [mem]`

### OR `[mem]`
`00010100` - 2 bytes

Computes binary or between word from the `%ACC` register and `mem` location, saving the result into `%ACC`

`%ACC  = %ACC | [mem]`

### XOR `[mem]`
`00010101` - 2 bytes

Computes binary xor between word from the `%ACC` register and `mem` location, saving the result into `%ACC`

`%ACC  = %ACC ^ [mem]`

### NOT `[mem]`
`00010110` - 1 byte

Computes binary not between word for the `%ACC` register

`%ACC  = ~%ACC`

### LSH `$num`
`10000010` - 2 bytes

Shifts the value from register `acc` by `num` to the left, saving the result in the register `acc`

`%ACC = %ACC << $num`

### RSH `$num`
`10000011` - 2 bytes

Shifts the value from register `acc` by `num` to the left, saving the result in the register `acc`

`%ACC = %ACC >> $num`

###  CALL `label`
`10000100` - 2 bytes

Pushes the next instruction's location on to the memory stack, transfers control to the location at `label`

###  CALL `$num`
`10000100` - 2 bytes

Pushes the next instruction's location on to the memory stack, transfers control to the location at `num`

###  CALL
`00010111` - 1 byte

Pushes the next instruction's location on to the memory stack, transfers control to the location at `%ACC`

### RET
`00011000` - 1 byte

Transfers control to the popped instruction location

### CMP `[mem]`
`00011001` - 2 bytes

Compares value of register `%ACC` and value at location `mem` (or with constant `num`), by subtracting the second one from the first one, changing the flags accordingly

### CMP `$imm`
`10000101` - 2 bytes

Compares value of register `%ACC` and value `imm`, by subtracting the second one from the first one, changing the flags accordingly

### TEST `$num`
`10000110` - 2 bytes

Performs bitwise `and` operation between the `num` and `%ACC`, changing the flags accordingly

### TEST `[mem]`
`10000110` - 2 bytes

Performs bitwise `and` operation between the value at `mem` and `%ACC`, changing the flags accordingly

### JMP `$num`
`10000111` - 2 bytes

If `ZF == 1`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JMP
`00011011` - 1 byte

If `ZF == 1`, Sets `%PC` to `%ACC`, effectively jumping to that address

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
