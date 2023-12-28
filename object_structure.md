# The structure of the object files

`Object`:
- `Header`:
    - `Parameters`:
        - `Opcode size` - the number of bits in an opcode
        - `Text byte` - the number of bits in a byte of text
        - `Data byte` - the number of bits in a byte of data
        - `Text address` - the number of bits in a text address
        - `Data address` - the number of bits in a data address
    - `Section table size` - the number of sections in the segment
- `Section table`:
    - `Section table entries`, each:
        - `Section type` - data, text, symbol table or relocation table
        - `Section size` - the number of bytes in a section (for text or data) or the number of symbols (for Symbol table or Relocation table)
- `Sections` (Text, Data, Symbol table or Relocation table)

`Text`, `Data`:
- Just a sequence of bytes

`Symbol table`:
- Each `symbol` (a symbol is basically a label):
    - `Section id` - in which section the symbol is located
    - `Offset` - the offset (in bytes) in the section to the symbol
    - `Name address` - the location (in bits), where to find the name of the symbol in the `String`
- `String` - concatinated names of the symbols (in format: `name1\0name2\0name3\0`)

`Relocation table`:
- Each `relocation`:
    - `Location`:
        - `Section` - the id of section in which the relocation is located
        - `Offset` - the offset (in bytes) in the section to byt of the relocation
    - `Name address` - the location (in bits), where to find the name of the symbol in the `String` o the `Symbol table`
    - `Symbol size` - how many bits should be replaced
    - `Symbol offset` - the offset inside the byte, where the bits should be replaced
    - `Relative` - if the relocation is relative or absolute (if relative, it will make the relocation position independently)
