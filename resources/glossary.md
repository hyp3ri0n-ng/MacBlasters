# Glossary

A comprehensive glossary of terms used throughout the MacBlasters course.

## A

**ASLR (Address Space Layout Randomization)**
- Security technique that randomizes memory addresses to make exploitation harder
- Enabled by default on macOS

**App Bundle**
- Directory structure containing an application and its resources
- Treated as a single entity by macOS
- Extension: .app

**App Sandbox**
- Restriction mechanism limiting app access to system resources
- Enforced through entitlements and profiles

**APFS (Apple File System)**
- Modern file system introduced in macOS 10.13
- Features: Encryption, snapshots, clones, space sharing

## B

**BSD (Berkeley Software Distribution)**
- Unix variant that forms part of macOS
- Provides POSIX compatibility layer in XNU

**Breakpoint**
- Debugging feature that pauses execution at specific location
- Types: Address, function name, line number

## C

**Code Signing**
- Cryptographic verification of app integrity and origin
- Required for all executables on modern macOS

**Container**
- Isolated directory for sandboxed app data
- Located in ~/Library/Containers/

**CVE (Common Vulnerabilities and Exposures)**
- Standardized identifier for security vulnerabilities
- Format: CVE-YYYY-NNNNN

**CVSS (Common Vulnerability Scoring System)**
- Standard for assessing vulnerability severity
- Score range: 0.0-10.0

## D

**Daemon**
- Background process running system-wide
- Typically runs as root
- Managed by launchd

**Darwin**
- Open-source Unix foundation of macOS
- Includes XNU kernel, BSD layer, and core utilities

**DTrace**
- Dynamic tracing framework for system observability
- Allows real-time system analysis without modification

**DYLD (Dynamic Linker)**
- Loads and links shared libraries at runtime
- Part of Darwin

**Dynamic Library**
- Shared code library loaded at runtime
- Extension: .dylib

## E

**Entitlements**
- Key-value pairs granting specific capabilities to apps
- Embedded in code signature
- Required for privileged operations

**Exploit**
- Code or technique that takes advantage of a vulnerability
- Purpose: Unauthorized access or control

## F

**Fat Binary**
- See Universal Binary

**FileVault**
- Full-disk encryption feature
- Uses XTS-AES-128 encryption

**Framework**
- Bundle containing shared code and resources
- Extension: .framework
- Located in /System/Library/Frameworks/ or /System/Library/PrivateFrameworks/

**Fuzzing**
- Automated testing technique using random/malformed inputs
- Goal: Find crashes and vulnerabilities

## G

**Gatekeeper**
- Security feature checking app authenticity
- Verifies code signatures and notarization

## H

**Hardened Runtime**
- Additional security restrictions on executables
- Introduced in macOS 10.14
- Requires specific entitlements for certain operations

**Helper Tool**
- Privileged executable performing operations on behalf of app
- Typically runs with elevated privileges

## I

**Info.plist**
- Property list file containing app metadata
- Located in app bundle's Contents directory

**IOKit**
- Object-oriented framework for device drivers
- Written in C++
- Part of XNU kernel

**IPC (Inter-Process Communication)**
- Mechanisms for processes to communicate
- Methods: Mach ports, XPC, pipes, sockets

## K

**Kernel Extension (KEXT)**
- Code running in kernel space (deprecated)
- Extension: .kext
- Being replaced by System Extensions

## L

**Launch Agent**
- Per-user background process
- Runs when user logs in
- Managed by launchd

**Launch Daemon**
- System-wide background process
- Runs at boot
- Managed by launchd

**launchd**
- First user-space process (PID 1)
- Manages all other processes on macOS

**LLDB (Low-Level Debugger)**
- Default debugger on macOS
- Part of LLVM project
- Replaced GDB

**LLVM (Low Level Virtual Machine)**
- Compiler infrastructure
- Provides LLDB, Clang, and other tools

**LPE (Local Privilege Escalation)**
- Vulnerability allowing attacker to gain elevated privileges
- Critical security issue

## M

**Mach**
- Microkernel forming core of XNU
- Provides: IPC, virtual memory, scheduling

**Mach-O (Mach Object)**
- Native executable format for macOS/iOS
- Contains: Header, load commands, data

**Mach Port**
- IPC mechanism in Mach kernel
- Used for inter-process communication

**MIG (Mach Interface Generator)**
- Tool generating client/server stubs for Mach IPC

## N

**Notarization**
- Apple's malware scanning service
- Required for distributed apps
- Doesn't require Mac App Store submission

## P

**PAC (Pointer Authentication Codes)**
- Hardware security feature on Apple Silicon
- Prevents pointer corruption attacks

**POSIX (Portable Operating System Interface)**
- Standard for Unix-like operating systems
- macOS is POSIX-compliant via BSD layer

**Private Framework**
- Framework with undocumented/unsupported APIs
- Located in /System/Library/PrivateFrameworks/

**PoC (Proof of Concept)**
- Code demonstrating vulnerability
- Used for verification and disclosure

## R

**Reverse Engineering**
- Process of analyzing software to understand its design
- Techniques: Disassembly, debugging, analysis

**Root**
- Superuser account with full system access
- UID 0

## S

**Sandbox**
- Security mechanism restricting app capabilities
- Can be system-enforced or self-imposed

**Secure Boot**
- Verification of boot chain integrity
- Ensures only trusted software runs

**Secure Enclave**
- Dedicated security coprocessor
- Handles cryptographic operations and Touch ID

**SIP (System Integrity Protection)**
- Also known as "rootless"
- Restricts root user access to system files
- Introduced in OS X El Capitan (10.11)

**System Extension**
- Modern replacement for kernel extensions
- Runs in user space
- Types: DriverKit, Network Extension, Endpoint Security

## T

**T2 Chip**
- Security coprocessor in Intel-based Macs
- Handles: Secure boot, encryption, Touch ID

**TCC (Transparency, Consent, and Control)**
- Privacy permission system
- Manages access to: Camera, microphone, contacts, etc.

**TOCTOU (Time-of-Check-Time-of-Use)**
- Race condition vulnerability
- Occurs when check and use are not atomic

## U

**UAF (Use-After-Free)**
- Memory corruption vulnerability
- Occurs when memory is used after being freed

**Universal Binary**
- Executable containing code for multiple architectures
- Also called "Fat Binary"
- Example: x86_64 and arm64 in one file

## V

**Vulnerability**
- Weakness allowing security policy violation
- Categories: Memory corruption, logic errors, information disclosure

## X

**XNU (X is Not Unix)**
- Hybrid kernel used in macOS
- Combines: Mach microkernel, BSD subsystem, IOKit

**XPC (Inter-Process Communication)**
- Modern IPC mechanism on macOS
- Built on Mach messages
- Provides process isolation and security

**XPC Service**
- Helper service using XPC for communication
- Provides process isolation
- Located in app bundle

## Z

**Zero-Day**
- Vulnerability unknown to vendor
- No patch available
- Actively exploited: "in the wild"

---

## Acronyms Quick Reference

- **ASLR**: Address Space Layout Randomization
- **APFS**: Apple File System
- **BSD**: Berkeley Software Distribution
- **CVE**: Common Vulnerabilities and Exposures
- **CVSS**: Common Vulnerability Scoring System
- **IPC**: Inter-Process Communication
- **KEXT**: Kernel Extension
- **LLDB**: Low-Level Debugger
- **LLVM**: Low Level Virtual Machine
- **LPE**: Local Privilege Escalation
- **PAC**: Pointer Authentication Codes
- **PoC**: Proof of Concept
- **POSIX**: Portable Operating System Interface
- **SIP**: System Integrity Protection
- **TCC**: Transparency, Consent, and Control
- **TOCTOU**: Time-of-Check-Time-of-Use
- **UAF**: Use-After-Free
- **XNU**: X is Not Unix
- **XPC**: Inter-Process Communication (Apple's implementation)

---

[← Back to Course Home](../README.md)
