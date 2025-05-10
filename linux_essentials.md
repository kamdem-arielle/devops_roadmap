

# Linux Basics for DevOps

This document covers essential Linux concepts and commands for DevOps engineers.

## Table of Contents
1. [Linux Introduction](#linux-introduction)
2. [Linux Principles](#linux-principles)
3. [Important Directories](#important-directories)
4. [Basic Commands](#basic-commands)
5. [File Operations](#file-operations)
6. [VIM Editor](#vim-editor)
7. [File Permissions](#file-permissions)
8. [Users & Groups](#users--groups)
9. [Package Management](#package-management)
10. [System Services](#system-services)
11. [Useful Commands](#useful-commands)

---

## Linux Introduction

### Open Source Principles
- Four essential freedoms:
  - Run the program for any purpose
  - Study and modify the source code
  - Redistribute the program
  - Create derivative programs

### Linux Origins
- **1984**: GNU Project and Free Software Foundation created UNIX utilities and GPL license
- **1991**: Linus Torvalds created the Linux kernel
- Today: Linux kernel + GNU utilities = complete OS

---

## Linux Principles
- Everything is a file (including hardware)
- Small, single-purpose programs
- Chain programs together for complex tasks
- Configuration stored in text files
- Avoid captive user interfaces

### Why Linux for DevOps?
- Open source
- Strong community support
- Highly customizable
- Dominant on servers
- Most DevOps tools are Linux-native
- Excellent for automation
- Secure by design

---

## Important Directories

| Directory | Purpose |
|-----------|---------|
| `/home/username` | User home directories |
| `/bin`, `/usr/bin` | User executables |
| `/sbin`, `/usr/sbin` | System executables |
| `/etc` | Configuration files |
| `/tmp` | Temporary files |
| `/var`, `/srv` | Server data |
| `/proc`, `/sys` | System information |
| `/lib` | Shared libraries |

---

## Basic Commands

### Navigation
```bash
pwd          # Print working directory
cd ~         # Go to home directory
cd /path     # Change to absolute path
cd folder    # Change to relative path
ls           # List directory contents
ls -l        # Long listing format
ls -a        # Show hidden files
```

### File/Directory Operations
```bash
mkdir dirname       # Create directory
touch file1 file2   # Create empty files
cp file1 dest/      # Copy file to directory
cp -r dir1 dest/    # Copy directory recursively
mv file1 dest/      # Move/rename file
rm file             # Remove file
rm -rf dir          # Remove directory recursively
```

### Path Types
- **Absolute path**: Starts from root (`/home/user/file`)
- **Relative path**: Relative to current directory (`../folder/file`)

---

## File Operations

### Find Command
```bash
find /path -name filename  # Search by name
find /home -size +10M     # Find files >10MB
find / -type f -user root # Find files owned by root
```

### Grep (Text Search)
```bash
grep "text" file          # Search for text
grep -i "Text" file       # Case-insensitive
grep -v "text" file       # Invert match (lines NOT containing)
grep -r "text" /path      # Recursive search
```

### File Viewing
```bash
cat file       # Display entire file
less file      # View file page by page
head -n 5 file # Show first 5 lines
tail -n 5 file # Show last 5 lines
tail -f log    # Follow log file in real-time
```

### I/O Redirection
```bash
command > file    # Overwrite file with output
command >> file   # Append output to file
command 2> file   # Redirect errors
command &>> file  # Redirect all output
```

### Piping
```bash
ls | head -5      # Show first 5 files
ps aux | grep ssh # Find SSH processes
```

---

## VIM Editor

### Modes
1. **Command mode** (default)
2. **Insert mode** (press `i`)
3. **Extended mode** (press `Esc` then `:`)

### Common Commands
```vim
i          # Enter insert mode
Esc        # Return to command mode
:w         # Save
:q         # Quit
:wq        # Save and quit
:q!        # Force quit without saving
dd         # Delete line
yy         # Copy line
p          # Paste
/search    # Search forward
?search    # Search backward
```

---

## File Permissions

### View Permissions
```bash
ls -l file
# Output: -rwxr-xr-- 1 user group size date filename
# First character: file type (-=file, d=directory, l=link)
# Next 9 characters: permissions (rwx for user, group, others)
```

### Change Permissions
**Symbolic method:**
```bash
chmod u+x file    # Add execute for user
chmod g-w file    # Remove write for group
chmod o=r file    # Set others to read-only
```

**Numeric method:**
```bash
chmod 755 file    # rwxr-xr-x
# 4=read, 2=write, 1=execute
# First digit: user, second: group, third: others
```

### Change Ownership
```bash
chown user:group file  # Change owner and group
chown user file        # Change owner only
chgrp group file       # Change group only
```

---

## Users & Groups

### User Management
```bash
useradd username      # Create user (RedHat)
adduser username      # Create user (Ubuntu)
passwd username       # Set password
usermod -aG group user # Add user to group
userdel -r username   # Delete user with home dir
```

### Important Files
```bash
/etc/passwd     # User accounts
/etc/shadow     # Encrypted passwords
/etc/group      # Group definitions
```

### Sudo Privileges
```bash
sudo -i         # Become root
visudo          # Edit sudoers file
# Add line: "username ALL=(ALL) NOPASSWD:ALL" for passwordless sudo
```

---

## Package Management

### RPM (RedHat/CentOS)
```bash
rpm -ivh package.rpm   # Install
rpm -e package         # Remove
rpm -qa                # List installed
```

### YUM/DNF (RedHat/CentOS)
```bash
yum install package    # Install
yum remove package     # Remove
yum update             # Update all
dnf search package     # Search
```

### APT (Debian/Ubuntu)
```bash
apt install package    # Install
apt remove package    # Remove
apt update            # Update package lists
apt upgrade           # Upgrade packages
```

---

## System Services

### Systemctl Commands
```bash
systemctl start service    # Start service
systemctl stop service     # Stop service
systemctl restart service  # Restart service
systemctl status service   # Check status
systemctl enable service   # Enable at boot
systemctl disable service  # Disable at boot
```

### Service Examples
```bash
# CentOS (httpd)
sudo systemctl start httpd

# Ubuntu (apache2)
sudo systemctl start apache2
```

---

## Useful Commands

### System Info
```bash
uname -a           # System information
hostname           # Show hostname
uptime             # System uptime
free -h            # Memory usage
df -h              # Disk space
top                # Process monitor
```

### Networking
```bash
ping host          # Test connectivity
ssh user@host      # Secure shell
scp file user@host:/path  # Secure copy
netstat -tuln      # Listening ports
```

### Process Management
```bash
ps aux             # List processes
kill PID           # Kill process
killall process    # Kill all instances
pkill process      # Kill by name
```

### Compression
```bash
tar -czf archive.tar.gz folder  # Create compressed archive
tar -xzf archive.tar.gz         # Extract
gzip file                       # Compress to .gz
gunzip file.gz                  # Decompress
```

---

## Cheat Sheets

### File Types
| Character | Type        |
|-----------|-------------|
| `-`       | Regular file|
| `d`       | Directory   |
| `l`       | Link        |
| `c`       | Special file|
| `s`       | Socket      |
| `p`       | Pipe        |

### Permission Codes
| # | Permission |
|---|------------|
| 0 | ---        |
| 1 | --x        |
| 2 | -w-        |
| 3 | -wx        |
| 4 | r--        |
| 5 | r-x        |
| 6 | rw-        |
| 7 | rwx        |

