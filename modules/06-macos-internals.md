# Module 6: macOS Internals

## Overview

This module explores the internal architecture of macOS, including the XNU kernel, system calls, inter-process communication, frameworks, and the various subsystems that make macOS work.

## Learning Objectives

By the end of this module, you will be able to:
- Understand the XNU kernel architecture
- Work with Mach messaging and IPC
- Understand the BSD layer and POSIX compatibility
- Explore IOKit and device drivers
- Analyze system calls and create syscall hooks
- Understand launch daemons and agents
- Work with macOS frameworks and private APIs

## Topics

### 6.1 XNU Kernel Architecture

XNU = X is Not Unix (but it kind of is)

#### Hybrid Kernel Design

```
┌─────────────────────────────────────────┐
│          User Space                     │
├─────────────────────────────────────────┤
│        System Call Interface            │
├─────────────────────────────────────────┤
│           BSD Layer                     │
│   (POSIX, VFS, Networking, etc.)        │
├──────────────┬──────────────────────────┤
│  Mach Kernel │      IOKit              │
│  (IPC, VM,   │   (Device Drivers)      │
│   Scheduling)│                          │
└──────────────┴──────────────────────────┘
```

**Key components:**
- **Mach microkernel**: Core services (IPC, VM, threads)
- **BSD subsystem**: Unix functionality (file systems, networking, process model)
- **IOKit**: Object-oriented driver framework
- **libkern**: C++ runtime for kernel

#### Kernel Source

```bash
# XNU source available at:
# https://opensource.apple.com/source/xnu/

# Download and explore
git clone https://github.com/apple/darwin-xnu.git

# Key directories:
# - osfmk/     - Mach kernel
# - bsd/       - BSD subsystem
# - iokit/     - IOKit framework
# - libkern/   - Kernel library
```

### 6.2 Mach Messaging and IPC

#### Mach Ports

Mach uses ports for inter-process communication.

**Port types:**
- **Send right**: Can send messages
- **Receive right**: Can receive messages (exclusive)
- **Send-once right**: One message only
- **Port set**: Group of ports

```c
// Getting task port
#include <mach/mach.h>

mach_port_t task;
kern_return_t kr = task_for_pid(mach_task_self(), pid, &task);

if (kr == KERN_SUCCESS) {
    printf("Got task port: %d\n", task);
}
```

#### Mach Messages

```c
typedef struct {
    mach_msg_header_t header;
    mach_msg_body_t body;
    // ... message data
} mach_msg_t;

// Send message
mach_msg_send(msg);

// Receive message
mach_msg_receive(msg);
```

#### MIG (Mach Interface Generator)

MIG generates client/server stubs from interface definitions.

```c
// Example MIG definition
subsystem my_service 1000;

#include <mach/std_types.defs>

routine my_function(
    server_port : mach_port_t;
    arg1 : int;
    arg2 : int;
    out result : int);
```

### 6.3 BSD Layer

#### System Calls

BSD layer provides POSIX syscalls.

```bash
# List syscalls
man 2 intro

# Common syscalls:
# - open, close, read, write
# - fork, exec, wait
# - socket, connect, send, recv
# - mmap, munmap
```

#### DTrace

Dynamic tracing for system observability.

```bash
# Trace system calls
sudo dtruss -p <PID>

# Trace specific syscall
sudo dtrace -n 'syscall::open:entry { printf("%s", copyinstr(arg0)); }'

# Trace all syscalls by process
sudo dtrace -n 'syscall:::entry /execname == "Safari"/ { @[probefunc] = count(); }'

# Requires: csrutil enable --without dtrace
```

#### File Systems

**Supported file systems:**
- **APFS**: Apple File System (default since macOS 10.13)
- **HFS+**: Hierarchical File System Plus (legacy)
- **FAT, ExFAT**: For compatibility
- **NTFS**: Read-only by default

```bash
# View file system info
diskutil info /

# APFS specific
diskutil apfs list

# Mount information
mount
```

### 6.4 IOKit

#### Overview

IOKit is an object-oriented framework for device drivers, written in C++.

**Key classes:**
- `IOService`: Base class for drivers
- `IOUserClient`: User-space communication
- `IOMemoryDescriptor`: Memory management
- `IOWorkLoop`: Event processing

#### Exploring IOKit

```bash
# View IORegistry
ioreg -l

# Filter by class
ioreg -c IOPCIDevice

# Filter by name
ioreg -n AppleT2Controller

# Show properties
ioreg -c AppleIntelCPUPowerManagement -r
```

#### IOKit User Clients

User-space apps communicate with kernel drivers via IOUserClient.

```objc
#import <IOKit/IOKitLib.h>

// Find service
io_service_t service = IOServiceGetMatchingService(
    kIOMasterPortDefault,
    IOServiceMatching("MyDriver")
);

// Open connection
io_connect_t connection;
IOServiceOpen(service, mach_task_self(), 0, &connection);

// Call method
uint64_t output;
uint32_t outputCount = 1;
IOConnectCallScalarMethod(
    connection,
    0,              // selector
    NULL, 0,        // input
    &output, &outputCount  // output
);

// Close
IOServiceClose(connection);
IOObjectRelease(service);
```

### 6.5 System Calls and Hooking

#### Syscall Table

```c
// Syscall numbers defined in:
// /usr/include/sys/syscall.h

#define SYS_fork    2
#define SYS_open    5
#define SYS_close   6
#define SYS_read    3
#define SYS_write   4
```

#### Interposing Functions

Use `DYLD_INSERT_LIBRARIES` to interpose functions:

```c
// interpose.c
#include <stdio.h>
#include <unistd.h>

#define DYLD_INTERPOSE(_replacement,_replacee) \
   __attribute__((used)) static struct{ const void* replacement; const void* replacee; } _interpose_##_replacee \
   __attribute__ ((section ("__DATA,__interpose"))) = { (const void*)(unsigned long)&_replacement, (const void*)(unsigned long)&_replacee };

ssize_t my_write(int fd, const void *buf, size_t count) {
    printf("[HOOK] write(%d, %p, %zu)\n", fd, buf, count);
    return write(fd, buf, count);
}

DYLD_INTERPOSE(my_write, write);
```

```bash
# Compile
clang -dynamiclib interpose.c -o libinterpose.dylib

# Use
DYLD_INSERT_LIBRARIES=./libinterpose.dylib DYLD_FORCE_FLAT_NAMESPACE=1 /bin/ls
```

#### Fishhook

Dynamic function hooking library by Facebook.

```c
#include "fishhook.h"

static ssize_t (*orig_write)(int, const void *, size_t);

ssize_t my_write(int fd, const void *buf, size_t count) {
    printf("[HOOK] write called\n");
    return orig_write(fd, buf, count);
}

int main() {
    rebind_symbols((struct rebinding[1]){{"write", my_write, (void *)&orig_write}}, 1);
    write(1, "Hello\n", 6);
    return 0;
}
```

### 6.6 Launch Daemons and Agents

#### Launchd

`launchd` is the first user-space process (PID 1) and manages all other processes.

**Types:**
- **Daemon**: System-wide, runs as root
- **Agent**: Per-user, runs as user

**Locations:**
```bash
# System daemons
/System/Library/LaunchDaemons/
/Library/LaunchDaemons/

# User agents
/System/Library/LaunchAgents/
/Library/LaunchAgents/
~/Library/LaunchAgents/
```

#### Plist Format

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.example.mydaemon</string>
    
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/mydaemon</string>
        <string>--config</string>
        <string>/etc/mydaemon.conf</string>
    </array>
    
    <key>RunAtLoad</key>
    <true/>
    
    <key>KeepAlive</key>
    <true/>
    
    <key>StandardErrorPath</key>
    <string>/var/log/mydaemon.err</string>
    
    <key>StandardOutPath</key>
    <string>/var/log/mydaemon.out</string>
</dict>
</plist>
```

#### Managing Launch Items

```bash
# Load daemon
sudo launchctl load /Library/LaunchDaemons/com.example.mydaemon.plist

# Unload daemon
sudo launchctl unload /Library/LaunchDaemons/com.example.mydaemon.plist

# List all loaded items
launchctl list

# Check status
launchctl list | grep com.example

# Start/stop service
sudo launchctl start com.example.mydaemon
sudo launchctl stop com.example.mydaemon

# Enable/disable
sudo launchctl enable system/com.example.mydaemon
sudo launchctl disable system/com.example.mydaemon
```

### 6.7 XPC Services

#### What is XPC?

XPC (Inter-Process Communication) is Apple's modern IPC mechanism, built on top of Mach messages.

**Benefits:**
- Security: Separate process boundaries
- Stability: Crashes don't affect main app
- Privilege separation

#### XPC Service Structure

```
MyApp.app/
└── Contents/
    ├── MacOS/
    │   └── MyApp
    └── XPCServices/
        └── com.example.MyXPCService.xpc/
            └── Contents/
                ├── Info.plist
                └── MacOS/
                    └── com.example.MyXPCService
```

#### Using XPC

```objc
// Client (main app)
#import <Foundation/Foundation.h>

NSXPCConnection *connection = [[NSXPCConnection alloc] initWithServiceName:@"com.example.MyXPCService"];
connection.remoteObjectInterface = [NSXPCInterface interfaceWithProtocol:@protocol(MyServiceProtocol)];
[connection resume];

id<MyServiceProtocol> service = [connection remoteObjectProxy];
[service performTaskWithCompletion:^(NSString *result) {
    NSLog(@"Result: %@", result);
}];
```

### 6.8 Frameworks

#### System Frameworks

Located in `/System/Library/Frameworks/`:

```bash
# List frameworks
ls /System/Library/Frameworks/

# Common frameworks:
# - Foundation.framework
# - AppKit.framework (macOS UI)
# - Core Foundation.framework
# - Security.framework
# - IOKit.framework
# - CoreGraphics.framework
# - CoreAudio.framework
```

#### Private Frameworks

Located in `/System/Library/PrivateFrameworks/`:

```bash
# List private frameworks
ls /System/Library/PrivateFrameworks/

# These are undocumented but can be reverse engineered
```

#### Using Private APIs

```objc
// Header dumping with class-dump
class-dump /System/Library/PrivateFrameworks/SomeFramework.framework

// Link private framework
#import <SomePrivateFramework/SomePrivateFramework.h>

// Or use dlopen
void *handle = dlopen("/System/Library/PrivateFrameworks/SomeFramework.framework/SomeFramework", RTLD_NOW);
if (handle) {
    // Use functions from framework
}
```

### 6.9 Kernel Extensions (Deprecated)

#### Overview

Kernel extensions (.kext) run in kernel space. **Deprecated** in favor of system extensions.

```bash
# List loaded kexts
kextstat

# Load kext (requires SIP disabled)
sudo kextload /path/to/MyKext.kext

# Unload kext
sudo kextunload /path/to/MyKext.kext

# View kext info
kextutil -print-diagnostics /path/to/MyKext.kext
```

#### System Extensions (Modern Approach)

System extensions run in user space:

- **DriverKit**: User-space drivers
- **Network Extension**: VPN, content filtering
- **Endpoint Security**: Security monitoring

## Practical Exercises

### Exercise 1: Exploring IORegistry

1. Use `ioreg` to explore your system's device tree
2. Find your CPU information
3. Find your GPU information
4. Identify all USB devices
5. Export the registry to a file

### Exercise 2: Tracing System Calls

1. Use DTrace to trace syscalls for a process
2. Identify the most frequently called syscalls
3. Trace file I/O operations
4. Monitor network operations
5. Create a custom DTrace script

### Exercise 3: Function Interposition

1. Create an interpose library
2. Hook `malloc` and `free`
3. Track allocations
4. Detect memory leaks
5. Test with various programs

### Exercise 4: Launch Daemon Creation

1. Create a simple daemon program
2. Write a launch daemon plist
3. Install and load the daemon
4. Monitor its logs
5. Unload and remove the daemon

### Exercise 5: XPC Service Analysis

1. Find an app that uses XPC services
2. Locate its XPC services
3. Use `lsof` to see XPC connections
4. Analyze the XPC protocol with LLDB
5. Document the communication flow

### Exercise 6: Framework Exploration

1. Choose a private framework
2. Use `class-dump` to extract headers
3. Find interesting classes and methods
4. Write a test program to use the framework
5. Document your findings

## Key Takeaways

- XNU is a hybrid kernel combining Mach and BSD
- Mach ports enable IPC and system services
- IOKit provides object-oriented driver framework
- System calls bridge user space and kernel
- Launch daemons/agents manage system services
- XPC provides secure inter-process communication
- Frameworks encapsulate system functionality

## Further Reading

- [Mac OS X Internals by Amit Singh](http://osxbook.com/)
- [*OS Internals by Jonathan Levin](http://newosxbook.com/)
- [XNU Source Code](https://opensource.apple.com/source/xnu/)
- [IOKit Fundamentals](https://developer.apple.com/library/archive/documentation/DeviceDrivers/Conceptual/IOKitFundamentals/)
- [Daemons and Agents](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPSystemStartup/)

## Next Module

[Module 7: Security Research Methodology →](07-research-methodology.md)
