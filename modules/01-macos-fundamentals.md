# Module 1: macOS Fundamentals

## Overview

This module introduces you to the basics of macOS, including terminal usage, file system structure, and essential tools. By the end of this module, you'll be comfortable navigating macOS and understand its fundamental architecture.

## Learning Objectives

By the end of this module, you will be able to:
- Navigate and use the macOS terminal effectively
- Understand the macOS file system hierarchy
- Work with application bundles and understand their structure
- Use essential command-line tools for system exploration
- Understand the basic architecture of macOS (Darwin, XNU kernel, user space)

## Topics

### 1.1 Terminal and Shell Basics

#### Shell Environment

macOS uses **zsh** (Z shell) as the default shell starting from macOS Catalina (10.15). Before that, **bash** was the default shell. Both are powerful POSIX-compatible shells with slightly different features.

**Key differences:**
- **zsh**: Better completion, themes, plugins (Oh My Zsh), improved globbing
- **bash**: More universal, simpler, traditional Unix shell

**Switching shells:**
```bash
# Change to bash
chsh -s /bin/bash

# Change to zsh
chsh -s /bin/zsh
```

#### Terminal Applications

- **Terminal.app**: Built-in terminal application (located in /Applications/Utilities/)
- **iTerm2**: Popular third-party terminal with advanced features
  - Split panes
  - Search functionality
  - Better customization
  - Infinite scrollback
  - Profile management

#### Essential Shell Commands

```bash
# Navigation
pwd                    # Print working directory
cd /path/to/directory  # Change directory
cd ~                   # Go to home directory
cd -                   # Go to previous directory

# File operations
ls -la                 # List files (including hidden)
file filename          # Identify file type
cat filename           # Display file contents
less filename          # Page through file
head -n 20 filename    # Show first 20 lines
tail -n 20 filename    # Show last 20 lines
tail -f filename       # Follow file updates (logs)

# Searching
find /path -name "*.txt"           # Find files by name
grep "pattern" filename            # Search in files
grep -r "pattern" /path            # Recursive search
mdfind "searchterm"                # Spotlight search from terminal

# Process management
ps aux                 # List all processes
top                    # Interactive process viewer
htop                   # Better process viewer (install via Homebrew)
kill <PID>             # Kill process by ID
killall processname    # Kill processes by name

# System information
uname -a               # System information
sw_vers                # macOS version
system_profiler        # Detailed system info
sysctl -a              # Kernel parameters
```

### 1.2 File System Hierarchy

macOS uses a Unix-style file system with some Apple-specific additions.

#### Root Directory Structure

```
/
├── Applications/        # User-installed applications
├── Library/            # System-wide libraries and support files
├── System/             # macOS system files (protected by SIP)
│   ├── Applications/   # Built-in apps (Mail, Safari, etc.)
│   ├── Library/        # System frameworks and resources
│   └── Volumes/        # Mounted volumes appear here
├── Users/              # User home directories
│   └── username/       # Your home directory
│       ├── Desktop/
│       ├── Documents/
│       ├── Downloads/
│       ├── Library/    # User-specific libraries
│       └── ...
├── bin/                # Essential user binaries
├── sbin/               # System binaries (superuser)
├── usr/                # User utilities and applications
│   ├── bin/           # User commands
│   ├── lib/           # Libraries
│   ├── local/         # Locally installed software
│   └── sbin/          # System administration commands
├── etc/                # System configuration files
├── var/                # Variable data (logs, caches, temp files)
├── tmp/                # Temporary files
├── dev/                # Device files
├── private/            # Private system files
└── Volumes/            # Mount points for drives
```

#### Important Locations

- **~/Library**: User-specific application data, preferences, caches
- **/System/Library/Frameworks**: System frameworks
- **/System/Library/PrivateFrameworks**: Private frameworks (undocumented APIs)
- **/System/Library/Extensions**: Kernel extensions (deprecated in favor of system extensions)
- **/Library/LaunchDaemons**: System-wide daemons
- **/Library/LaunchAgents**: System-wide agents
- **~/Library/LaunchAgents**: User-specific agents
- **/var/log**: System logs

#### Hidden Files

Files starting with a dot (`.`) are hidden in Unix systems.

```bash
# Show hidden files in Finder
defaults write com.apple.finder AppleShowAllFiles YES
killall Finder

# Hide them again
defaults write com.apple.finder AppleShowAllFiles NO
killall Finder

# List hidden files in terminal
ls -la
```

### 1.3 Application Bundles

Unlike Windows executables (.exe), macOS applications are **bundles** - directories with a specific structure that the OS treats as single files.

#### App Bundle Structure

```
Application.app/
├── Contents/
│   ├── Info.plist              # Application metadata
│   ├── PkgInfo                 # Package type information
│   ├── MacOS/                  # Executable binaries
│   │   └── Application         # Main executable (Mach-O)
│   ├── Resources/              # Images, sounds, localization
│   │   ├── Icon.icns          # App icon
│   │   ├── *.lproj/           # Localized resources
│   │   └── ...
│   ├── Frameworks/             # Bundled frameworks
│   ├── PlugIns/                # Plugins and extensions
│   ├── XPCServices/            # XPC services
│   └── _CodeSignature/         # Code signature
│       └── CodeResources
```

#### Exploring App Bundles

```bash
# Navigate into an app bundle
cd /Applications/TextEdit.app/Contents

# View the Info.plist
cat Info.plist
plutil -p Info.plist            # Pretty print

# Find the executable
cd MacOS/
file TextEdit                   # Check file type
ls -lh TextEdit                 # Check size
```

#### Info.plist

The `Info.plist` file contains metadata about the application:
- Bundle identifier (e.g., com.apple.TextEdit)
- Version numbers
- Minimum OS version
- Supported file types
- Required capabilities
- Entitlements

```bash
# Read specific values
defaults read /Applications/TextEdit.app/Contents/Info CFBundleIdentifier
defaults read /Applications/TextEdit.app/Contents/Info CFBundleShortVersionString
```

### 1.4 macOS Architecture Overview

macOS is built on **Darwin**, a Unix-based open-source foundation.

```
┌─────────────────────────────────────────────┐
│         Applications & User Space           │
├─────────────────────────────────────────────┤
│      Cocoa / Carbon / Other Frameworks      │
├─────────────────────────────────────────────┤
│         Core Services (BSD Layer)           │
├─────────────────────────────────────────────┤
│            XNU Kernel (Hybrid)              │
│  ┌────────────────┬──────────────────────┐  │
│  │   Mach Kernel  │    BSD Subsystem     │  │
│  └────────────────┴──────────────────────┘  │
├─────────────────────────────────────────────┤
│              IOKit (Drivers)                │
├─────────────────────────────────────────────┤
│                Hardware                     │
└─────────────────────────────────────────────┘
```

#### Key Components:

1. **XNU Kernel**: 
   - Hybrid kernel combining Mach microkernel and BSD subsystem
   - Mach: Memory management, IPC, scheduling
   - BSD: File systems, networking, POSIX APIs

2. **IOKit**: 
   - Object-oriented framework for device drivers
   - Written in C++

3. **Core Services**:
   - Core Foundation
   - Core Graphics
   - Core Audio
   - etc.

4. **Application Frameworks**:
   - **Cocoa**: Modern Objective-C/Swift framework
   - **Carbon**: Legacy C-based framework (deprecated)
   - **SwiftUI**: Declarative UI framework

### 1.5 Essential Tools

#### System Information Tools

```bash
# System version
sw_vers

# Hardware info
system_profiler SPHardwareDataType

# All system info (very verbose)
system_profiler

# Kernel info
uname -a
sysctl hw.model
sysctl kern.version
```

#### Package Managers

**Homebrew** - The de facto package manager for macOS

```bash
# Install Homebrew (if not already installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install packages
brew install wget
brew install python3
brew install htop

# Search for packages
brew search keyword

# Update Homebrew
brew update

# Upgrade installed packages
brew upgrade
```

**MacPorts** - Alternative package manager

```bash
# Install package
sudo port install wget

# Search
port search keyword
```

#### File and Binary Analysis Tools

```bash
# File type identification
file /bin/ls
file /Applications/Safari.app/Contents/MacOS/Safari

# Hex dump
hexdump -C filename | head
xxd filename | head

# Strings in binary
strings /bin/ls

# Binary information (Mach-O specific)
otool -L /bin/ls              # Linked libraries
otool -h /bin/ls              # Header
otool -l /bin/ls              # Load commands

# Code signing info
codesign -dv /bin/ls
codesign -dv --entitlements - /bin/ls
```

## Practical Exercises

### Exercise 1: Terminal Exploration

1. Open Terminal.app (or iTerm2)
2. Navigate to `/System/Applications`
3. List all applications
4. Choose one application and explore its bundle structure
5. Find and examine the main executable
6. View the Info.plist

### Exercise 2: File System Navigation

1. Explore the `/System/Library/Frameworks` directory
2. List all frameworks
3. Pick one framework and examine its structure
4. Find the main library file and check its type
5. Use `otool` to see what it links to

### Exercise 3: Process Exploration

1. Run `ps aux` and identify interesting processes
2. Use `lsof` to see what files a process has open:
   ```bash
   sudo lsof -p <PID>
   ```
3. Check what network connections are active:
   ```bash
   lsof -i
   netstat -an
   ```

### Exercise 4: System Information Gathering

1. Get your macOS version
2. Find your computer's serial number
3. List all installed applications
4. Check disk usage: `df -h`
5. Check memory usage: `vm_stat`

## Key Takeaways

- macOS is built on Unix foundations (Darwin/XNU)
- The terminal is a powerful tool for system exploration
- Application bundles are directories with a specific structure
- The file system follows Unix conventions with Apple additions
- Many tools are available for binary and system analysis

## Further Reading

- [Apple Developer Documentation](https://developer.apple.com/documentation/)
- [Darwin Source Code](https://opensource.apple.com/)
- [XNU Kernel Overview](https://github.com/apple/darwin-xnu)
- [File System Programming Guide](https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/)

## Next Module

[Module 2: Mach-O Binary Format →](02-mach-o-binaries.md)
