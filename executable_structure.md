# The structure of executable files

The executable starts with a header:
- The first byte specifies the architecture (harward, von neumann)
- The $[1;4]$ bytes are the entry point (little endian. The extra 2 bytes were added for possible future changes. For now, if the entry point is greater than `0xffff`, it errors)
- The rest of the bytes are the segments

Each segment:
- First 17 bytes are the metadata:
    - The first 4 - where the segment start lies in the address space
    - The next 4 - the size of a single segment byte, in bits
    - The next 4 - the size of the segment in the executable file, in its own bytes
    - The next 4 - the size of the segment in the address space, in its own bytes (can be larger than the disk size - the rest is zero-filled)
    - The next 1 - the flags of the segment:
        - 0 - executable
        - 1 - writable
        - 2 - readable
        - 3 - special (whether should be mapped to the address space at all)
        - 4 - stripped (whether should remain when generating in release mode)
- The rest of the bytes are the segment data (the instructions, the data)
