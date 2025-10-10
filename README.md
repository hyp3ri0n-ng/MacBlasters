# MacBlasters: A Comprehensive Course on macOS Internals and Security Research

Welcome to MacBlasters! This is a hands-on, in-depth course designed to take you from macOS fundamentals to advanced security research and exploitation techniques. Whether you're a security researcher, reverse engineer, or simply curious about macOS internals, this course will provide you with the knowledge and practical skills needed to understand and analyze macOS systems.

## About This Course

This course covers:
- **macOS Fundamentals**: Understanding the operating system, file systems, and basic architecture
- **Mach-O Binary Format**: Deep dive into macOS native executable format
- **LLDB Mastery**: From basic debugging to advanced techniques and scripting
- **macOS Internals**: XNU kernel, system calls, IPC mechanisms, and frameworks
- **Security Architecture**: SIP, Gatekeeper, Code Signing, and other protection mechanisms
- **Security Research Methodology**: How to approach research, find vulnerabilities, and document findings
- **Practical Exploitation**: Real-world techniques and defense strategies

## Course Philosophy

This course takes a hands-on, exploratory approach. We learn by doing, experimenting, and understanding the "why" behind everything. No prior macOS experience is required, but basic knowledge of operating systems and programming will be helpful.

## Course Modules

### Module 1: macOS Fundamentals
[📖 Go to Module 1](modules/01-macos-fundamentals.md)

Learn the basics of macOS, terminal usage, file system structure, and essential tools. This module provides the foundation for everything else in the course.

**Topics covered:**
- Terminal and shell basics (zsh, bash)
- File system hierarchy and structure
- Essential command-line tools
- macOS architecture overview
- Application bundles and package structure

### Module 2: Mach-O Binary Format
[📖 Go to Module 2](modules/02-mach-o-binaries.md)

Deep dive into the Mach-O executable format used by macOS. Understand how binaries are structured, loaded, and executed.

**Topics covered:**
- Mach-O file structure and headers
- Load commands and segments
- Dynamic linking and libraries
- Analyzing binaries with otool and jtool
- Universal binaries and fat files

### Module 3: Introduction to LLDB
[📖 Go to Module 3](modules/03-lldb-basics.md)

Master the LLDB debugger, an essential tool for reverse engineering and security research on macOS.

**Topics covered:**
- LLDB fundamentals and commands
- Setting breakpoints and watchpoints
- Examining memory and registers
- Step-by-step execution
- Process attachment and control

### Module 4: Advanced LLDB Techniques
[📖 Go to Module 4](modules/04-lldb-advanced.md)

Take your LLDB skills to the next level with advanced debugging techniques and automation.

**Topics covered:**
- LLDB Python scripting
- Custom commands and aliases
- Memory analysis and pattern searching
- Symbolic debugging
- Debugging stripped binaries
- Remote debugging

### Module 5: macOS Security Architecture
[📖 Go to Module 5](modules/05-security-architecture.md)

Understand the security mechanisms that protect macOS systems and how they work.

**Topics covered:**
- System Integrity Protection (SIP)
- Gatekeeper and notarization
- Code signing and entitlements
- Sandbox and App Sandbox
- TCC (Transparency, Consent, and Control)
- Secure Boot and FileVault

### Module 6: macOS Internals
[📖 Go to Module 6](modules/06-macos-internals.md)

Explore the inner workings of macOS, from the XNU kernel to user-space frameworks.

**Topics covered:**
- XNU kernel architecture
- Mach messaging and IPC
- BSD layer and POSIX compatibility
- IOKit and kernel extensions
- System calls and syscall hooking
- Launch daemons and agents
- macOS frameworks and private APIs

### Module 7: Security Research Methodology
[📖 Go to Module 7](modules/07-research-methodology.md)

Learn systematic approaches to security research, vulnerability discovery, and documentation.

**Topics covered:**
- Research methodology and planning
- Attack surface analysis
- Fuzzing and testing strategies
- Vulnerability classification
- Responsible disclosure
- Writing proof-of-concepts
- Documentation and reporting

### Module 8: Practical Security Analysis
[📖 Go to Module 8](modules/08-practical-security.md)

Apply your knowledge to real-world scenarios with practical exercises and case studies.

**Topics covered:**
- Binary analysis workflow
- Identifying vulnerabilities
- Exploit development basics
- Privilege escalation techniques
- Defense strategies and hardening
- Case studies and CTF challenges

## Prerequisites

- Basic understanding of operating systems concepts
- Familiarity with command-line interfaces
- Programming knowledge (C/C++ helpful but not required)
- A Mac computer with macOS (or access to a macOS VM)

## How to Use This Course

1. **Follow the modules in order** - Each module builds on previous knowledge
2. **Practice hands-on** - Every concept includes practical exercises
3. **Experiment freely** - Use VMs or test systems to explore safely
4. **Take notes** - Document your findings and learning
5. **Contribute** - Share your discoveries and improvements via pull requests

## Additional Resources

- [Resources and References](resources/references.md) - Curated list of books, papers, and tools
- [Labs and Exercises](labs/README.md) - Hands-on exercises for each module
- [Glossary](resources/glossary.md) - Common terms and definitions

## Contributing

This course is a community effort. If you find errors, have suggestions, or want to contribute new content, please open an issue or submit a pull request!

---

## Legacy Content (Original Explorations)

The sections below contain the original exploratory notes that inspired this course. They're kept here for historical context.

<details>
<summary>Alex's Corner - Original Notes on Mach-O Binaries</summary>

#### Terminal basics

The terminal by default is zsh. This is an alternative to Bash. There's really no reason to use zsh unless you like it, I'd stick with bash which you can configure to be the default shell. I use iTerm2 instead of Terminal application, it's just cleaner looking by default. I like to set infinite scrollback in case i lose stuff and, because i'm old, set the font larger. There are other terminal applications (like Terminator) that will work on OS X. So if you don't like the shell experience, there are lots of alternatives.

#### Native binary files

Mac OS X can run a number of types of binary files. The preferred and main file type we'll look at is the Mach-O (pronounced mock-oh). You can see a variety of applications in `/System/Applications`:

```
alejandrocaceres@Alejandros-MacBook-Pro /S/Applications> ls
App Store.app          FaceTime.app           Messages.app           Preview.app            TextEdit.app
Automator.app          FindMy.app             Mission Control.app    QuickTime Player.app   Time Machine.app
Books.app              Font Book.app          Music.app              Reminders.app          Utilities
Calculator.app         Home.app               News.app               Siri.app               VoiceMemos.app
Calendar.app           Image Capture.app      Notes.app              Stickies.app
Chess.app              Launchpad.app          Photo Booth.app        Stocks.app
Contacts.app           Mail.app               Photos.app             System Preferences.app
Dictionary.app         Maps.app               Podcasts.app           TV.app
```

note that these are actually directories (pedantic note: they are not FOLDERS, folders are MS Windows, in \*nix operating systems they are directories) and not single files (like a .exe). This is because these are app bundles, which are treated specially by the OS and has a rigid directory structure. So let's find the file that I'd actually run. Let's pick the Music.app for no particular reason:


```
alejandrocaceres@Alejandros-MacBook-Pro /Library [1]> cd /System/Applications/Music.app/
alejandrocaceres@Alejandros-MacBook-Pro /Library [1]> cd Contents
alejandrocaceres@Alejandros-MacBook-Pro /S/A/M/Contents> ls
Frameworks     MacOS          PlugIns        XPCServices    version.plist
Info.plist     PkgInfo        Resources      _CodeSignature
alejandrocaceres@Alejandros-MacBook-Pro /S/A/M/Contents> cd MacOS/
alejandrocaceres@Alejandros-MacBook-Pro /S/A/M/C/MacOS> ls
Music
alejandrocaceres@Alejandros-MacBook-Pro /S/A/M/C/MacOS> file Music
Music: Mach-O universal binary with 2 architectures: [x86_64:Mach-O 64-bit executable x86_64] [arm64e]
Music (for architecture x86_64):	Mach-O 64-bit executable x86_64
Music (for architecture arm64e):	Mach-O 64-bit executable arm64e
```

OK great, we're not crazy, the Mach-O format is being used. But what about non-.app stuff? I have many many seemingly Linux commands - are these in ELF (Linux) format or are they in Mach-O format? It sure feels like I'm running an ELF...

```
alejandrocaceres@Alejandros-MacBook-Pro /S/A/M/Contents> which ls
/bin/ls
alejandrocaceres@Alejandros-MacBook-Pro /S/A/M/Contents> file /bin/ls
./bin/ls: Mach-O universal binary with 2 architectures: [x86_64:Mach-O 64-bit executable x86_64] [arm64e:Mach-O 64-bit executable arm64e]
```

...but OS X doesn't care about my feelings. The ls program has been recompiled as a Mach-O format. Great, we're locked in to something, perhaps this will help us attack binaries in the future. In compilation there are a few stages, first is the linking and importing stage where all of your imports to use libraries must be listed. The linking phase is critical in that it is what allows you to reuse code, you just need to tell it where that code is. Typically, this code is compiled. These are called `dll` (dynamic link library) in Windows and `.so` (shared object) in Linux. With that in mind let's see what other stuff we can discover about this executable.

Before we do anything, there is a critical step to bug hunting and offensive security that is often skipped. Google the sh** out of your topic. So let's see what useful stuff Google has to tell us about binaries in OS X. The first thing I really want to know is what does a Mach-O file "look" like. These binaries must follow some sort of standard such that the OS knows how to run it, knows where to find metadata that it needs, etc. So check out this diagram:

[Broken down Mach-O file](https://www.symbolcrash.com/wp-content/uploads/2019/02/File_structure.png)

And let's use Google to go through these sections:

- Starting with the Mach Header. It's purpose is to describe what the file contains, and how the Kernel and Dynamic Linker should handle it. The first 4 bytes are, like with any file, it's "Magic Number". A Magic Number is used to identify a file format. In the case of Mach-O's there are three Magic Numbers that one may come across. 0xfeedface for 32-bit, 0xfeedfacf for 64-bit and 0xcafebabe for Mach Universal Binaries / Object files. This is a breakdown of the C struct that makes up that section: ```    struct mach_header {
        uint32_t            magic;          // mach magic number
        cpu_type_t          cputype;        // cpu specifier
        cpu_subtype_t       cpusubtype;     // cpu subtype specifier
        uint32_t            filetype;       // type of mach-o e.g. exec, dylib ...
        uint32_t            ncmds;          // number of load commands
        uint32_t            sizeofcmds;     // size of load command region
        uint32_t            flags;          // flags
        uint32_t            reserved;       // *64-bit only* reserved
    };```
- Load Commands are placed directly after the Mach-O header in the file. They specify the logical structure of the file and the layout of the file in virtual memory.
- If you are unfamiliar with how object files work, you have a number of these segments. The __TEXT segment contains the instructions that will be executed by the CPU, and the __DATA segment contains both static local variables and global variables. These are both standard, however you may find additional segments such as __PAGEZERO and __LINKEDIT, and in XNU Kernelcaches, you'll get even more funky segment names like __PRELINK_INFO and __LAST.

Segments are further divided into sections, so for example you'll find __cstring in the __TEXT segment, formatted as __TEXT.__cstring, as a common one.

Note that there is a segment specific to allowing more segments.
- Finally the LINKEDIT section of a binary contains linker information that is used by the binary

Now let's start dissecting!

First, I want to see what it's linked to! Introducing `otool`, if you're familiar with `objdump` on Linux, this is essentially the same thing except for Mac. So let's see how we find the linked libraries. Turns out the -L switch will give us these libraries:


```
alejandrocaceres@Alejandros-MacBook-Pro /S/A/M/C/MacOS> otool -L Music | head -n 30
Music:
	/System/Library/PrivateFrameworks/AVFoundationCF.framework/Versions/A/AVFoundationCF (compatibility version 1.0.0, current version 2.0.0)
	/System/Library/Frameworks/iTunesLibrary.framework/Versions/A/iTunesLibrary (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/PrivateFrameworks/AMPDevices.framework/Versions/A/AMPDevices (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/PrivateFrameworks/AMPLibrary.framework/Versions/A/AMPLibrary (compatibility version 0.0.0, current version 0.0.0)
	/System/Library/Frameworks/CoreLocation.framework/Versions/A/CoreLocation (compatibility version 1.0.0, current version 2420.19.23)
	/System/Library/Frameworks/Accelerate.framework/Versions/A/Accelerate (compatibility version 1.0.0, current version 4.0.0)
	/System/Library/PrivateFrameworks/AppleMediaServicesUI.framework/Versions/A/AppleMediaServicesUI (compatibility version 1.0.0, current version 1.0.0, weak)
	/System/Library/PrivateFrameworks/AppleSRP.framework/Versions/A/AppleSRP (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/PrivateFrameworks/AskPermission.framework/Versions/A/AskPermission (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/PrivateFrameworks/MediaControlSender.framework/Versions/A/MediaControlSender (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/PrivateFrameworks/FamilyCircle.framework/Versions/A/FamilyCircle (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/PrivateFrameworks/AppleMediaServices.framework/Versions/A/AppleMediaServices (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/Frameworks/Accounts.framework/Versions/A/Accounts (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/Frameworks/AVKit.framework/Versions/A/AVKit (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/Frameworks/AVFoundation.framework/Versions/A/AVFoundation (compatibility version 1.0.0, current version 2.0.0)
	/System/Library/PrivateFrameworks/AuthKitUI.framework/Versions/A/AuthKitUI (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/PrivateFrameworks/AuthKit.framework/Versions/A/AuthKit (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/Frameworks/AudioUnit.framework/Versions/A/AudioUnit (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/Frameworks/AudioToolbox.framework/Versions/A/AudioToolbox (compatibility version 1.0.0, current version 1000.0.0)
	/System/Library/PrivateFrameworks/ApplePushService.framework/Versions/A/ApplePushService (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/PrivateFrameworks/AppleIDSSOAuthentication.framework/Versions/A/AppleIDSSOAuthentication (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/PrivateFrameworks/AOSAccounts.framework/Versions/A/AOSAccounts (compatibility version 1.0.0, current version 1.9.95)
	/System/Library/Frameworks/AGL.framework/Versions/A/AGL (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/PrivateFrameworks/BiometricKit.framework/Versions/A/BiometricKit (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/Frameworks/CryptoTokenKit.framework/Versions/A/CryptoTokenKit (compatibility version 0.0.0, current version 487.141.1)
	/System/Library/PrivateFrameworks/CrashReporterSupport.framework/Versions/A/CrashReporterSupport (compatibility version 1.0.0, current version 15053.1.0)
	/System/Library/PrivateFrameworks/CoreUtils.framework/Versions/A/CoreUtils (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/PrivateFrameworks/CoreUI.framework/Versions/A/CoreUI (compatibility version 1.0.0, current version 692.1.0)
	/System/Library/PrivateFrameworks/CoreRecognition.framework/Versions/A/CoreRecognition (compatibility version 1.0.0, current version 157.0.0)     
```

</details>

<details>
<summary>Rich's Corner - Original Notes on Meterpreter</summary>

Meterpreter:

http://www.ethicalpentest.com/2018/03/meterpreter-metasploit-tutorial-part1.html

</details>

<details>
<summary>Emily's Corner - Original Notes on System Profiling</summary>

#### Services that may be helpful

I highly recommend looking into the system report, whether through the GUI [ > About this Mac > System Report] or in the terminal using ```system_profiler```. This can be accessed with minimal privilages, especially if you have physical access to an unlocked device.

I also recommend using the following commands 
```ps aux
ps -ax
top
```
to use Activity Monitor in terminal

</details>

<details>
<summary>Nicholas' Corner</summary>

(To be added)

</details>
