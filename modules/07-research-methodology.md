# Module 7: Security Research Methodology

## Overview

This module covers the systematic approach to security research on macOS, including planning, attack surface analysis, vulnerability discovery techniques, documentation, and responsible disclosure.

## Learning Objectives

By the end of this module, you will be able to:
- Develop a systematic research methodology
- Perform attack surface analysis
- Use fuzzing and testing strategies effectively
- Classify and assess vulnerabilities
- Write clear proof-of-concepts
- Follow responsible disclosure practices
- Document research effectively

## Topics

### 7.1 Research Planning and Methodology

#### The Research Process

```
1. Topic Selection
   ↓
2. Background Research
   ↓
3. Attack Surface Mapping
   ↓
4. Hypothesis Formation
   ↓
5. Testing and Analysis
   ↓
6. Vulnerability Discovery
   ↓
7. Proof-of-Concept Development
   ↓
8. Documentation
   ↓
9. Responsible Disclosure
```

#### Choosing a Research Target

**Consider:**
- **Attack surface**: Complexity, exposure, input handling
- **Security history**: Past vulnerabilities, patch frequency
- **Impact**: Privilege escalation, data access, remote code execution
- **Feasibility**: Available tools, documentation, reversibility

**Good targets:**
- System services processing untrusted input
- IPC interfaces exposed to unprivileged processes
- File format parsers
- Network protocol implementations
- Privileged helper tools
- Sandboxed app escape vectors

#### Setting Research Goals

**SMART goals:**
- **Specific**: "Find privilege escalation in XPC service X"
- **Measurable**: "Identify 3 potential attack vectors"
- **Achievable**: Based on time and resources
- **Relevant**: Meaningful security impact
- **Time-bound**: "Complete initial analysis in 2 weeks"

### 7.2 Background Research

#### Information Gathering

**Sources:**
- Apple security updates and CVE database
- Previous research papers and blog posts
- Apple developer documentation
- Open source components
- Community forums and discussions

```bash
# Search for past CVEs
curl https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=macos

# Check Apple security updates
# https://support.apple.com/en-us/HT201222

# Search for research
# Google Scholar, arXiv, Black Hat/DEF CON archives
```

#### Understanding the Target

**Questions to answer:**
- What does the target do?
- What privileges does it run with?
- What input does it accept?
- How does it communicate (IPC, network, files)?
- What frameworks does it use?
- Is source code available?

**Tools:**
```bash
# Process information
ps aux | grep target
lsof -p <PID>
launchctl list | grep target

# Binary analysis
otool -L /path/to/binary
codesign -dv /path/to/binary
nm /path/to/binary

# Dynamic analysis
lldb /path/to/binary
dtruss -p <PID>
```

### 7.3 Attack Surface Analysis

#### Identifying Attack Vectors

**Types of attack surfaces:**

1. **File-based**:
   - File parsers (images, documents, archives)
   - Configuration files
   - Plist files
   - Database files

2. **Network-based**:
   - Network services
   - Protocol implementations
   - URL handlers

3. **IPC-based**:
   - XPC services
   - Mach ports
   - Unix domain sockets
   - Distributed objects

4. **Input-based**:
   - Command-line arguments
   - Environment variables
   - Keyboard/mouse input
   - Clipboard data

#### Mapping IPC Interfaces

```bash
# Find XPC services
find /System -name "*.xpc" 2>/dev/null
find /Applications -name "*.xpc" 2>/dev/null

# Analyze XPC service
lldb /path/to/service.xpc/Contents/MacOS/service
(lldb) image dump sections
(lldb) image lookup -r -n "NSXPCInterface"

# Monitor XPC communication
sudo fs_usage -w -f filesys | grep xpc
```

#### Analyzing Entitlements

```bash
# Extract entitlements
codesign -d --entitlements :- /path/to/binary > entitlements.xml

# Look for dangerous entitlements:
# - com.apple.private.* (private entitlements)
# - com.apple.security.cs.allow-unsigned-executable-memory
# - com.apple.security.get-task-allow
# - com.apple.rootless.* (SIP exceptions)
```

### 7.4 Static Analysis Techniques

#### Reverse Engineering Workflow

```
1. Initial triage (file, otool, codesign)
   ↓
2. Identify interesting functions (nm, strings)
   ↓
3. Disassemble (Hopper, IDA, Ghidra)
   ↓
4. Identify vulnerabilities
   ↓
5. Validate with dynamic analysis
```

#### Finding Interesting Code

```bash
# Search for interesting strings
strings /path/to/binary | grep -i password
strings /path/to/binary | grep -i key
strings /path/to/binary | grep -i secret
strings /path/to/binary | grep "file://"

# Look for debugging strings
strings /path/to/binary | grep -i "debug"
strings /path/to/binary | grep -i "log"

# Find format strings (potential vulnerabilities)
strings /path/to/binary | grep "%"

# Find function names (if not stripped)
nm -g /path/to/binary | grep authenticate
nm -g /path/to/binary | grep validate
nm -g /path/to/binary | grep parse
```

#### Pattern Recognition

**Dangerous patterns:**
- Buffer operations without bounds checking
- Format string usage
- Integer overflows in size calculations
- Race conditions (TOCTOU)
- Improper input validation
- Insecure deserialization

```objc
// Example dangerous patterns to look for:

// 1. Unchecked buffer copy
strcpy(dest, source);  // BAD
strncpy(dest, source, sizeof(dest));  // Better

// 2. Format string vulnerability
printf(user_input);  // BAD
printf("%s", user_input);  // Better

// 3. Integer overflow
size_t total = count * size;  // Potential overflow
void *buffer = malloc(total);

// 4. TOCTOU race
if (access(filename, R_OK) == 0) {  // Check
    fd = open(filename, O_RDONLY);  // Use (race window here)
}
```

### 7.5 Dynamic Analysis Techniques

#### Fuzzing

Fuzzing automatically generates test inputs to find crashes.

**Types:**
- **Mutation-based**: Modify existing valid inputs
- **Generation-based**: Create inputs from scratch based on format
- **Coverage-guided**: Use code coverage to guide mutations (AFL, libFuzzer)

**Tools:**
```bash
# AFL (American Fuzzy Lop) for macOS
brew install afl-fuzz

# Compile target with AFL
afl-clang target.c -o target

# Fuzz
afl-fuzz -i input_dir -o output_dir -- ./target @@

# LibFuzzer (LLVM)
clang -fsanitize=fuzzer,address target.c -o target-fuzzer
./target-fuzzer corpus/
```

#### Creating a Simple Fuzzer

```python
#!/usr/bin/env python3
import subprocess
import random
import string

def generate_input():
    """Generate random input"""
    length = random.randint(1, 10000)
    return ''.join(random.choices(string.printable, k=length))

def fuzz_target(binary, iterations=1000):
    """Fuzz a target binary"""
    crashes = 0
    
    for i in range(iterations):
        test_input = generate_input()
        
        try:
            result = subprocess.run(
                [binary],
                input=test_input.encode(),
                timeout=5,
                capture_output=True
            )
            
            # Check for crash
            if result.returncode < 0:
                print(f"[!] Crash detected with return code {result.returncode}")
                with open(f"crash_{i}.txt", "w") as f:
                    f.write(test_input)
                crashes += 1
                
        except subprocess.TimeoutExpired:
            print(f"[!] Timeout on iteration {i}")
        except Exception as e:
            print(f"[!] Exception: {e}")
    
    print(f"\n[+] Fuzzing complete. Found {crashes} crashes.")

if __name__ == "__main__":
    fuzz_target("./target_binary")
```

#### Instrumentation

Add logging/tracing to understand behavior.

```bash
# Use LLDB to trace function calls
(lldb) target create /path/to/binary
(lldb) br set -r ".*"  # Break on all functions (careful!)
(lldb) br command add
> bt 1
> continue
> DONE
(lldb) run
```

### 7.6 Vulnerability Classification

#### Common Vulnerability Types

**Memory corruption:**
- Buffer overflow
- Heap overflow
- Use-after-free
- Double free
- Integer overflow leading to memory corruption

**Logic errors:**
- Authentication bypass
- Authorization bypass
- TOCTOU (Time-of-check-time-of-use)
- Path traversal
- Injection vulnerabilities

**Information disclosure:**
- Memory disclosure
- File disclosure
- Stack/heap leaks

**Privilege escalation:**
- Local privilege escalation (LPE)
- Sandbox escape
- SIP bypass
- Container escape

#### CVSS Scoring

Use Common Vulnerability Scoring System to assess severity:

**Metrics:**
- **Attack Vector**: Network, Adjacent, Local, Physical
- **Attack Complexity**: Low, High
- **Privileges Required**: None, Low, High
- **User Interaction**: None, Required
- **Scope**: Unchanged, Changed
- **Impact**: Confidentiality, Integrity, Availability

```bash
# Calculate CVSS score using online calculator
# https://www.first.org/cvss/calculator/3.1
```

### 7.7 Proof-of-Concept Development

#### PoC Guidelines

**Good PoC characteristics:**
- Minimal: Only demonstrates the vulnerability
- Reliable: Works consistently
- Safe: Doesn't cause system damage
- Clear: Well-commented code
- Documented: Explains what it does

#### PoC Template

```c
/*
 * CVE-XXXX-XXXX - [Vulnerability Name]
 * 
 * Description:
 *   Brief description of the vulnerability
 *
 * Impact:
 *   What an attacker can achieve
 *
 * Affected versions:
 *   macOS 10.x - 13.x
 *
 * Author: [Your Name]
 * Date: [Date]
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char *argv[]) {
    printf("[*] CVE-XXXX-XXXX PoC\n");
    printf("[*] Triggering vulnerability...\n");
    
    // PoC code here
    
    printf("[+] Vulnerability triggered successfully\n");
    return 0;
}
```

#### Testing PoC

```bash
# Test in controlled environment
# - Virtual machine
# - Test system
# - Container (if applicable)

# Document:
# - Prerequisites
# - Steps to reproduce
# - Expected results
# - Actual results
```

### 7.8 Responsible Disclosure

#### Disclosure Process

```
1. Verify vulnerability
   ↓
2. Develop PoC
   ↓
3. Document thoroughly
   ↓
4. Report to vendor
   ↓
5. Wait for acknowledgment
   ↓
6. Provide additional info if requested
   ↓
7. Wait for patch
   ↓
8. Verify fix
   ↓
9. Public disclosure (coordinated)
```

#### Reporting to Apple

**Apple Security Bounty Program:**
- Website: https://developer.apple.com/security-bounty/
- Email: product-security@apple.com
- PGP key available on website

**Report should include:**
- Detailed description
- Affected versions
- Steps to reproduce
- Proof-of-concept code
- Impact assessment
- Your contact information

#### Disclosure Timeline

**Standard timeline:**
- Day 0: Report submitted
- Day 7: Acknowledgment expected
- Day 90: Vendor has 90 days to fix
- Day 90+: Coordinated public disclosure

**Exceptions:**
- Active exploitation: Immediate
- Critical infrastructure: Extended timeline
- Vendor request: Case-by-case

### 7.9 Documentation

#### Research Report Structure

```markdown
# [Vulnerability Title]

## Executive Summary
Brief overview of the vulnerability and impact.

## Technical Details
### Background
Context and affected components.

### Vulnerability Description
Detailed explanation of the flaw.

### Attack Scenario
How an attacker would exploit this.

### Root Cause Analysis
Why the vulnerability exists.

## Proof of Concept
Code and steps to reproduce.

## Impact Assessment
Security implications and CVSS score.

## Mitigation
Temporary workarounds (if any).

## Recommendations
How to fix the vulnerability.

## Timeline
- Discovery date
- Report date
- Acknowledgment date
- Fix date
- Disclosure date

## References
Related CVEs, papers, etc.
```

#### Lab Notes

Keep detailed notes during research:

```markdown
# Research Log: [Target Name]

## Date: 2024-01-15
### Observations
- Found interesting XPC service at /System/Library/XPCServices/...
- Service runs with elevated privileges
- No input validation on method X

### Tests Performed
1. Sent malformed plist - crashed
2. Sent oversized buffer - crashed
3. Sent negative size - integer overflow

### Next Steps
- Analyze crash dumps
- Develop reliable PoC
- Test on different macOS versions
```

## Practical Exercises

### Exercise 1: Attack Surface Mapping

1. Choose a system service
2. Identify all input sources
3. Map IPC interfaces
4. Analyze entitlements
5. Create attack surface diagram

### Exercise 2: Static Analysis

1. Pick a small open-source macOS tool
2. Read the source code
3. Identify potential vulnerabilities
4. Rank by severity
5. Write a report

### Exercise 3: Fuzzing Practice

1. Create a simple file parser
2. Intentionally add a bug
3. Write a fuzzer for it
4. Run the fuzzer until crash
5. Analyze the crash

### Exercise 4: PoC Development

1. Find a published CVE for macOS
2. Read the advisory
3. Attempt to recreate the PoC
4. Test it in a VM
5. Document your findings

### Exercise 5: Full Research Project

1. Choose a research target
2. Perform full analysis
3. Document everything
4. Write a complete report
5. Present findings

## Key Takeaways

- Systematic methodology improves research effectiveness
- Attack surface analysis identifies entry points
- Both static and dynamic analysis are essential
- Fuzzing automates vulnerability discovery
- Responsible disclosure protects users
- Documentation is crucial for reproducibility
- Practice and experience improve skills

## Further Reading

- [The Art of Software Security Assessment](https://www.amazon.com/Art-Software-Security-Assessment/dp/0321444426)
- [Fuzzing: Brute Force Vulnerability Discovery](https://www.amazon.com/Fuzzing-Brute-Force-Vulnerability-Discovery/dp/0321446119)
- [Google Project Zero Blog](https://googleprojectzero.blogspot.com/)
- [Apple Security Bounty](https://developer.apple.com/security-bounty/)
- [CVE Database](https://cve.mitre.org/)

## Next Module

[Module 8: Practical Security Analysis →](08-practical-security.md)
