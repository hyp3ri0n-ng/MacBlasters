# Labs and Exercises

This directory contains hands-on labs and exercises for the MacBlasters course. Each module has corresponding practical exercises to reinforce learning.

## Lab Environment Setup

### Requirements

- macOS system (physical or virtual)
- Xcode Command Line Tools
- Homebrew (recommended)
- At least 50GB free disk space
- Admin/sudo access

### Installation

```bash
# Install Xcode Command Line Tools
xcode-select --install

# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install useful tools
brew install wget curl git
brew install python3
brew install llvm
```

### Virtual Machine Setup (Optional)

For security research, it's recommended to use a VM:

1. **Using VMware Fusion** (best compatibility)
   - Download macOS installer
   - Create new VM
   - Install macOS

2. **Using UTM** (free, Apple Silicon native)
   - Install UTM from App Store or GitHub
   - Create macOS VM
   - Good for testing on Apple Silicon

3. **Using Parallels Desktop**
   - Commercial option
   - Good performance
   - Easy setup

### Creating Snapshots

Before destructive testing:

```bash
# VMware Fusion
# Take snapshot through GUI: Virtual Machine → Snapshots

# UTM
# Take snapshot through GUI: Machine → Save State

# Or use Time Machine for backup
```

## Lab Structure

Each module has its own lab directory:

```
labs/
├── README.md (this file)
├── module-01-fundamentals/
├── module-02-mach-o/
├── module-03-lldb-basics/
├── module-04-lldb-advanced/
├── module-05-security/
├── module-06-internals/
├── module-07-research/
└── module-08-practical/
```

## Module 1: macOS Fundamentals Labs

### Lab 1.1: Terminal Mastery
**Objective**: Master basic terminal operations
**Duration**: 30 minutes

Tasks:
1. Navigate the file system
2. Explore application bundles
3. Use essential commands
4. Create shell scripts

**Files**: `labs/module-01-fundamentals/lab-1.1-terminal.md`

### Lab 1.2: File System Exploration
**Objective**: Understand macOS file system structure
**Duration**: 45 minutes

Tasks:
1. Map system directories
2. Find hidden files
3. Examine permissions
4. Locate important system files

**Files**: `labs/module-01-fundamentals/lab-1.2-filesystem.md`

## Module 2: Mach-O Binary Labs

### Lab 2.1: Binary Analysis
**Objective**: Analyze Mach-O binary structure
**Duration**: 1 hour

Tasks:
1. Examine headers
2. List load commands
3. Analyze segments and sections
4. Extract information

**Files**: `labs/module-02-mach-o/lab-2.1-analysis.md`

### Lab 2.2: Universal Binaries
**Objective**: Work with multi-architecture binaries
**Duration**: 45 minutes

Tasks:
1. Identify architectures
2. Extract slices
3. Compare architectures
4. Create universal binary

**Files**: `labs/module-02-mach-o/lab-2.2-universal.md`

## Module 3: LLDB Basics Labs

### Lab 3.1: First Steps with LLDB
**Objective**: Learn basic LLDB operations
**Duration**: 1 hour

Tasks:
1. Launch programs in LLDB
2. Set breakpoints
3. Examine variables
4. Step through code

**Files**: `labs/module-03-lldb-basics/lab-3.1-first-steps.md`

### Lab 3.2: Memory and Registers
**Objective**: Examine and modify memory
**Duration**: 1 hour

Tasks:
1. Read memory regions
2. Modify memory
3. Work with registers
4. Understand stack frames

**Files**: `labs/module-03-lldb-basics/lab-3.2-memory.md`

## Module 4: Advanced LLDB Labs

### Lab 4.1: Python Scripting
**Objective**: Extend LLDB with Python
**Duration**: 2 hours

Tasks:
1. Write custom commands
2. Automate debugging tasks
3. Create analysis scripts
4. Build debugging tools

**Files**: `labs/module-04-lldb-advanced/lab-4.1-python.md`

### Lab 4.2: Stripped Binary Analysis
**Objective**: Debug without symbols
**Duration**: 1.5 hours

Tasks:
1. Find entry points
2. Identify functions
3. Reconstruct symbols
4. Trace execution

**Files**: `labs/module-04-lldb-advanced/lab-4.2-stripped.md`

## Module 5: Security Architecture Labs

### Lab 5.1: SIP Exploration
**Objective**: Understand System Integrity Protection
**Duration**: 1 hour

Tasks:
1. Check SIP status
2. Test restrictions
3. Work with SIP-protected paths
4. Document behavior

**Files**: `labs/module-05-security/lab-5.1-sip.md`

### Lab 5.2: Code Signing Analysis
**Objective**: Analyze code signatures and entitlements
**Duration**: 1.5 hours

Tasks:
1. Examine signatures
2. Extract entitlements
3. Verify signatures
4. Self-sign binaries

**Files**: `labs/module-05-security/lab-5.2-codesign.md`

## Module 6: macOS Internals Labs

### Lab 6.1: System Call Tracing
**Objective**: Trace and analyze system calls
**Duration**: 1 hour

Tasks:
1. Use DTrace
2. Monitor syscalls
3. Analyze patterns
4. Create custom traces

**Files**: `labs/module-06-internals/lab-6.1-syscalls.md`

### Lab 6.2: XPC Service Analysis
**Objective**: Analyze XPC communication
**Duration**: 2 hours

Tasks:
1. Find XPC services
2. Analyze protocols
3. Monitor communication
4. Test security

**Files**: `labs/module-06-internals/lab-6.2-xpc.md`

## Module 7: Research Methodology Labs

### Lab 7.1: Attack Surface Mapping
**Objective**: Map attack surface of an application
**Duration**: 2 hours

Tasks:
1. Identify inputs
2. Map IPC interfaces
3. Find attack vectors
4. Document findings

**Files**: `labs/module-07-research/lab-7.1-attack-surface.md`

### Lab 7.2: Fuzzing Exercise
**Objective**: Fuzz a target application
**Duration**: 2 hours

Tasks:
1. Setup fuzzer
2. Create test cases
3. Run fuzzing campaign
4. Analyze crashes

**Files**: `labs/module-07-research/lab-7.2-fuzzing.md`

## Module 8: Practical Security Labs

### Lab 8.1: Vulnerability Discovery
**Objective**: Find and exploit a vulnerability
**Duration**: 3 hours

Tasks:
1. Analyze provided binary
2. Identify vulnerability
3. Develop exploit
4. Write PoC

**Files**: `labs/module-08-practical/lab-8.1-vuln-discovery.md`

### Lab 8.2: CTF Challenges
**Objective**: Solve security challenges
**Duration**: 4 hours

Tasks:
1. Reverse engineering challenges
2. Binary exploitation
3. Privilege escalation
4. Flag capture

**Files**: `labs/module-08-practical/lab-8.2-ctf.md`

## Lab Guidelines

### Before Starting

1. **Read instructions carefully**
2. **Set up proper environment**
3. **Take snapshots/backups**
4. **Have documentation ready**

### During Labs

1. **Take notes** - Document what you learn
2. **Experiment** - Try variations
3. **Ask questions** - Use community resources
4. **Don't skip steps** - Each builds on previous

### After Completion

1. **Review solutions** - Compare with others
2. **Document findings** - Write up your work
3. **Explore further** - Extend the exercises
4. **Share knowledge** - Help others

## Safety Guidelines

### Important Warnings

⚠️ **Only test on systems you own or have explicit permission to test**

⚠️ **Never disable SIP on production systems**

⚠️ **Don't run untrusted code on your main system**

⚠️ **Backup important data before security testing**

⚠️ **Use VMs for potentially dangerous operations**

### Ethical Guidelines

- **Obtain authorization** before testing any system
- **Respect privacy** - Don't access unauthorized data
- **Follow disclosure** - Report vulnerabilities responsibly
- **Obey laws** - Stay within legal boundaries
- **Be professional** - Maintain ethical standards

## Getting Help

### Resources

- **Course discussion forum**
- **Discord/Slack community**
- **Stack Overflow**
- **Reddit: r/macOSsecurity**
- **GitHub Issues**

### Troubleshooting

If you encounter issues:

1. Check the FAQ
2. Review error messages carefully
3. Search for similar problems
4. Ask in community channels
5. Open an issue with details

## Contributing Labs

Want to contribute a lab? Great!

**Requirements:**
- Clear learning objectives
- Step-by-step instructions
- Expected outcomes
- Sample code/binaries
- Solutions (in separate file)
- Testing on multiple macOS versions

**Process:**
1. Open an issue proposing the lab
2. Get feedback from maintainers
3. Develop the lab
4. Test thoroughly
5. Submit pull request

## Additional Practice

### External Resources

- **HackTheBox** - Practice penetration testing
- **TryHackMe** - Guided security learning
- **OverTheWire** - Command-line challenges
- **PicoCTF** - Beginner-friendly CTF
- **RingZer0 CTF** - Various challenges

### Real-World Practice

- **Open source projects** - Audit code
- **Bug bounty programs** - Find real vulnerabilities
- **CTF competitions** - Compete with others
- **Personal projects** - Build security tools

## Lab Completion Checklist

- [ ] Complete all required tasks
- [ ] Document findings
- [ ] Answer review questions
- [ ] Experiment with variations
- [ ] Share interesting discoveries
- [ ] Help others with questions

## Next Steps

1. Choose a module to start
2. Set up your environment
3. Begin with Lab 1.1
4. Progress through modules sequentially
5. Complete all labs for certification

---

Ready to begin? Start with [Module 1 Labs](module-01-fundamentals/README.md)!

[← Back to Course Home](../README.md)
