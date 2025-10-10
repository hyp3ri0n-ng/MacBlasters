# Module 3: Introduction to LLDB

## Overview

LLDB (Low Level Debugger) is the default debugger on macOS and is part of the LLVM project. It's an essential tool for reverse engineering, security research, and understanding how programs work at a low level. This module covers the fundamentals of using LLDB to debug and analyze applications.

## Learning Objectives

By the end of this module, you will be able to:
- Launch and attach to processes with LLDB
- Navigate through program execution
- Set and manage breakpoints
- Examine memory and registers
- Understand the program call stack
- Use LLDB to reverse engineer binaries

## Topics

### 3.1 Introduction to LLDB

#### What is LLDB?

LLDB is a next-generation, high-performance debugger built on top of LLVM compiler infrastructure. It replaced GDB as the default debugger on macOS.

**Key features:**
- Full source-level debugging
- Disassembly and low-level debugging
- Python scriptability
- Remote debugging support
- Expression evaluation
- Thread and process control

#### Installing LLDB

LLDB comes with Xcode Command Line Tools:

```bash
# Install Command Line Tools
xcode-select --install

# Verify installation
lldb --version

# Or use system lldb
which lldb
# /usr/bin/lldb
```

#### LLDB vs GDB

| Feature | LLDB | GDB |
|---------|------|-----|
| Default on macOS | Yes | No (deprecated) |
| Python scripting | Yes | Limited |
| Syntax | More consistent | Historical inconsistencies |
| Performance | Generally faster | Mature |
| macOS integration | Excellent | Limited |

### 3.2 Getting Started with LLDB

#### Launching LLDB

```bash
# Start LLDB
lldb

# Launch a program
lldb /bin/ls
(lldb) run
(lldb) r       # short form

# Launch with arguments
lldb -- /usr/bin/grep "pattern" file.txt
(lldb) run

# Or in one command
lldb /bin/ls -- -la /tmp
```

#### Attaching to a Running Process

```bash
# Attach by process ID
lldb -p <PID>
(lldb) process attach --pid <PID>

# Attach by process name
lldb -n ProcessName
(lldb) process attach --name "Safari"

# Wait for process to launch
(lldb) process attach --name "MyApp" --waitfor
```

#### Basic Navigation

```bash
# Show help
(lldb) help
(lldb) help breakpoint    # Help for specific command

# Apropos - search help
(lldb) apropos thread
(lldb) apropos memory

# Command abbreviation
(lldb) br  # breakpoint
(lldb) p   # print/expression
(lldb) po  # print object (Objective-C/Swift)
```

### 3.3 Controlling Program Execution

#### Basic Commands

```bash
# Continue execution
(lldb) continue
(lldb) c

# Step over (execute current line, don't enter functions)
(lldb) next
(lldb) n

# Step into (enter functions)
(lldb) step
(lldb) s

# Step out (finish current function)
(lldb) finish
(lldb) fin

# Step instruction (assembly level)
(lldb) stepi
(lldb) si

# Next instruction (assembly level, step over calls)
(lldb) nexti
(lldb) ni

# Run until line number
(lldb) thread until 100

# Jump to line (dangerous!)
(lldb) thread jump --line 50
```

#### Process Control

```bash
# Kill the process
(lldb) kill

# Detach from process (let it continue)
(lldb) detach

# Exit LLDB
(lldb) quit
(lldb) exit
```

### 3.4 Breakpoints

Breakpoints pause execution at specific locations.

#### Setting Breakpoints

```bash
# Break at function name
(lldb) breakpoint set --name main
(lldb) br s -n main          # short form
(lldb) b main                # shortest form

# Break at method (Objective-C)
(lldb) br s -n "-[MyClass myMethod:]"

# Break at C++ method
(lldb) br s -n "MyClass::myMethod()"

# Break at address
(lldb) br s -a 0x100001000

# Break at file:line
(lldb) br s -f main.c -l 42
(lldb) br s --file main.c --line 42

# Break with regular expression
(lldb) br s -r "^NS.*init"   # All NSObject init methods

# Break on all C++ throw
(lldb) br s -E c++

# Break on all Objective-C exceptions
(lldb) br s -E objc

# Break on selector
(lldb) br s -S "description"  # All description methods
```

#### Managing Breakpoints

```bash
# List breakpoints
(lldb) breakpoint list
(lldb) br list

# Delete breakpoint
(lldb) breakpoint delete 1
(lldb) br del 1

# Disable/Enable breakpoint
(lldb) breakpoint disable 1
(lldb) breakpoint enable 1

# Delete all breakpoints
(lldb) breakpoint delete

# Set condition on breakpoint
(lldb) br modify -c "x > 10" 1

# Set ignore count
(lldb) br modify -i 5 1       # Ignore first 5 hits

# Add commands to breakpoint
(lldb) br command add 1
> p x
> continue
> DONE
```

#### Watchpoints

Watch for memory access/modification.

```bash
# Watch variable
(lldb) watchpoint set variable myVar
(lldb) w s v myVar

# Watch expression
(lldb) watchpoint set expression &myVar

# Watch address
(lldb) watchpoint set expression -w write -- 0x12345678

# List watchpoints
(lldb) watchpoint list
(lldb) w list

# Delete watchpoint
(lldb) watchpoint delete 1
(lldb) w del 1

# Watch options:
# -w read       # Break on read
# -w write      # Break on write (default)
# -w read_write # Break on both
# -s size       # Size in bytes
```

### 3.5 Examining Memory and Registers

#### Reading Memory

```bash
# Read memory at address
(lldb) memory read 0x100001000
(lldb) x 0x100001000         # x is alias for memory read

# Read with format
(lldb) x/10xb 0x100001000    # 10 bytes in hex
(lldb) x/4xw 0x100001000     # 4 words in hex
(lldb) x/s 0x100001000       # String

# Format specifiers:
# x - hex
# d - decimal
# u - unsigned decimal
# o - octal
# t - binary
# f - float
# c - char
# s - string

# Size specifiers:
# b - byte (1)
# h - halfword (2)
# w - word (4)
# g - giant word (8)

# Examples:
(lldb) x/10i 0x100001000     # 10 instructions
(lldb) x/40xb 0x100001000    # 40 bytes in hex
(lldb) x/10gx 0x100001000    # 10 64-bit values in hex
```

#### Writing Memory

```bash
# Write to memory
(lldb) memory write 0x100001000 0x90
(lldb) memory write 0x100001000 0x90 0x90 0x90

# Write string
(lldb) memory write -s "Hello" 0x100001000
```

#### Registers

```bash
# Read all registers
(lldb) register read
(lldb) re r

# Read specific register
(lldb) register read rax
(lldb) re r rax

# Read register set
(lldb) register read --all    # All registers including vectors

# Write register
(lldb) register write rax 0x1234
(lldb) re w rax 0x1234

# Common x86_64 registers:
# rax, rbx, rcx, rdx     - General purpose
# rsi, rdi               - Source/destination index
# rsp                    - Stack pointer
# rbp                    - Base pointer
# rip                    - Instruction pointer
# rflags                 - Flags register
# xmm0-xmm15            - SSE registers

# ARM64 registers:
# x0-x30                 - General purpose
# sp                     - Stack pointer
# pc                     - Program counter
# cpsr                   - Current program status register
```

### 3.6 Stack and Frame Navigation

#### Call Stack

```bash
# Show backtrace (call stack)
(lldb) thread backtrace
(lldb) bt

# Show all threads
(lldb) thread list

# Backtrace all threads
(lldb) thread backtrace all
(lldb) bt all

# Select thread
(lldb) thread select 2

# Show current thread info
(lldb) thread info
```

#### Stack Frames

```bash
# List frames
(lldb) frame select 0
(lldb) f 0

# Show frame info
(lldb) frame info

# Show frame variables
(lldb) frame variable
(lldb) fr v

# Show specific variable
(lldb) frame variable myVar
(lldb) fr v myVar

# Show all locals
(lldb) frame variable -a

# Move up/down stack
(lldb) up
(lldb) down
```

### 3.7 Expressions and Variables

#### Evaluating Expressions

```bash
# Print expression
(lldb) expression x + y
(lldb) expr x + y
(lldb) p x + y           # print is alias

# Print with format
(lldb) p/x value         # Hex
(lldb) p/t value         # Binary
(lldb) p/d value         # Decimal
(lldb) p/c value         # Character

# Call function
(lldb) expr (int)printf("Hello\n")
(lldb) p (int)puts("Test")

# Create variable
(lldb) expr int $myvar = 42
(lldb) p $myvar

# Modify variable
(lldb) expr x = 100
(lldb) p x = 100
```

#### Examining Variables

```bash
# Print variable
(lldb) p myVar

# Print pointer contents
(lldb) p *ptr

# Print array
(lldb) p myArray[0]
(lldb) p myArray[0]@10   # Print 10 elements

# Print structure
(lldb) p myStruct
(lldb) p myStruct.field

# Print Object (Objective-C/Swift)
(lldb) po myObject       # Calls description method

# Print with type info
(lldb) ptype myVar

# Print raw bytes
(lldb) memory read -s 16 &myVar
```

### 3.8 Disassembly

```bash
# Disassemble current function
(lldb) disassemble
(lldb) di

# Disassemble function by name
(lldb) disassemble --name main
(lldb) di -n main

# Disassemble address range
(lldb) disassemble --start-address 0x100001000 --end-address 0x100001100
(lldb) di -s 0x100001000 -e 0x100001100

# Disassemble with bytes
(lldb) di -b

# Disassemble with line info
(lldb) di -m

# Show frame disassembly
(lldb) disassemble --frame
(lldb) di -f

# Disassemble around PC
(lldb) disassemble --pc
```

### 3.9 Image (Binary) Information

```bash
# List loaded modules
(lldb) image list
(lldb) im li

# Search for module
(lldb) image list Foundation

# Show module details
(lldb) image dump sections main
(lldb) image dump symtab main

# Look up symbol
(lldb) image lookup --name malloc
(lldb) im loo -n malloc

# Look up address
(lldb) image lookup --address 0x100001000
(lldb) im loo -a 0x100001000

# Show module slide (ASLR offset)
(lldb) image list -o -f
```

### 3.10 Practical Debugging Workflow

#### Example: Debugging a Simple Program

```c
// test.c
#include <stdio.h>

int add(int a, int b) {
    return a + b;
}

int main() {
    int x = 5;
    int y = 10;
    int result = add(x, y);
    printf("Result: %d\n", result);
    return 0;
}
```

Compile and debug:

```bash
# Compile with debug symbols
clang -g -O0 test.c -o test

# Start LLDB
lldb test

# Set breakpoint
(lldb) b main
(lldb) b add

# Run
(lldb) run

# At main breakpoint
(lldb) frame variable        # Show local variables
(lldb) next                  # Step over
(lldb) step                  # Step into add()

# At add breakpoint
(lldb) p a
(lldb) p b
(lldb) finish                # Return to main

# Continue
(lldb) continue
```

## Practical Exercises

### Exercise 1: Basic Debugging

1. Create a simple C program with a bug
2. Compile with debug symbols: `clang -g program.c -o program`
3. Load in LLDB: `lldb program`
4. Set breakpoint at main
5. Run and step through the program
6. Find and fix the bug

### Exercise 2: Analyzing System Binaries

1. Attach LLDB to a running process: `lldb -p $(pgrep Calculator)`
2. Show the backtrace
3. List all loaded images
4. Find the main executable's load address
5. Disassemble the main function
6. Set a breakpoint and trigger it
7. Detach without killing the process

### Exercise 3: Memory Inspection

1. Launch `/bin/ls` in LLDB
2. Break at main
3. Examine the stack pointer (rsp/sp)
4. Dump 100 bytes from the stack
5. Find strings in memory
6. Examine environment variables

### Exercise 4: Breakpoint Practice

1. Create a program with multiple functions
2. Set breakpoints in different ways:
   - By function name
   - By file and line
   - By address
3. Add conditions to breakpoints
4. Add commands to run at breakpoints
5. Practice enabling/disabling breakpoints

### Exercise 5: Register Manipulation

1. Launch a simple program in LLDB
2. Break at main
3. Read all registers
4. Modify a register value
5. Continue execution and observe the effect
6. Experiment with instruction pointer manipulation

### Exercise 6: Advanced Breakpoints

1. Find a system framework (e.g., Foundation)
2. Set a breakpoint on all methods matching a pattern
3. Run an app that uses that framework
4. Observe which methods are called
5. Examine arguments and return values

## Key Takeaways

- LLDB is the primary debugger for macOS development and reverse engineering
- Breakpoints allow you to pause execution at specific points
- You can examine and modify memory, registers, and variables
- The stack trace shows the current execution context
- LLDB can be used on both your own code and system binaries
- Expression evaluation allows runtime code execution

## Common LLDB Commands Quick Reference

```
# Execution Control
r, run            - Start program
c, continue       - Continue execution
n, next           - Step over
s, step           - Step into
fin, finish       - Step out
si, stepi         - Step instruction
ni, nexti         - Next instruction

# Breakpoints
b, breakpoint     - Set breakpoint
br list           - List breakpoints
br del N          - Delete breakpoint N
br dis N          - Disable breakpoint N

# Information
bt                - Backtrace
f N               - Select frame N
fr v              - Frame variables
p expr            - Print expression
po obj            - Print object
x addr            - Examine memory
di                - Disassemble

# Registers & Memory
re r              - Read registers
re w reg val      - Write register
x/fmt addr        - Read memory
memory write      - Write memory

# Images & Symbols
im li             - List images
im loo -n name    - Lookup symbol
im loo -a addr    - Lookup address
```

## Further Reading

- [LLDB Official Documentation](https://lldb.llvm.org/)
- [GDB to LLDB Command Map](https://lldb.llvm.org/use/map.html)
- [LLDB Tutorial](https://lldb.llvm.org/use/tutorial.html)
- [Advanced Apple Debugging & Reverse Engineering (book)](https://www.raywenderlich.com/books/advanced-apple-debugging-reverse-engineering)

## Next Module

[Module 4: Advanced LLDB Techniques →](04-lldb-advanced.md)
