# STACK


## Structure

Harvard architecture; a 16-bit address space of 6-bit text bytes and a 16-bit address space of 8-bit data bytes


Three addressing modes: register, immediate, memory location

Register names start with `%`


6-bit instructions, 37 ones (1.5 nibbles)

0-31 - there are no immediate constants afterward

32-63 - there are immediate constants afterward (each somehow 16 bits)


TODO: figure out how to encode immediate constants


Two stacks:
- register stack - 16 bits per item, stores the data for operations. Starts at the 256th byte, grows upward. Stack underflow not handled
- memory stack - 16 bits per item, stores the function return addresses. Starts at the 1024th byre, grows downward. Stack underflow nor collisions with the register stack not handled


IO:
- multiple serial ports
- 16-bit address space for port numbers
- simple in and out operations


Registers:
- `PC` - `16 bits` - program counter, points to a 6-bit byte in the `text` section. Is always *the next* instruction to be executed, similar to x86
- `FR` - `4-16 bits` - flag register with the least significant bits representing `CF` `ZF` `OF` `SF`; (up to 16 bits because it has to fit on the stack)
  - TODO: figure out when it's cleared
- `TOS` - `15 or 16 bits` - initial value 256 (assuming 16 bits) - points to two bytes in the data section - the top of the *register stack*; grows upward (a push increments)
- `SP` - `15 or 16 bits` - initial value 1024 (assuming 16 bits) - points to two bytes in the data section - the top of the *memory stack*; grows downward (a push subtracts)
  - TODO: figure out whether the stack pointers are byte-level
  - TODO: figure out whether collisions are prevented
  - TODO: figure out how to decrement `tos` below 0 / increment above $2^n$


Opcode structure:
- `| 0b0 (immediate not there) | 5-bit opcode |`
- `| 0b1 (immediate present) | 5-bit opcode | 0b00 (padding) | 16-bit immediate |`


Arithmetic operations (big-endian):
- signed - look at `OF` to determine whether there was overflow; and at `SF` to determine the sign of the result (regardless of overflow)
- unsigned - look at `CF` to determine whether there was overflow

Only division is a strictly signed operation.


## Instructions

### HALT

`halt` - `000000`

Stop the execution


### LOAD

`load` - `000001`

`load %FR` (old assembler's `loadf`) - `000010`

`load [mem]` (old assembler's `load $mem`) - `100000`

Push a value onto the register stack. Value source:
- no arguments - pop an address from the register stack and use it as the address
- `%FR` - the flag register itself
- `[mem]` - from the memory location at `[mem]`

Assumption: the data is always present; the address space is filled up entirely


### STORE

`store` - `000100`

Pop the value from the register stack, then store it at the value the top register stack item points to (this whole operation does two consecutive pops from the stack)

`store $imm` - `100001`

`store %FR` (old assembler's `storef`) - `000101`

Store the immediate value or the word in the flag register at `[tos]` (popping the address from the stack)

Assumption: the data is always present; the address space is filled up entirely


### SWAP

`swap` - `000110`

Swap the values of the two top elements of the register stack


### DUP

`dup` - `000111`

Duplicate from the top register stack element, pushing it onto the register stack


### DUP2

`dup2` - `001000`

Duplicate the second from the top element of the register stack, pushing it onto the register stack


### MOV

`mov $num` - `100010`

Push the value of `num` onto the register stack


### PUSH

`push` - `001001`

`push %FR` (old assembler's `pushf`) - `001010`

Copy the value of the flag register or pop a value from the register stack, push the value onto the memory stack


### POP

`pop` - `001100`

`pop %FR` (old assembler's `popf`) - `001101`

Pop a value from the memory stack and set the flag register to it or push it onto the register stack


### ADD

`add` - `001110`

Pop two items from the register stack, add their values, pushing the result onto the register stack.

Reset the flag register, then set the `CF` to 1 if the result has an extra carry, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

Note: the implementation isn't at all working as expected, as the `change_flag_result` function takes in the truncated result.

Note: further operations' TODOs say "as if it's addition"; `OF` isn't affected how you'd expect it to with addition for them - we compute (most_significant_bit_a $=$ most_significant_bit_b and most_significant_bit_result $\ne$ most_significant_bit_a). [See implementation](https://github.com/ucu-computer-science/Hardware-Simulator-and-Assembler/blob/master/modules/functions.py#L430)


### SUB

`sub` - `001111`

Pop two items from the register stack, subtracting the second one (in order of popping) from the first, and push the result (the two's complement) onto the register stack.

Reset the flag register, then set the `CF` to 1 if the result is negative, `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).


### MUL

`mul` - `010000`

Pop two items from the register stack, multiply them, and push the result onto the register stack. Set the `CF` to 1 if there is an unsigned overflow (the upper half of the result isn't 0), the `OF` if there is a signed overflow (the sign complement of the truncated result is not the same as the non-truncated result), the `ZF` to 1 if the truncated result is 0 and the `SF` to 1 if the sign of the result is negative (even if it was truncated).

TODO: figure out why the implementation sets all the other flags as if it's addition

TODO: figure out whether setting the `ZF` to 0 only if the second half of the result is 0 is intended


### DIV

`div` - `010001`

Pop two items from the register stack, dividing the first one popped by the second, and push the result onto the register stack. Purely unsigned.

Reset the flag register, and set the `ZF` to 0 if the result is truly 0 (and not rounded).

If zero division is encountered, set the result, the `CF` and `OF` to all-1's and the rest of the flags to 0.

TODO: figure out whether the implementation ever sets the `CF` to 1

TODO: figure out why the implementation sets all the flags other than `CF` as if it's addition

TODO: figure out whether setting `ZF` should work like this or like the implementation (only looking at the rounded result)


### AND

`and` - `010010`

Compute binary `and` between two popped values from the register stack, push the result onto the register stack.

TODO: figure out why this operation modifies the flag register in the implementation as if it's addition, except setting `CF` (but not `OF`) to 0


### OR

`or` - `010011`

Compute binary `or` between two popped values from the register stack, pushe the result onto the register stack.

TODO: figure out why flags are modified


### XOR

`xor` - `010100`

Compute binary `xor` between two popped values from the register stack, push the result onto the register stack.

TODO: figure out why flags are modified


### NOT

`not` - `010101`

Replace the top of the register stack with its' bitwise not.

Reset the flag register, and set the `OF` to 1, the `ZF` to 1 if the result is 0.

Flip the value of `SF`

TODO: figure out whether flags should be modified

TODO: think about setting `ZF`


### LSH

`lsh $num` - `100011`

Pop a value from the register stack, shift it by `num` to the left ($x = x*2^n$), and push the result onto the register stack.

Reset the flag register, and set the `CF` to 1 if any 1's are shifted out and the `ZF` if the truncated result is 0.

TODO: figure out why the implementation sets all the other flags as if it's addition

TODO: figure out whether we should set the `OF` to the shifted out bit for 1-bit shifts


### RSH

`rsh $num` - 100100

Pop a value from the register stack, shift it by `num` to the right, and push the result onto the register stack.

Reset the flag register, and set the `CF` to 1 if any 1's are shifted out and the `ZF` if the truncated result is 0.

TODO: figure out why the implementation sets all the other flags as if it's addition

TODO: figure out whether we should set the `OF` to the shifted out bit for 1-bit shifts


### CALL

`call $num` (`call label`) - `100101`

`call` - `010110`

Push the next instruction's *absolute* location (the current `PC`) onto the memory stack, pop a value from the register stack if it's not provided, and set it as the *relative* address to execute next (add it to the `PC` after incrementing it; `call 0` pushes the instruction; `call -3` is an infinite loop)


### RET

`ret` - `010111`

Pop an address from the memory stack and set it as the *absolute* address to execute next (set `PC` to it instead of incrementing it).


### CMPE

`cmpe` (compare equals) - `011000`

`cmpe $imm` - `100110`

Pop a value (or two if the immediate isn't provided) from the register stack, compare them and push the result onto the register stack as a boolean (`0xffff` if they are equal, `0x0000` otherwise).


### CMPB

`cmpb` (compare bigger) - `011001`

`cmpb $imm` - `100111`

Pop a value (or two if the immediate isn't provided) from the register stack, compare them and push the result onto the register stack as a boolean (`0xffff` if the first one popped is larger than the second one popped or the immediate, `0x0000` otherwise).


### JMP

`jmp` - `011010`

`jmp $imm` - `101000`

Pop a value from the register stack if it's not provided, and set it as the *relative* address to execute next (add it to the `PC` after incrementing it; `jmp 0` does nothing; `jmp -3` is an infinite loop)


### JC

`jc` - `011011`

`jc $num` - `101001`

Pop a value from the register stack. If it's a boolean true (`0xffff`):
- pop a value from the register stack if an immediate isn't provided
- set it as the *relative* address to execute next (add it to the `PC` after incrementing it; `jc 0` does nothing; `jc -3` loops until all boolean true values are popped)


### IN

`in $imm` - `101010`

Hang the processor, waiting for the single-character user input on the device at the port specified, then push it onto the register stack

TODO: remove this.


### OUT

`out $imm` - `101011`

Pop a value from the register stack and output it on the specified port


### NOP

`nop` - `011100`

Do not do anything
