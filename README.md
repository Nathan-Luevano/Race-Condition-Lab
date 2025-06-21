# Race Condition Vulnerability Lab

A comprehensive demonstration of Time-of-Check-Time-of-Use (TOCTOU) race condition vulnerabilities in Unix-like systems. This lab explores how symbolic link attacks can exploit the time window between access permission checks and file operations in setuid programs.

## Video Walkthrough

[![Race Condition Vulnerability Lab Walkthrough](https://img.youtube.com/vi/Gf91QS8l-i8/maxresdefault.jpg)](https://www.youtube.com/watch?v=Gf91QS8l-i8)

*Click the image above to watch a complete walkthrough of this lab*

## Overview

This repository contains a vulnerable C program and attack scripts that demonstrate a classic race condition vulnerability. The vulnerable program appears harmless at first glance but contains a critical flaw in how it handles file access permissions, allowing attackers to manipulate symbolic links and gain unauthorized access to protected system files.

## Files Included

### vulp.c - Vulnerable Program
A setuid root program that:
- Accepts user input (up to 50 characters)
- Checks write permissions for `/tmp/XYZ` using `access()`
- Opens and appends user input to the file if permission check passes
- Contains a race condition between the permission check and file operation

### target_process.sh - Attack Orchestration Script
Bash script that:
- Monitors changes to `/etc/passwd` file
- Continuously executes the vulnerable program with crafted input
- Stops execution when the target file has been successfully modified
- Demonstrates the persistence required for successful race condition attacks

## Vulnerability Details

### The Race Condition
The vulnerability exists in the time window between:
1. **Time of Check**: `access(fn, W_OK)` - checking write permissions
2. **Time of Use**: `fopen(fn, "a+")` - actually opening the file

During this brief window, an attacker can:
- Remove the original `/tmp/XYZ` file
- Create a symbolic link from `/tmp/XYZ` to a protected file (e.g., `/etc/passwd`)
- Cause the program to write to the protected file instead

### Attack Methodology
1. **Target Selection**: `/etc/passwd` - the system password file
2. **Payload Creation**: Crafted user entry with root privileges (UID 0)
3. **Symbolic Link Manipulation**: Rapidly switching `/tmp/XYZ` between regular file and symbolic link
4. **Privilege Escalation**: Adding a new root account without password

## Prerequisites and Setup

### System Requirements
- Linux system (Ubuntu/Debian preferred)
- GCC compiler
- Root/sudo access for initial setup

### Disabling Ubuntu Protection
Ubuntu 10.10+ includes built-in protection against symbolic link attacks. Disable this protection:

```bash
sudo sysctl -w fs.protected_symlinks=0
```

To re-enable protection:
```bash
sudo sysctl -w fs.protected_symlinks=1
```

### Compilation and Setup
1. Compile the vulnerable program:
```bash
gcc vulp.c -o vulp
```

2. Set appropriate permissions (setuid root):
```bash
sudo chown root vulp
sudo chmod 4755 vulp
```

3. Make scripts executable:
```bash
chmod +x target_process.sh
```

## Attack Demonstration

### Payload Construction
The attack payload creates a new user entry for `/etc/passwd`:
```
test:U6aMy0wojraho:0:0:test:/root:/bin/bash
```

Where:
- `test`: Username
- `U6aMy0wojraho`: Passwordless hash (empty password)
- `0`: User ID (root privilege)
- `0`: Group ID (root group)

### Execution Steps
1. **Start the target process**:
```bash
./target_process.sh
```

2. **Run the attack process** (in separate terminal):
```bash
./attack.sh  # (Attack script creating symbolic links)
```

3. **Monitor for success**: The target process will output "STOP... The passwd file has been changed" when successful

4. **Test privilege escalation**:
```bash
su test  # No password required
id       # Should show uid=0 (root)
```

## Key Learning Objectives

### Security Concepts Demonstrated
- **TOCTOU Vulnerabilities**: Understanding time-based race conditions
- **Setuid Program Risks**: How privilege escalation can be exploited
- **Symbolic Link Attacks**: Manipulating file system links for unauthorized access
- **System File Protection**: Importance of atomic operations and proper validation

### Attack Techniques
- **Race Condition Exploitation**: Timing attacks against file operations
- **Symbolic Link Manipulation**: Rapid switching between file types
- **Privilege Escalation**: Gaining root access through system file modification
- **Persistence Mechanisms**: Continuous attack attempts until success

## Defensive Measures

### Code-Level Protections
1. **Use file descriptors consistently**: Open file once and reuse the descriptor
2. **Implement atomic operations**: Use `openat()` with `O_NOFOLLOW` flag
3. **Avoid TOCTOU patterns**: Eliminate separate check and use operations
4. **Validate file types**: Ensure files are not symbolic links before use

### System-Level Protections
1. **Enable symbolic link protection**: Keep `fs.protected_symlinks=1`
2. **Use temporary directories with proper permissions**: Restrict access to temp files
3. **Implement mandatory access controls**: SELinux, AppArmor, etc.
4. **Regular security audits**: Identify and fix TOCTOU vulnerabilities

## Educational Value

This lab demonstrates:
- Real-world vulnerability exploitation techniques
- The importance of secure coding practices in privileged programs
- How seemingly minor timing issues can lead to critical security flaws
- The effectiveness of modern OS protections against classic attack vectors

## Warning

This code is for educational purposes only. The techniques demonstrated should only be used in controlled environments for learning about security vulnerabilities. Unauthorized use of these techniques against systems you do not own is illegal and unethical.

## Additional Notes

- The attack success rate depends on system load and timing
- Multiple attempts may be required for successful exploitation
- Modern systems have various protections that make this attack more difficult
- Understanding this vulnerability helps in writing more secure code and implementing proper protections
