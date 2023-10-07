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

### MOV `%REG, $imm`
Puts the value `$imm` in `%REG`: `%REG = $imm`

### MOV `%REG1, %REG2`
Puts the value `%REG2` in `%REG1`: `%REG1 = %REG2`

### PUSH `%REG`
Pushes the value of the `%REG` register on to the top of the memory stack

### POP `%REg`
Pops the previous value from the memory stack into `%REG`

### ADD `%REG1, %REG2, %REG3`
Add items from the `%REG2` register and register `%REG3`, and puts the result into `%REG1`

`%REG1 = %REG2 + %REG3`

### SUB `%REG1, %REG2, %REG3`
Subtract from the `%REG2` register  the value in register `%REG3`, and puts the result into `%REG1`

`%REG1 = %REG2 - %REG3`

### MUL `%REG1, %REG2, %REG3`
Multiply items from the `%REG2` register and register `%REG3`, and puts the result into `%REG1`

`%REG1 = %REG2 * %REG3`

### DIV `%REG1, %REG2, %REG3`
Divide the `%REG3` register  by register `%REG2`, and puts the result into `%REG1`

`%REG1 = %REG2 / %REG3`

### AND `%REG1, %REG2, %REG3`
Computes binary and between word from the `%REG2` register and `%REG3` location, saving the result into `%REG1`

`%REG1  = %REG2 & %REG3`

### OR `%REG1, %REG2, %REG3`
Computes binary or between word from the `%REG2` register and `%REG3` location, saving the result into `%REG1`

`%REG1  = %REG2 | %REG3`

### XOR `%REG1, %REG2, %REG3`
Computes binary xor between word from the `%REG2` register and `%REG3` location, saving the result into `%REG1`

`%REG1  = %REG2 ^ %REG3`

### NOT `%REG1, %REG2`
Computes binary not of the `%REG2` register, saving the result into `%REG1`

`%REG1  = ~%REG2`

### LSH `%REG1, %REG2, %REG3`
Shifts `%REG2` left by `%REG3` bits, and saves it to `%REG1`

`%REG1  = %REG2  << %REG3`

### RSH `%REG1, %REG2, %REG3`
Shifts `%REG2` right by `%REG3` bits, and saves it to `%REG1`

`%REG1  = %REG2  >> %REG3`

###  CALL `label`

Pushes the next instruction's location on to the memory stack, transfers control to the location at `label`

###  CALL `[$num]`

Pushes the next instruction's location on to the memory stack, transfers control to the location at `[$num]`

###  CALL `[%REG]`
Pushes the next instruction's location on to the memory stack, transfers control to the location at `%REg`

### RET
Transfers control to the popped instruction location

### CMP `%REG1, %REG2`
Compares value of register `%REG1` and value `%REG2`, by subtracting the second one from the first one, changing the flags accordingly

### CMP `%REG1, $num`
Compares value of register `%REG1` and value `$num`, by subtracting the second one from the first one, changing the flags accordingly

### TEST `%REG1, %REG2`
Performs bitwise `and` operation between the `%REG1` and `%REG2`, changing the flags accordingly

### TEST `%REG1, $num`
Performs bitwise `and` operation between the `%REG1` and the value `$unm`, changing the flags accordingly

### JMP `$num`
If `ZF == 1`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JMP `%REG`
If `ZF == 1`, Sets `%PC` to `%REG`, effectively jumping to that address

`%PC = %REG`

### JE `$num`
If `ZF == 1`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JE `%REG`
If `ZF == 1`, Sets `%PC` to `%REG`, effectively jumping to that address

`%PC = %REG`

### JNE `$num`
If `ZF == 0`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JNE `%REG`
If `ZF == 0`, Sets `%PC` to `%REG`, effectively jumping to that address

`%PC = %REG`

### JG `$num`
If `ZF == 0 && SF == OF`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JG `%REG`
If `ZF == 0 && SF == OF`, Sets `%PC` to `%REG`, effectively jumping to that address

`%PC = %REG`

### JGE `$num`
If `Cf == 0`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JGE `%REG`
If `CF == 0`, Sets `%PC` to `%REG`, effectively jumping to that address

`%PC = %REG`

### JL `$num`
If `SF != OF`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JL `%REG`
If `SF != OF`, Sets `%PC` to `%REG`, effectively jumping to that address

`%PC = %REG`

### JLE `$num`
If `CF == 1 || ZF == 1`, Sets `%PC` to `$num`, effectively jumping to that address

`%PC = $num`

### JLE `%REG`
If `CF == 1 || ZF == 1`, Sets `%PC` to `%REG`, effectively jumping to that address

`%PC = %REG`

### IN %REG, `$num`
Transfers data from the device at port `num` to `%REG`

### OUT `$num1, $num2`
Transfers data from `num2` to the device at port `num1`

### OUT `$num1, %REG`
Transfers data from `%REG` to the device at port `num1`
