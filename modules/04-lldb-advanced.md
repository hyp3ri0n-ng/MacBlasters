# Module 4: Advanced LLDB Techniques

## Overview

This module dives deep into advanced LLDB features including Python scripting, custom commands, debugging stripped binaries, and advanced analysis techniques essential for security research and reverse engineering.

## Learning Objectives

By the end of this module, you will be able to:
- Write Python scripts to extend LLDB functionality
- Create custom LLDB commands
- Debug stripped binaries without symbols
- Perform advanced memory analysis
- Use LLDB for dynamic analysis and instrumentation
- Remote debug applications
- Automate repetitive debugging tasks

## Topics

### 4.1 LLDB Python Scripting

LLDB has a powerful Python API that allows you to extend its functionality.

#### Accessing Python in LLDB

```bash
# Enter Python REPL
(lldb) script
>>> import lldb
>>> print(lldb.debugger)
>>> exit()

# Execute Python code
(lldb) script print("Hello from Python")

# Execute Python file
(lldb) command script import /path/to/script.py
```

#### Basic Python Script Example

```python
# lldb_script.py
import lldb

def hello_world(debugger, command, result, internal_dict):
    """Say hello from LLDB"""
    print("Hello from custom LLDB command!")
    result.AppendMessage("This is a custom command")

def __lldb_init_module(debugger, internal_dict):
    """Initialize the module when loaded in LLDB"""
    debugger.HandleCommand(
        'command script add -f lldb_script.hello_world hello'
    )
    print('The "hello" command has been installed')
```

Usage:
```bash
(lldb) command script import lldb_script.py
(lldb) hello
```

#### Accessing Process Information

```python
import lldb

def show_process_info(debugger, command, result, internal_dict):
    """Display detailed process information"""
    target = debugger.GetSelectedTarget()
    process = target.GetProcess()
    
    result.AppendMessage(f"Process ID: {process.GetProcessID()}")
    result.AppendMessage(f"State: {lldb.SBDebugger.StateAsCString(process.GetState())}")
    result.AppendMessage(f"Number of threads: {process.GetNumThreads()}")
    
    # Iterate through threads
    for thread in process:
        result.AppendMessage(f"Thread {thread.GetThreadID()}: {thread.GetName()}")

def __lldb_init_module(debugger, internal_dict):
    debugger.HandleCommand(
        'command script add -f script.show_process_info procinfo'
    )
```

#### Reading Memory with Python

```python
import lldb

def dump_memory(debugger, command, result, internal_dict):
    """Dump memory at address"""
    # command contains the address
    try:
        addr = int(command, 16)
    except ValueError:
        result.SetError("Invalid address format")
        return
    
    target = debugger.GetSelectedTarget()
    process = target.GetProcess()
    
    error = lldb.SBError()
    data = process.ReadMemory(addr, 256, error)
    
    if error.Success():
        # Display as hex dump
        for i in range(0, len(data), 16):
            hex_str = ' '.join(f'{b:02x}' for b in data[i:i+16])
            ascii_str = ''.join(chr(b) if 32 <= b < 127 else '.' for b in data[i:i+16])
            result.AppendMessage(f"{addr+i:016x}  {hex_str:<48}  {ascii_str}")
    else:
        result.SetError(f"Failed to read memory: {error}")

def __lldb_init_module(debugger, internal_dict):
    debugger.HandleCommand(
        'command script add -f script.dump_memory hexdump'
    )
```

### 4.2 Custom Commands and Aliases

#### Creating Aliases

```bash
# Simple alias
(lldb) command alias bm breakpoint set -n

# Alias with argument
(lldb) command alias mema memory read -c %1 -s 1

# Usage
(lldb) mema 256 0x100001000

# Unalias
(lldb) command unalias bm
```

#### Regex Commands

```bash
# Create regex command to set breakpoint with short syntax
(lldb) command regex bp
# Add pattern: s/(.+)/breakpoint set --name %1/
# Now you can use: bp main

# Another useful regex command
(lldb) command regex rlook
# Pattern: s/(.+)/image lookup --regex %1/
```

### 4.3 Debugging Stripped Binaries

Stripped binaries have no symbol information, making debugging harder.

#### Identifying Stripped Binaries

```bash
# Check if binary is stripped
nm /path/to/binary
# If output is "no symbols", it's stripped

# Or use file command
file /path/to/binary
# Will show "stripped" if symbols removed
```

#### Techniques for Stripped Binaries

**1. Find Entry Point**
```bash
(lldb) image dump header executable
# Look for entryoff field

# Or use otool
otool -l binary | grep -A 3 LC_MAIN
```

**2. Use Dynamic Libraries**
```bash
# Break on library functions
(lldb) b malloc
(lldb) b free
(lldb) b printf
(lldb) b objc_msgSend

# These work because libc/objc are not stripped
```

**3. Use Address Breakpoints**
```bash
# Calculate ASLR offset
(lldb) image list -o -f
# Shows offset for each module

# Set breakpoint at offset address
(lldb) br s -a 0x100001234
```

**4. Trace Execution**
```bash
# Step through instructions
(lldb) si
(lldb) di -p -c 10    # Disassemble 10 instructions from PC

# Create trace script
(lldb) br set -a 0x100001000
(lldb) br command add 1
> di -p -c 1
> register read rip
> continue
> DONE
```

**5. Reconstruct Symbols**

Use pattern recognition and heuristics:
```python
import lldb

def find_functions(debugger, command, result, internal_dict):
    """Attempt to identify function boundaries in stripped binary"""
    target = debugger.GetSelectedTarget()
    module = target.GetModuleAtIndex(0)
    
    text_section = module.FindSection("__TEXT").FindSubSection("__text")
    start_addr = text_section.GetLoadAddress(target)
    size = text_section.GetByteSize()
    
    # Look for function prologues (push rbp; mov rbp, rsp)
    # x86_64: 55 48 89 e5
    # ARM64: different pattern
    
    process = target.GetProcess()
    error = lldb.SBError()
    data = process.ReadMemory(start_addr, size, error)
    
    if error.Success():
        func_count = 0
        for i in range(len(data) - 4):
            if data[i:i+4] == b'\x55\x48\x89\xe5':
                result.AppendMessage(f"Possible function at: 0x{start_addr + i:x}")
                func_count += 1
        result.AppendMessage(f"Found {func_count} possible functions")
```

### 4.4 Memory Analysis

#### Finding Strings

```bash
# Search for string in memory
(lldb) memory find -s "password" -- 0x100000000 0x200000000

# Search for pattern
(lldb) memory find -e '\x90\x90\x90' -- 0x100000000 0x200000000
```

#### Scanning Memory Regions

```python
import lldb

def scan_memory(debugger, command, result, internal_dict):
    """Scan memory regions for readable content"""
    target = debugger.GetSelectedTarget()
    process = target.GetProcess()
    
    # Get memory regions
    region_list = process.GetMemoryRegions()
    
    for i in range(region_list.GetSize()):
        region = lldb.SBMemoryRegionInfo()
        region_list.GetMemoryRegionAtIndex(i, region)
        
        if region.IsReadable():
            start = region.GetRegionBase()
            end = region.GetRegionEnd()
            size = end - start
            
            result.AppendMessage(
                f"Region: 0x{start:x} - 0x{end:x} "
                f"({size} bytes) "
                f"R{'W' if region.IsWritable() else '-'}"
                f"{'X' if region.IsExecutable() else '-'}"
            )
```

#### Memory Protection

```bash
# Check memory protection
(lldb) memory region 0x100001000

# Output shows:
# [0x100000000-0x100001000) r-x
```

### 4.5 Dynamic Analysis and Instrumentation

#### Method Swizzling Detection

```python
import lldb

def check_swizzling(debugger, command, result, internal_dict):
    """Check if Objective-C methods have been swizzled"""
    # Break on method_exchangeImplementations
    target = debugger.GetSelectedTarget()
    breakpoint = target.BreakpointCreateByName("method_exchangeImplementations")
    
    # Add callback
    def callback(frame, bp_loc, dict):
        # Get arguments (class, sel1, sel2)
        arg0 = frame.FindRegister("rdi")
        arg1 = frame.FindRegister("rsi")
        
        print(f"Method swizzling detected!")
        print(f"Class: {arg0.GetValue()}")
        print(f"Selector 1: {arg1.GetValue()}")
        return False  # Don't stop
    
    breakpoint.SetScriptCallbackFunction("script.callback")
```

#### Hooking Functions

```bash
# Break before function
(lldb) br s -n malloc
(lldb) br command add 1
> expr -- (void*)$arg1  # First argument (size)
> continue
> DONE

# Break after function (on return)
(lldb) br s -n malloc
(lldb) br command add 1
> fin  # Step out
> expr -- (void*)$rax  # Return value
> continue
> DONE
```

### 4.6 Remote Debugging

#### iOS Device Debugging

```bash
# Connect to iOS device
(lldb) platform select remote-ios
(lldb) platform connect connect://[device-ip]:port

# Launch app
(lldb) platform shell open [bundle-id]
(lldb) platform process attach --name MyApp
```

#### Network Debugging

```bash
# Start debug server on remote machine
debugserver localhost:1234 /path/to/binary

# Connect from local machine
lldb
(lldb) platform select remote-macosx
(lldb) platform connect connect://remote-host:1234
(lldb) target create /local/path/to/binary
(lldb) process connect connect://remote-host:1234
```

### 4.7 Automation and Scripting

#### LLDB Init File

Create `~/.lldbinit`:

```python
# ~/.lldbinit

# Load custom commands
command script import ~/lldb-scripts/custom_commands.py

# Set aliases
command alias bd breakpoint disable
command alias be breakpoint enable
command alias bl breakpoint list
command alias po expression -O --

# Settings
settings set target.x86-disassembly-flavor intel
settings set target.skip-prologue false

# Pretty printing
type summary add -F lldb_script.custom_summary MyClass
```

#### Batch Processing

```bash
# Run LLDB with commands from file
lldb -s commands.txt program

# commands.txt:
# breakpoint set -n main
# run
# next
# next
# print variable
# continue
```

#### Scripted Breakpoints

```python
def breakpoint_callback(frame, bp_loc, dict):
    """Custom breakpoint handler"""
    # Get function name
    func_name = frame.GetFunctionName()
    
    # Get arguments (x86_64 calling convention)
    arg1 = frame.FindRegister("rdi").GetValueAsUnsigned()
    arg2 = frame.FindRegister("rsi").GetValueAsUnsigned()
    
    print(f"Hit {func_name} with args: {arg1}, {arg2}")
    
    # Modify behavior
    if arg1 == 0:
        # Change argument
        frame.FindRegister("rdi").SetValueFromCString("1")
        print("Modified argument!")
    
    # Return False to continue, True to stop
    return False
```

### 4.8 Performance Analysis

#### Counting Function Calls

```python
import lldb

call_counts = {}

def count_calls(frame, bp_loc, dict):
    """Count function calls"""
    func = frame.GetFunctionName()
    call_counts[func] = call_counts.get(func, 0) + 1
    return False  # Don't stop

def show_stats(debugger, command, result, internal_dict):
    """Display call statistics"""
    for func, count in sorted(call_counts.items(), key=lambda x: x[1], reverse=True):
        result.AppendMessage(f"{func}: {count} calls")
```

#### Timing Functions

```python
import time

entry_times = {}

def time_function_entry(frame, bp_loc, dict):
    """Record entry time"""
    func = frame.GetFunctionName()
    entry_times[func] = time.time()
    return False

def time_function_exit(frame, bp_loc, dict):
    """Calculate duration"""
    func = frame.GetFunctionName()
    if func in entry_times:
        duration = time.time() - entry_times[func]
        print(f"{func} took {duration:.6f} seconds")
        del entry_times[func]
    return False
```

### 4.9 Anti-Debugging Detection and Bypass

#### Common Anti-Debug Techniques

1. **ptrace check (P_TRACED flag)**
2. **sysctl check**
3. **Debugger check via task_get_exception_ports**
4. **Timing checks**

#### Bypassing Anti-Debug

```python
def bypass_ptrace_check(debugger, command, result, internal_dict):
    """Bypass ptrace detection"""
    # Break on ptrace
    target = debugger.GetSelectedTarget()
    bp = target.BreakpointCreateByName("ptrace")
    
    # Modify return value
    def callback(frame, bp_loc, dict):
        # Finish the function
        thread = frame.GetThread()
        thread.StepOut()
        
        # Set return value to 0 (success)
        frame = thread.GetFrameAtIndex(0)
        frame.FindRegister("rax").SetValueFromCString("0")
        
        return False
    
    bp.SetScriptCallbackFunction("script.callback")
```

## Practical Exercises

### Exercise 1: Python Script Development

1. Write a script that monitors all calls to `malloc` and `free`
2. Track memory allocations and detect leaks
3. Display statistics when program exits

### Exercise 2: Stripped Binary Analysis

1. Find a stripped binary (or strip one with `strip`)
2. Find the entry point
3. Identify function boundaries
4. Set breakpoints at key functions
5. Reconstruct program flow

### Exercise 3: Custom Command Creation

1. Create a custom command that:
   - Takes a function name as argument
   - Sets a breakpoint
   - Logs all calls with arguments
   - Displays statistics

### Exercise 4: Memory Forensics

1. Attach to a running process
2. Scan memory for sensitive data (passwords, keys)
3. Dump interesting memory regions
4. Analyze data structures

### Exercise 5: Dynamic Instrumentation

1. Choose a system binary
2. Identify interesting functions
3. Hook them with custom handlers
4. Log their behavior
5. Modify their return values

### Exercise 6: Anti-Debug Bypass

1. Create a program with anti-debugging
2. Detect the anti-debug mechanism
3. Write LLDB script to bypass it
4. Verify bypass works

## Key Takeaways

- LLDB's Python API enables powerful automation and analysis
- Custom commands can streamline repetitive tasks
- Stripped binaries can be analyzed with pattern recognition
- Memory analysis reveals runtime behavior
- Dynamic instrumentation enables deep inspection
- Anti-debugging can often be bypassed with LLDB

## Further Reading

- [LLDB Python Reference](https://lldb.llvm.org/python_reference/index.html)
- [LLDB Python API Examples](https://github.com/llvm/llvm-project/tree/main/lldb/examples/python)
- [Advanced Debugging Techniques (Apple WWDC)](https://developer.apple.com/videos/)
- [The LLDB Debugger (blog posts)](https://developer.apple.com/library/archive/documentation/IDEs/Conceptual/gdb_to_lldb_transition_guide/)

## Next Module

[Module 5: macOS Security Architecture →](05-security-architecture.md)
