# Module 5: macOS Security Architecture

## Overview

macOS has a multi-layered security architecture designed to protect users from malware, unauthorized access, and system compromise. Understanding these protections is essential for security research, as they often present challenges that must be understood or bypassed for legitimate analysis.

## Learning Objectives

By the end of this module, you will be able to:
- Understand System Integrity Protection (SIP) and its implications
- Explain how Gatekeeper and notarization work
- Analyze code signatures and entitlements
- Understand sandboxing mechanisms
- Work with TCC (Transparency, Consent, and Control)
- Understand Secure Boot and hardware security features

## Topics

### 5.1 System Integrity Protection (SIP)

#### What is SIP?

System Integrity Protection (also called "rootless") restricts what even the root user can do on critical system files and directories.

**Protected locations:**
- `/System`
- `/usr` (except `/usr/local`)
- `/bin`
- `/sbin`
- Most pre-installed apps

**What SIP prevents:**
- Modifying protected files
- Loading unsigned kernel extensions
- Attaching debuggers to protected processes
- Modifying kernel memory

#### Checking SIP Status

```bash
# Check if SIP is enabled
csrutil status

# Output: System Integrity Protection status: enabled
```

#### SIP Flags

```bash
# View detailed SIP configuration
csrutil status

# Flags:
# - Filesystem protections
# - Runtime protections  
# - Debug restrictions
# - DTrace restrictions
# - NVRAM protections
# - Kernel extension restrictions
# - BaseSystem protection
```

#### Disabling SIP (For Research Only)

⚠️ **Warning**: Only disable SIP on test/research systems, never on production machines.

```bash
# 1. Reboot into Recovery Mode (Cmd+R at startup)
# 2. Open Terminal from Utilities menu
# 3. Disable SIP
csrutil disable

# 4. Reboot
reboot

# To re-enable:
csrutil enable
```

#### Partial SIP Disable

```bash
# Disable specific protections
csrutil enable --without debug    # Allow debugging
csrutil enable --without dtrace   # Allow DTrace
csrutil enable --without fs       # Disable filesystem protection
```

#### SIP and Security Research

Debugging system processes often requires SIP modifications:

```bash
# With SIP fully enabled, this fails:
lldb -p $(pgrep -x loginwindow)
# Error: attach failed: unable to attach

# With debug restrictions disabled:
csrutil enable --without debug
# Now you can attach to system processes
```

### 5.2 Gatekeeper and Notarization

#### Gatekeeper

Gatekeeper checks if apps are from identified developers and notarized by Apple.

**Three levels:**
1. **App Store only**: Only apps from Mac App Store
2. **App Store and identified developers**: Signed and notarized apps
3. **Anywhere**: Any app (removed in recent macOS)

```bash
# Check Gatekeeper status
spctl --status

# Check if app will be allowed
spctl --assess --verbose /Applications/MyApp.app

# Check specific assessment
spctl --assess --type execute /path/to/binary

# Add app to allow list (requires admin)
spctl --add /path/to/app

# Remove quarantine attribute (use with caution)
xattr -d com.apple.quarantine /path/to/app
```

#### Quarantine Attribute

Downloaded files get a quarantine attribute:

```bash
# Check for quarantine
xattr /path/to/file
# Output: com.apple.quarantine

# View quarantine info
xattr -p com.apple.quarantine /path/to/file

# Remove quarantine (allows running without Gatekeeper check)
xattr -d com.apple.quarantine /path/to/file
```

#### Notarization

Notarization is Apple's malware scanning service.

**Requirements:**
- Hardened runtime
- Secure timestamp
- Signed with Developer ID

```bash
# Submit for notarization
xcrun notarytool submit app.zip --keychain-profile "AC_PASSWORD"

# Check status
xcrun notarytool info <submission-id> --keychain-profile "AC_PASSWORD"

# Staple notarization ticket
xcrun stapler staple MyApp.app

# Validate
xcrun stapler validate MyApp.app
```

### 5.3 Code Signing

#### Understanding Code Signing

Every executable must be code signed on modern macOS.

```bash
# Check signature
codesign -dv /Applications/Safari.app

# Output shows:
# - Developer
# - Authority
# - Identifier
# - Format
# - Signature size
```

#### Detailed Signature Information

```bash
# Verbose signature info
codesign -dvvv /Applications/Safari.app

# View all signatures
codesign -d --extract-certificates /path/to/app

# Check if hardened runtime is enabled
codesign -dv --verbose=4 /path/to/app | grep flags
```

#### Code Signature Requirements

```bash
# View requirements
codesign -d --requirements - /Applications/Safari.app

# Check specific requirement
codesign --verify --deep --strict --verbose=2 /Applications/Safari.app
```

#### Entitlements

Entitlements grant special permissions to apps.

```bash
# View entitlements
codesign -d --entitlements - /Applications/Safari.app
codesign -d --entitlements - /Applications/Safari.app --xml

# Common entitlements:
# - com.apple.security.app-sandbox: App is sandboxed
# - com.apple.security.network.client: Network access
# - com.apple.security.files.user-selected.read-write: File access
# - com.apple.security.cs.allow-jit: JIT compilation
# - com.apple.security.cs.allow-unsigned-executable-memory: Execute unsigned code
# - com.apple.security.cs.disable-library-validation: Load unsigned libraries
# - com.apple.security.get-task-allow: Allow debugging
```

#### Self-Signing

```bash
# Create self-signed certificate (Keychain Access)
# Or use:
security create-keypair -a rsa -s 2048 -f /tmp/key

# Sign binary
codesign -s "Certificate Name" /path/to/binary

# Sign with entitlements
codesign -s "Certificate Name" --entitlements entitlements.plist /path/to/binary

# Ad-hoc signing (local only)
codesign -s - /path/to/binary
```

### 5.4 Sandboxing

#### App Sandbox

The App Sandbox restricts app access to system resources.

**Restrictions:**
- File system access
- Network access
- Hardware access
- IPC with other apps
- System information

```bash
# Check if app is sandboxed
codesign -d --entitlements - /Applications/TextEdit.app | grep sandbox

# View sandbox profile
sandbox-exec -n no-network ls
sandbox-exec -n no-write-except-temporary ls
```

#### Sandbox Profiles

Profiles define sandbox restrictions:

```scheme
;; Example sandbox profile
(version 1)
(deny default)
(allow file-read* (subpath "/usr/lib"))
(allow file-read* file-write* (subpath "/tmp"))
(allow network-outbound)
```

```bash
# Run with custom profile
sandbox-exec -f profile.sb /path/to/binary
```

#### Container Directories

Sandboxed apps use containers:

```bash
# Container location
~/Library/Containers/[bundle-id]

# Structure:
# - Data/              - App's writeable files
# - Container.plist    - Container metadata
```

### 5.5 Transparency, Consent, and Control (TCC)

#### What is TCC?

TCC manages privacy permissions (camera, microphone, contacts, etc.).

**Protected resources:**
- Camera
- Microphone
- Location
- Contacts
- Calendar
- Photos
- Reminders
- Accessibility
- Full Disk Access

#### TCC Database

Permissions stored in:
```bash
# User permissions
~/Library/Application Support/com.apple.TCC/TCC.db

# System permissions  
/Library/Application Support/com.apple.TCC/TCC.db
```

```bash
# View TCC permissions (requires Full Disk Access)
sqlite3 ~/Library/Application\ Support/com.apple.TCC/TCC.db "SELECT * FROM access"

# Reset permissions for app
tccutil reset All com.example.myapp

# Reset all permissions
tccutil reset All
```

#### Requesting Permissions

Apps must request permissions via Info.plist:

```xml
<key>NSCameraUsageDescription</key>
<string>This app needs camera access</string>

<key>NSMicrophoneUsageDescription</key>
<string>This app needs microphone access</string>
```

### 5.6 Secure Boot and Hardware Security

#### Secure Boot Process

```
┌─────────────────────────────┐
│    Hardware (Boot ROM)      │
│    - Verify iBoot signature │
├─────────────────────────────┤
│         iBoot               │
│    - Verify kernel signature│
├─────────────────────────────┤
│         Kernel (XNU)        │
│    - Verify kexts           │
├─────────────────────────────┤
│      User Space Launch      │
└─────────────────────────────┘
```

#### T2 Security Chip / Apple Silicon

**T2 Chip (Intel Macs):**
- Secure boot
- Encrypted storage
- Hardware key generation
- Touch ID processing

**Apple Silicon Security:**
- Secure Enclave
- Secure boot chain
- Kernel integrity
- Pointer Authentication Codes (PAC)

```bash
# Check chip type
system_profiler SPHardwareDataType | grep Chip

# Check secure boot status (Apple Silicon)
nvram 94b73556-2197-4702-82a8-3e1337dafbfb:AppleSecureBootPolicy
```

#### FileVault

Full-disk encryption using XTS-AES-128.

```bash
# Check FileVault status
fdesetup status

# Enable FileVault (requires admin)
sudo fdesetup enable

# List FileVault users
sudo fdesetup list

# Add user
sudo fdesetup add -usertoadd username
```

### 5.7 Hardened Runtime

Introduced in macOS 10.14, hardened runtime imposes additional restrictions.

**Restrictions:**
- No JIT without entitlement
- No dynamic linker environment variables
- Library validation
- Restricted debugging

```bash
# Check if hardened runtime enabled
codesign -dv --verbose=4 /path/to/app | grep runtime

# Hardened runtime entitlements:
# - com.apple.security.cs.allow-jit
# - com.apple.security.cs.allow-unsigned-executable-memory
# - com.apple.security.cs.allow-dyld-environment-variables
# - com.apple.security.cs.disable-library-validation
# - com.apple.security.cs.disable-executable-page-protection
```

### 5.8 Bypassing Protections (For Research)

⚠️ **For legitimate research and testing only!**

#### Bypassing Gatekeeper

```bash
# Remove quarantine
xattr -d com.apple.quarantine /path/to/app

# Right-click → Open (first time)
# Or: System Preferences → Security & Privacy → Open Anyway
```

#### Bypassing Code Signing (Development)

```bash
# Disable library validation
codesign -s - --entitlements entitlements.plist --force binary

# entitlements.plist:
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.security.cs.disable-library-validation</key>
    <true/>
</dict>
</plist>
```

#### Working with SIP Enabled

Techniques for research with SIP:
1. Use system debugger entitlements (legit debugging)
2. Debug non-system processes
3. Use DTrace with `csrutil enable --without dtrace`
4. Analyze in VM with SIP disabled

## Practical Exercises

### Exercise 1: SIP Exploration

1. Check current SIP status
2. Boot into Recovery Mode
3. Disable SIP debug restrictions
4. Reboot and verify
5. Attempt to debug a system process
6. Re-enable full SIP

### Exercise 2: Code Signature Analysis

1. Choose 5 different applications
2. Examine their code signatures
3. Extract and compare entitlements
4. Identify sandboxed vs non-sandboxed apps
5. Document security differences

### Exercise 3: Gatekeeper Testing

1. Download an unsigned app
2. Examine quarantine attribute
3. Attempt to run (observe Gatekeeper)
4. Remove quarantine and run
5. Document the process

### Exercise 4: TCC Database Analysis

1. Grant Full Disk Access to Terminal
2. Examine TCC database
3. Identify apps with sensitive permissions
4. Reset permissions for test app
5. Observe permission request flow

### Exercise 5: Sandbox Profile Creation

1. Create a restrictive sandbox profile
2. Run a test app with the profile
3. Observe denied operations
4. Modify profile to allow necessary operations
5. Document the process

## Key Takeaways

- SIP protects critical system files and processes
- Gatekeeper and notarization verify app sources
- Code signing ensures binary integrity
- Entitlements grant specific capabilities
- TCC manages privacy permissions
- Multiple layers provide defense in depth
- Research requires understanding these protections

## Further Reading

- [Apple Platform Security Guide](https://support.apple.com/guide/security/welcome/web)
- [System Integrity Protection](https://developer.apple.com/documentation/security/disabling_and_enabling_system_integrity_protection)
- [Code Signing Guide](https://developer.apple.com/library/archive/documentation/Security/Conceptual/CodeSigningGuide/)
- [App Sandbox Design Guide](https://developer.apple.com/library/archive/documentation/Security/Conceptual/AppSandboxDesignGuide/)
- [TCC Database Internals](https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive)

## Next Module

[Module 6: macOS Internals →](06-macos-internals.md)
