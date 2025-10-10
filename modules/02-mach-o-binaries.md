# Module 2: Mach-O Binary Format

## Overview

The Mach-O (Mach Object) format is the executable format used by macOS, iOS, and other Apple operating systems. Understanding Mach-O is essential for reverse engineering, security research, and low-level macOS development.

## Learning Objectives

By the end of this module, you will be able to:
- Understand the structure of Mach-O files
- Identify and parse Mach-O headers
- Understand load commands and their purposes
- Analyze segments and sections
- Work with dynamic libraries and frameworks
- Use tools to inspect and analyze Mach-O binaries

## Topics

### 2.1 Introduction to Mach-O

#### What is Mach-O?

Mach-O is the native executable format for macOS and iOS. It's similar to ELF (Linux) and PE (Windows) formats, but with unique characteristics specific to Apple platforms.

**File types using Mach-O format:**
- Executables (apps, command-line tools)
- Dynamic libraries (.dylib)
- Frameworks (.framework)
- Bundles (.bundle)
- Kernel extensions (.kext) - deprecated
- Object files (.o)

#### Universal Binaries (Fat Files)

macOS supports universal binaries containing multiple architectures in one file:
- **x86_64**: Intel 64-bit
- **arm64**: Apple Silicon (M1/M2/M3)
- **arm64e**: Apple Silicon with pointer authentication

```bash
# Check file type
file /bin/ls
# Output: Mach-O universal binary with 2 architectures: [x86_64:Mach-O 64-bit executable x86_64] [arm64e:...]

# Extract specific architecture
lipo /bin/ls -thin x86_64 -output ls_x86_64
lipo /bin/ls -thin arm64e -output ls_arm64e

# List architectures
lipo -info /bin/ls

# Create universal binary
lipo -create binary_x86_64 binary_arm64 -output binary_universal
```

### 2.2 Mach-O File Structure

A Mach-O file consists of three main regions:

```
┌─────────────────────────────────────┐
│         Mach-O Header               │  ← Describes the file
├─────────────────────────────────────┤
│         Load Commands               │  ← Defines structure and layout
│  • LC_SEGMENT_64                    │
│  • LC_DYLD_INFO_ONLY                │
│  • LC_SYMTAB                        │
│  • LC_DYSYMTAB                      │
│  • LC_LOAD_DYLIB                    │
│  • LC_CODE_SIGNATURE                │
│  • ...                              │
├─────────────────────────────────────┤
│         Data                        │
│  ┌───────────────────────────────┐  │
│  │  Segment: __TEXT              │  │
│  │  ├─ Section: __text (code)   │  │
│  │  ├─ Section: __cstring       │  │
│  │  └─ Section: __const         │  │
│  ├───────────────────────────────┤  │
│  │  Segment: __DATA              │  │
│  │  ├─ Section: __data          │  │
│  │  ├─ Section: __bss           │  │
│  │  └─ Section: __common        │  │
│  ├───────────────────────────────┤  │
│  │  Segment: __LINKEDIT          │  │
│  │  (Symbol table, strings, etc) │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

### 2.3 Mach-O Header

The header contains metadata about the file.

#### 32-bit Header (mach_header)

```c
struct mach_header {
    uint32_t    magic;          // Magic number (0xfeedface)
    cpu_type_t  cputype;        // CPU type (CPU_TYPE_X86, CPU_TYPE_ARM, etc.)
    cpu_subtype_t cpusubtype;   // CPU subtype
    uint32_t    filetype;       // File type (MH_EXECUTE, MH_DYLIB, etc.)
    uint32_t    ncmds;          // Number of load commands
    uint32_t    sizeofcmds;     // Size of load commands region
    uint32_t    flags;          // Flags
};
```

#### 64-bit Header (mach_header_64)

```c
struct mach_header_64 {
    uint32_t    magic;          // Magic number (0xfeedfacf)
    cpu_type_t  cputype;
    cpu_subtype_t cpusubtype;
    uint32_t    filetype;
    uint32_t    ncmds;
    uint32_t    sizeofcmds;
    uint32_t    flags;
    uint32_t    reserved;       // Reserved (64-bit only)
};
```

#### Magic Numbers

- **0xfeedface**: 32-bit Mach-O
- **0xfeedfacf**: 64-bit Mach-O
- **0xcafebabe**: Universal binary (Fat file)
- **0xcafebabf**: 64-bit Universal binary

```bash
# Check magic number
xxd -l 4 /bin/ls
# Output shows: cffaedfe (little-endian 0xfeedfacf)
```

#### File Types

Common file types (defined in `filetype` field):
- **MH_EXECUTE** (0x2): Executable file
- **MH_DYLIB** (0x6): Dynamic library
- **MH_BUNDLE** (0x8): Bundle
- **MH_DYLINKER** (0x7): Dynamic linker
- **MH_OBJECT** (0x1): Object file (.o)

```bash
# View header information
otool -h /bin/ls
```

### 2.4 Load Commands

Load commands immediately follow the Mach-O header and describe the file's structure.

#### Common Load Commands

1. **LC_SEGMENT_64** / **LC_SEGMENT**: Define memory segments
2. **LC_DYLD_INFO_ONLY**: Dynamic linker information
3. **LC_SYMTAB**: Symbol table
4. **LC_DYSYMTAB**: Dynamic symbol table
5. **LC_LOAD_DYLIB**: Load a dynamic library
6. **LC_UUID**: Unique identifier for the binary
7. **LC_VERSION_MIN_MACOSX**: Minimum OS version
8. **LC_SOURCE_VERSION**: Source version
9. **LC_MAIN**: Entry point
10. **LC_ENCRYPTION_INFO_64**: Encryption information (iOS)
11. **LC_CODE_SIGNATURE**: Code signature information

```bash
# View all load commands
otool -l /bin/ls

# View specific load command
otool -l /bin/ls | grep -A 10 LC_SEGMENT
```

#### LC_SEGMENT_64 Structure

```c
struct segment_command_64 {
    uint32_t    cmd;            // LC_SEGMENT_64
    uint32_t    cmdsize;        // Command size
    char        segname[16];    // Segment name (__TEXT, __DATA, etc.)
    uint64_t    vmaddr;         // Virtual memory address
    uint64_t    vmsize;         // Virtual memory size
    uint64_t    fileoff;        // File offset
    uint64_t    filesize;       // File size
    vm_prot_t   maxprot;        // Maximum protection
    vm_prot_t   initprot;       // Initial protection
    uint32_t    nsects;         // Number of sections
    uint32_t    flags;          // Flags
};
```

### 2.5 Segments and Sections

Segments contain one or more sections with related data.

#### Standard Segments

**__PAGEZERO**
- Size: 4GB (on 64-bit)
- No permissions (catches NULL pointer dereferences)
- Virtual memory mapping only

**__TEXT** (Read + Execute)
- **__text**: Executable code
- **__stubs**: Stub functions for dynamic linking
- **__stub_helper**: Helper code for stubs
- **__cstring**: C strings (NULL-terminated)
- **__const**: Constant data
- **__unwind_info**: Exception unwinding information

**__DATA** (Read + Write)
- **__data**: Initialized mutable data
- **__bss**: Uninitialized data
- **__common**: Common symbols
- **__const**: Constant data (that needs relocation)
- **__mod_init_func**: Module initialization functions
- **__mod_term_func**: Module termination functions
- **__objc_***: Objective-C runtime data

**__LINKEDIT** (Read only)
- Symbol table
- String table
- Code signature
- Other linking information

```bash
# View segments
otool -l /bin/ls | grep -A 5 "segname __TEXT"

# View sections in a segment
otool -l /bin/ls | grep -A 8 "sectname"

# Dump specific section
otool -s __TEXT __text /bin/ls | head
```

### 2.6 Dynamic Linking

#### Viewing Linked Libraries

```bash
# List all linked libraries
otool -L /bin/ls

# Example output:
# /bin/ls:
#     /usr/lib/libutil.dylib (compatibility version 1.0.0, current version 1.0.0)
#     /usr/lib/libncurses.5.4.dylib (compatibility version 5.4.0, current version 5.4.0)
#     /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1311.0.0)

# Find what depends on a library
otool -L /System/Library/Frameworks/*.framework/Versions/Current/* | grep libSystem
```

#### Library Search Paths

Dynamic linker searches for libraries in specific order:
1. @executable_path (relative to executable)
2. @loader_path (relative to loader)
3. @rpath (runtime search path)
4. Absolute paths
5. /usr/lib
6. /System/Library/Frameworks

```bash
# Check rpath settings
otool -l binary | grep -A 3 LC_RPATH

# View dyld environment variables
man dyld
```

#### Framework Structure

```
Framework.framework/
├── Framework -> Versions/Current/Framework
├── Headers -> Versions/Current/Headers
├── Resources -> Versions/Current/Resources
└── Versions/
    ├── A/
    │   ├── Framework           # Main library
    │   ├── Headers/            # Public headers
    │   ├── PrivateHeaders/     # Private headers (optional)
    │   └── Resources/          # Resources
    └── Current -> A
```

### 2.7 Symbol Table

The symbol table maps addresses to function/variable names.

```bash
# View symbol table
nm /bin/ls

# Common symbol types:
# T - Text section (code)
# D - Data section
# B - BSS section
# U - Undefined (imported)
# t/d/b - Local symbols

# View only function names
nm /bin/ls | grep " T "

# Dump symbol table with otool
otool -I -v /bin/ls
```

### 2.8 Code Signing

All executables on modern macOS must be code signed.

```bash
# View code signature
codesign -dv /bin/ls

# View entitlements
codesign -d --entitlements - /bin/ls

# Verify signature
codesign -v /bin/ls

# View certificate chain
codesign -dvvv /bin/ls

# Check if binary is signed
codesign -d /path/to/binary 2>&1 | grep "not signed"
```

### 2.9 Tools for Mach-O Analysis

#### otool

Built-in tool for examining Mach-O files.

```bash
otool -h binary        # Header
otool -l binary        # Load commands
otool -L binary        # Linked libraries
otool -t binary        # Text section disassembly
otool -d binary        # Data section
otool -o binary        # Objective-C segment
otool -I binary        # Indirect symbol table
otool -v binary        # Verbose mode
```

#### nm

List symbols in object files.

```bash
nm binary              # All symbols
nm -g binary           # External symbols only
nm -u binary           # Undefined symbols
nm -p binary           # Don't sort
nm -m binary           # Display more info
```

#### jtool2

Third-party tool with advanced features.

```bash
# Install jtool2
curl -O http://www.newosxbook.com/tools/jtool2.tgz
tar xzvf jtool2.tgz

# Usage
jtool2 -l binary       # List segments
jtool2 -L binary       # List libraries
jtool2 -h binary       # Show header
```

#### MachOView

GUI application for viewing Mach-O structure (no longer maintained, but useful).

#### Hopper Disassembler

Commercial tool for disassembly and decompilation.

#### IDA Pro / Ghidra

Professional reverse engineering tools with Mach-O support.

## Practical Exercises

### Exercise 1: Basic Mach-O Inspection

1. Choose a system binary (e.g., `/bin/ls`)
2. View its header: `otool -h /bin/ls`
3. Identify:
   - Magic number
   - CPU type
   - File type
   - Number of load commands

### Exercise 2: Exploring Load Commands

1. View all load commands: `otool -l /bin/ls`
2. Count the number of LC_LOAD_DYLIB commands
3. Find the LC_MAIN command and note the entry point
4. Find the LC_CODE_SIGNATURE command

### Exercise 3: Segment and Section Analysis

1. List all segments in `/usr/bin/python3`
2. For the __TEXT segment:
   - What is its virtual address?
   - What is its size?
   - What permissions does it have?
3. Find and examine the __TEXT.__text section
4. Dump the first 100 bytes of __TEXT.__cstring

### Exercise 4: Dynamic Library Analysis

1. Choose an application from `/Applications`
2. Find its main executable
3. List all linked libraries
4. Identify system frameworks it uses
5. Check if it uses any private frameworks

### Exercise 5: Symbol Table Exploration

1. Use `nm` to list symbols in `/bin/ls`
2. Count how many functions it exports
3. Find imported (undefined) symbols
4. Compare the output with `otool -I`

### Exercise 6: Universal Binary Handling

1. Find a universal binary: `file /usr/bin/python3`
2. Extract the arm64 version: `lipo /usr/bin/python3 -thin arm64 -output python_arm64`
3. Extract the x86_64 version: `lipo /usr/bin/python3 -thin x86_64 -output python_x86`
4. Compare the file sizes
5. Compare their linked libraries

### Exercise 7: Code Signing Verification

1. Check the code signature of a system binary
2. View its entitlements
3. Verify the signature
4. Try to modify the binary (will break signature)
5. Verify again to see the failure

## Key Takeaways

- Mach-O is the native executable format for macOS
- The format consists of a header, load commands, and data
- Segments contain sections with related data
- Dynamic linking is handled through LC_LOAD_DYLIB commands
- Code signing is mandatory on modern macOS
- Multiple tools are available for Mach-O analysis

## Further Reading

- [Mach-O Programming Topics (Apple)](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/MachOTopics/)
- [Mach-O File Format Reference (Apple)](https://github.com/aidansteele/osx-abi-macho-file-format-reference)
- [OS X ABI Mach-O File Format Reference](http://www.opensource.apple.com/source/xnu/xnu-1456.1.26/EXTERNAL_HEADERS/mach-o/)
- [The Mac Hacker's Handbook (book)](https://www.wiley.com/en-us/The+Mac+Hacker%27s+Handbook-p-9780470395363)
- [*OS Internals series by Jonathan Levin](http://newosxbook.com/)

## Next Module

[Module 3: Introduction to LLDB →](03-lldb-basics.md)
