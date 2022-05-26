# bfb
Version of brainfuck with better file IO

## Instructions
- `+` Increments the byte under the tape pointer
- `-` Decrements the byte under the tape pointer
- `>` Increments the tape pointer
- `<` Decrements the tape pointer
- `[` Conditional jump to the instruction after it's corresponding `]` instruction, jumps if the byte under the tape pointer is `0`
- `]` Unconditional jump to it's corresponding `[` instruction
- `,` Pull from the top of the Interface Stack and overwrite the byte under the tape pointer
- `.` Push the byte under the pointer onto the Interface Stack, (does not overwrite the tape)
- `%` Calls the Interface Stack

## Interface Stack
The Interface Stack is the main way of handling IO in bfb

When called it should be structured like this:

```
OPCODE
ARG1
ARG2
ARG3
...
ARGN
```

After being called it should be structured like this:

```
OPCODE
RET1
RET2
...
RETN
...
```

Each Opcode has a fixed number of arguments

The stack thus may hold multiple valid consecutive calls at one time

When the `%` instruction calls the Interface Stack, the Opcode is popped

In compiled implementations the stack must be able to hold the largest possible Opcode argument combination and so requires that no Opcode have an arbitrary number of arguments

Any call to the Interface Stack must replace on the stack it's Opcode and any return values with the Opcode at the top

An Opcode cannot have a variable number of return values

Each return value must be an integer, signed or unsigned, that fits into a byte

### Opcodes

| Opcode | Name | Arg 1 | Arg 2 | Ret 1 |
| --- | --- | --- | --- | --- |
| 0 | Read  | `ptr int64 fd`  | `ltr int8 count`  | `ptr char[] buf`
| 1 | Write | `ptr int64 fd`  | `ptr char[] buf` | `int8 status`
| 2 | Open  | `ptr char[] filename` | `int8 file_mode` | `int8 status`
| 3 | Close | `ptr int64 fd`
| 4 | Tape Pointer  | | | `uint64 pointer_value`
| 5 | Fork |
| 6 | PID | | | `uint64 PID`
| 7 | Exit | `ptr int64 exit_code`

`ptr` is a 64bit unsigned integer signifying a location in memory represented by pushing 8 values onto the Interface Stack

In interpreted implementations a pointer to the start of the tape can be 0

#### file_mode

- `0` opens for reading, will return a status of `1`
- `1` opens and truncates a file for writing

#### Write call status values
- `0` write successful
- `1` wrong file mode
