# bfb
Version of brainfuck with reworked file IO

## Instructions
- `+` Increments the byte under the tape pointer
- `-` Decrements the byte under the tape pointer
- `>` Increments the tape pointer
- `<` Decrements the tape pointer
- `[` Conditional jump to the instruction after it's corresponding `]` instruction, jumps if the byte under the tape pointer is `0`
- `]` Unconditional jump to it's corresponding `[` instruction
- `,` Pull from the top of the Interface Stack and overwrite the byte under the tape pointer
- `.` Push the byte under the pointer onto the Interface Stack, (does not consume values on the tape)
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

Numbers composed of numerous bytes are to be positioned in the Interface Stack so that the most significant byte is at the top, this means that to push an 8 byte number starting at the current position of the tape pointer, this bfb code could be used
```bf
.>.>.>.>.>.>.>.
```

Pulling off of the Interface Stack after reaching the bottom results in undefined behaviour

## Interface Stack Opcodes

| Op| Name          | Arg 1                 | Arg 2             | Arg 3                   | Ret 1 |
| - | ------------- | --------------------- | ----------------- | ----------------------- | ----- |
| 0 | Read8         | `int8 fd`             | `int8 count`      | `ptr char[] buf`        |
| 1 | Write8        | `int8 fd`             | `int8 count`      | `ptr char[] buf`        |
| 2 | Open8         | `ptr char[] filename` | `int8 file_mode`  |                         | `int8 fd`
| 3 | Close8        | `int8 fd`             |                   |                         |
| 4 | TapeOrigin    |                       |                   |                         | `int64 origin_pointer`
| 5 | TapePos       |                       |                   |                         | `int64 pointer_value`
| 6 | Fork          |                       |                   |                         | `bool forked`
| 7 | PID           |                       |                   |                         | `int64 PID`
| 8 | Exit          | `int64 exit_code`     |                   |                         |

`ptr` is a 64bit unsigned integer signifying a location in memory represented by pushing 8 values onto the Interface Stack

`bool` is 1 byte, `0` is false and any value that is not `0` is true

`int64` is an unsigned 64 bit integer

`int8` is an unsigned 8 bit integer

In interpreted implementations the memory location for the tape origin can be `0`

With `f(arg1, arg2)` notation, the stack is arganised as follows
```
f
arg1
arg2
...
```

### Read8
```
read8(int8 fd, int8 count, ptr char[] buf)
Opcode: 0
```

Programs using this call to read files can only read from file descriptors that fit within 1 byte

#### fd
As the file descriptor is stored as 1 byte, only 256 individual files can be read from using this opcode

The file descriptor returned by Open8 should be used with this call

File descriptors returned by other Open* calls do not have to work with this call

#### count
As this argument is only 1 byte, it means that with this call a max of only 256 bytes can be read at a time

If this argument is `0` then it will result in undefined behaviour

#### buf
This argument is formed of 8 bytes, representing a 64 bit pointer to a memory location

`TapePos()` can be used to push the position of a buffer

#### Code Example
```bf
read from STDIN

We will read 1 byte from STDIN and store it in the byte under the tape pointer

Get the tape pointer onto the stack
+++++ 5, call tapePos()
.     Push onto the stack
%     Call the stack

----  1, count argument
.     Push onto the stack

-     0, fd argument, 0 is the fd for STDIN
.     Push onto the stack

%     Call the stack
```

### Write8
```
write8(int8 fd, ptr char[] buf)
Opcode: 1
```

#### fd
As the file descriptor is stored as 1 byte, only 256 individual files can be written to using this opcode

The file descriptor returned by Open8 should be used with this call

File descriptors returned by other Open* calls do not have to work with this call

#### count
As this argument is only 1 byte, it means that with this call a max of only 256 bytes can be written at a time

If this argument is `0` then it will result in undefined behaviour

#### buf
This argument is formed of 8 bytes, representing a 64 bit pointer to a memory location

`TapePos()` can be used to push the position of a buffer

#### Code Example
```bf
Write to STDOUT

We will store the value 69 ('E') in the cell under the pointer and then write it to STDOUT

Get the tape pointer onto the stack
```

### Open8
#### file_mode

- `0` opens for reading, will return a status of `1`
- `1` opens and truncates a file for writing

#### status

## Useful constants
```
== File Descriptors ==
STDIN   0
STDOUT  1
STDERR  2
```
