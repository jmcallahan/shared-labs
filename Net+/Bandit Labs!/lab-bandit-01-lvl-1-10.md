---
title: "Bandit Levels 0-10: SSH Fundamentals and Linux File Navigation"
difficulty: Foundation
Cert:
  - Network+
  - Sec+
  - Linux+
Exam-Objectives:
  - Sec+ 4.4
  - Sec+ 4.5
  - Lin+ 1.1
  - Lin+ 1.3
created: 2026-02-13
completed:
time-hours: 2
complete: false
GitHub-ready: false
evidence-files: 12
notes:
tags:
  - linux
  - ssh
  - bash
  - file-navigation
  - soc-analyst-skills
  - overthewire
  - capture-the-flag
---
---
## Scenario

You're a **junior SOC analyst** who just gained access to a compromised Linux server for forensic investigation. The attackers left behind a series of passwords hidden in various locations. Your task is to **practice Linux command-line skills** in a safe, gamified environment (OverTheWire Bandit) before performing real incident response.

**Bandit is the perfect training ground** because:

- No VMs required (SSH directly to `bandit.labs.overthewire.org`)
- Progressive difficulty (each level builds on previous)
- Mirrors real forensics (finding hidden files, reading logs, decoding strings)

This lab series covers **Levels 0-10**, which teach:

- SSH authentication and key management
- File navigation (`ls`, `cd`, `cat`, `file`)
- Searching file contents (`grep`, `find`, `strings`)
- Working with special characters in filenames

## Learning Objectives

By completing Levels 0-10, you will demonstrate ability to:

- **Authenticate via SSH** using passwords and key files
- **Navigate Linux directories** using absolute and relative paths
- **Read file contents** with `cat`, `more`, `less`
- **Find files** by name, size, or properties using `find`
- **Search text** within files using `grep`
- **Handle special characters** in filenames (spaces, dashes, escapes)
- **Examine file types** to identify encoded or binary data

## Required Environment

### Hardware/Software

- **SSH Client**:
    - Linux/macOS: Built-in `ssh` command
    - Windows: PuTTY, WSL, or Windows Terminal (built-in OpenSSH)
- **Internet Connection**: Access to `bandit.labs.overthewire.org` port 2220
- **Text Editor**: Notepad++, VS Code, or nano (for documenting passwords)

### Host System

- **OS**: Any (Windows, Linux, macOS)
- **RAM**: 2GB minimum
- **Network**: Internet access (SSH outbound port 2220)

## Important Rules

**DO NOT SHARE PASSWORDS**: Bandit is a learning platform. Finding passwords is the learning process. Skipping ahead defeats the purpose.

**Password Storage**: Keep a local text file of passwords for each level (you'll need them to progress).

**No Automation**: Don't script the solutions. Type commands manually to build muscle memory.

## Deliverables

### Evidence Files (Required)

1. `bandit0-connection.txt` - SSH connection output
2. `bandit0-password-found.txt` - Password for Level 1
3. `bandit1-dashed-filename.txt` - Contents of file named `-`
4. `bandit2-spaces-filename.txt` - Contents of file with spaces
5. `bandit3-hidden-file.txt` - Hidden file in inhere directory
6. `bandit4-human-readable.txt` - Only human-readable file
7. `bandit5-specific-file.txt` - File with exact size 1033 bytes
8. `bandit6-system-wide-search.txt` - File owned by bandit7:bandit6
9. `bandit7-grep-word.txt` - Line containing "millionth"
10. `bandit8-unique-line.txt` - Line that appears only once
11. `bandit9-strings-extraction.txt` - Human-readable strings from binary
12. `bandit10-base64-decode.txt` - Base64 decoded password

### Documentation (Required)

- **Command cheat sheet**: One-liner commands for each level
- **Lessons learned**: What each level teaches about Linux forensics

## Lab Tasks

### Level 0: SSH Connection Basics

**Objective**: Connect to remote server using SSH with password authentication.

#### Connection Details

```
Host: bandit.labs.overthewire.org
Port: 2220
Username: bandit0
Password: bandit0
```

#### Task 0.1: Connect via SSH

- [ ] **Linux/macOS**:
    
    ```bash
    ssh bandit0@bandit.labs.overthewire.org -p 2220
    # Enter password: bandit0
    ```
    
- [ ] **Windows (PowerShell)**:
    
    ```powershell
    ssh bandit0@bandit.labs.overthewire.org -p 2220
    # Enter password: bandit0
    ```
    
- [ ] **PuTTY (Windows)**:
    
    ```
    Host Name: bandit.labs.overthewire.org
    Port: 2220
    Connection Type: SSH
    Login as: bandit0
    Password: bandit0
    ```
    

**Expected Output**:

```
Welcome to OverTheWire!
bandit0@bandit:~$
```

#### Task 0.2: Find Password for Next Level

- [ ] **List files in home directory**:
    
    ```bash
    ls
    # Output: readme
    ```
    
- [ ] **Read file contents**:
    
    ```bash
    cat readme
    # Output: [password for bandit1]
    ```
    
- [ ] **Copy password** and save to `bandit0-password-found.txt`
    
- [ ] **Exit SSH session**:
    
    ```bash
    exit
    ```
    

**🔑 Key Concept**: Passwords are stored in plaintext files. Always check home directory first.

---

### Level 1: Dashed Filenames

**Connection**:

```bash
ssh bandit1@bandit.labs.overthewire.org -p 2220
# Password: [from Level 0]
```

**Objective**: Read a file named `-` (dash character, which bash interprets as stdin).

#### Challenge

- [ ] **Attempt to read file**:
    
    ```bash
    cat -
    # Problem: Hangs (waiting for stdin input)
    # Press Ctrl+C to exit
    ```
    
- [ ] **Solution: Use ./ to specify current directory**:
    
    ```bash
    cat ./-
    # Output: [password for bandit2]
    ```
    

**Alternative Methods**:

```bash
cat < -          # Redirect file as input
/bin/cat -       # Full path to cat
cat -- -         # End of options delimiter
```

**Evidence**: Save output to `bandit1-dashed-filename.txt`

**🔑 Key Concept**: Special characters in filenames require escaping or path specification.

---

### Level 2: Filenames with Spaces

**Connection**:

```bash
ssh bandit2@bandit.labs.overthewire.org -p 2220
```

**Objective**: Read a file named `spaces in this filename`.

#### Challenge

- [ ] **List files**:
    
    ```bash
    ls
    # Output: spaces in this filename
    ```
    
- [ ] **Attempt to read (WRONG)**:
    
    ```bash
    cat spaces in this filename
    # Error: cat tries to read 4 separate files: "spaces", "in", "this", "filename"
    ```
    
- [ ] **Solution 1: Escape spaces with backslash**:
    
    ```bash
    cat spaces\ in\ this\ filename
    ```
    
- [ ] **Solution 2: Use tab completion**:
    
    ```bash
    cat [press TAB after typing 'cat s']
    # Bash auto-completes with escaped spaces
    ```
    
- [ ] **Solution 3: Quote the filename**:
    
    ```bash
    cat "spaces in this filename"
    ```
    

**Evidence**: Save to `bandit2-spaces-filename.txt`

**🔑 Key Concept**: Use quotes or backslashes to handle spaces in filenames.

---

### Level 3: Hidden Files

**Connection**:

```bash
ssh bandit3@bandit.labs.overthewire.org -p 2220
```

**Objective**: Find hidden file in the `inhere` directory.

#### Challenge

- [ ] **List directory contents**:
    
    ```bash
    ls
    # Output: inhere
    
    cd inhere
    ls
    # Output: (empty)
    ```
    
- [ ] **Show hidden files (start with dot)**:
    
    ```bash
    ls -a
    # Output: .  ..  .hidden
    ```
    
- [ ] **Read hidden file**:
    
    ```bash
    cat .hidden
    # Output: [password for bandit4]
    ```
    

**Alternative (one-liner)**:

```bash
cat inhere/.hidden
```

**Evidence**: Save to `bandit3-hidden-file.txt`

**🔑 Key Concept**: Hidden files start with `.` and require `-a` flag to list.

---

### Level 4: Human-Readable Files

**Connection**:

```bash
ssh bandit4@bandit.labs.overthewire.org -p 2220
```

**Objective**: Find the only human-readable file among 10 files in `inhere`.

#### Challenge

- [ ] **List files**:
    
    ```bash
    cd inhere
    ls
    # Output: -file00  -file01  -file02 ... -file09
    ```
    
- [ ] **Check file types**:
    
    ```bash
    file ./*
    # Output:
    # ./-file00: data
    # ./-file01: data
    # ./-file07: ASCII text
    # ./-file08: data
    ```
    
- [ ] **Read the ASCII text file**:
    
    ```bash
    cat ./-file07
    # Output: [password for bandit5]
    ```
    

**Alternative (find + grep)**:

```bash
file ./* | grep "ASCII"
# Output: ./-file07: ASCII text

cat ./-file07
```

**Evidence**: Save to `bandit4-human-readable.txt`

**🔑 Key Concept**: Use `file` command to identify file types (binary vs. text).

---

### Level 5: File with Specific Properties

**Connection**:

```bash
ssh bandit5@bandit.labs.overthewire.org -p 2220
```

**Objective**: Find file that is:

- Human-readable
- 1033 bytes in size
- Not executable

#### Challenge

- [ ] **Explore directory structure**:
    
    ```bash
    cd inhere
    ls
    # Output: maybehere00  maybehere01 ... maybehere19
    
    ls maybehere00
    # Many files per subdirectory
    ```
    
- [ ] **Use find with multiple criteria**:
    
    ```bash
    find . -type f -size 1033c ! -executable
    # Output: ./maybehere07/.file2
    ```
    

**Breakdown**:

- `.` = search current directory
    
- `-type f` = files only (not directories)
    
- `-size 1033c` = exactly 1033 bytes (c = bytes)
    
- `! -executable` = NOT executable
    
- [ ] **Read the file**:
    
    ```bash
    cat ./maybehere07/.file2
    # Output: [password for bandit6]
    ```
    

**Evidence**: Save to `bandit5-specific-file.txt`

**🔑 Key Concept**: `find` is powerful for forensic file searches with complex criteria.

---

### Level 6: System-Wide File Search

**Connection**:

```bash
ssh bandit6@bandit.labs.overthewire.org -p 2220
```

**Objective**: Find file anywhere on the server with:

- Owned by user `bandit7`
- Owned by group `bandit6`
- 33 bytes in size

#### Challenge

- [ ] **Search entire filesystem**:
    
    ```bash
    find / -user bandit7 -group bandit6 -size 33c 2>/dev/null# Output: /var/lib/dpkg/info/bandit7.password
    ```
    

**Breakdown**:

- `/` = search from root directory
    
- `-user bandit7` = owner is bandit7
    
- `-group bandit6` = group is bandit6
    
- `-size 33c` = exactly 33 bytes
    
- `2>/dev/null` = hide "Permission denied" errors
    
- [ ] **Read the file**:
    
    ```bash
    cat /var/lib/dpkg/info/bandit7.password
    # Output: [password for bandit7]
    ```
    

**Evidence**: Save to `bandit6-system-wide-search.txt`

**🔑 Key Concept**: Use `2>/dev/null` to suppress error messages during forensic searches.

---

### Level 7: Grep for Specific String

**Connection**:

```bash
ssh bandit7@bandit.labs.overthewire.org -p 2220
```

**Objective**: Find the password next to the word "millionth" in a large text file.

#### Challenge

- [ ] **Check file size**:
    
    ```bash
    ls -lh data.txt
    # Output: -rw-r----- 1 bandit8 bandit7 4.0M ... data.txt
    ```
    
- [ ] **Search for string**:
    
    ```bash
    grep "millionth" data.txt
    # Output: millionth       [password for bandit8]
    ```
    

**Alternative (case-insensitive)**:

```bash
grep -i "millionth" data.txt
```

**Evidence**: Save to `bandit7-grep-word.txt`

**🔑 Key Concept**: `grep` searches for patterns in text files (essential for log analysis).

---

### Level 8: Find Unique Line

**Connection**:

```bash
ssh bandit8@bandit.labs.overthewire.org -p 2220
```

**Objective**: Find the line that appears only once in a file where all others repeat.

#### Challenge

- [ ] **Sort and count unique lines**:
    
    ```bash
    sort data.txt | uniq -u# Output: [password for bandit9]
    ```
    

**Breakdown**:

- `sort data.txt` = alphabetize lines (required for uniq to work)
- `uniq -u` = print only unique lines (appear once)

**Alternative (show counts)**:

```bash
sort data.txt | uniq -c | grep "^\s*1 "
# Shows lines with count of 1
```

**Evidence**: Save to `bandit8-unique-line.txt`

**🔑 Key Concept**: `uniq` detects duplicates (useful for finding anomalies in logs).

---

### Level 9: Extract Readable Strings from Binary

**Connection**:

```bash
ssh bandit9@bandit.labs.overthewire.org -p 2220
```

**Objective**: Extract human-readable strings from a binary file, find ones preceded by `=` signs.

#### Challenge

- [ ] **Check file type**:
    
    ```bash
    file data.txt
    # Output: data.txt: data
    ```
    
- [ ] **Extract readable strings**:
    
    ```bash
    strings data.txt | grep "==="
    # Output: 
    # ========== the
    # ========== password
    # ========== is
    # ========== [password for bandit10]
    ```
    

**Breakdown**:

- `strings` = extract printable characters from binary
- `grep "==="` = filter lines containing ===

**Evidence**: Save to `bandit9-strings-extraction.txt`

**🔑 Key Concept**: `strings` extracts text from binary files (malware analysis, forensics).

---

### Level 10: Base64 Decoding

**Connection**:

```bash
ssh bandit10@bandit.labs.overthewire.org -p 2220
```

**Objective**: Decode a Base64-encoded file.

#### Challenge

- [ ] **View encoded data**:
    
    ```bash
    cat data.txt
    # Output: VGhlIHBhc3N3b3JkIGlzIElGdWt3S0dzRlc4TU9xM0lSRnFyeEUxaHhUTkViVVBSCg==
    ```
    
- [ ] **Decode Base64**:
    
    ```bash
    base64 -d data.txt
    # Output: The password is [password for bandit11]
    ```
    

**Alternative (one-liner)**:

```bash
cat data.txt | base64 -d
```

**Evidence**: Save to `bandit10-base64-decode.txt`

**🔑 Key Concept**: Base64 encoding obfuscates data (common in malware C2 communication).

---

## Command Cheat Sheet

|Level|Key Command|Purpose|
|---|---|---|
|0|`cat readme`|Read file contents|
|1|`cat ./-`|Handle dashed filename|
|2|`cat "spaces in this filename"`|Handle spaces|
|3|`ls -a`|Show hidden files|
|4|`file ./*`|Identify file types|
|5|`find . -type f -size 1033c ! -executable`|Complex file search|
|6|`find / -user bandit7 -group bandit6 -size 33c 2>/dev/null`|System-wide search|
|7|`grep "millionth" data.txt`|Search file for string|
|8|`sort data.txt \| uniq -u`|Find unique line|
|9|`strings data.txt \| grep "==="`|Extract readable strings|
|10|`base64 -d data.txt`|Decode Base64|

---

## Lessons Learned

### Linux Forensics Skills Developed

**File Navigation**:

- Absolute paths start with `/` (e.g., `/var/lib/file`)
- Relative paths start with `.` (e.g., `./inhere/file`)
- Parent directory: `..` (e.g., `cd ..`)

**Special Characters**:

- **Spaces**: Escape with `\` or use quotes
- **Dashes**: Use `./` prefix
- **Dots (hidden)**: Use `ls -a`

**File Properties**:

- **Type**: `file [filename]` (ASCII text, data, binary)
- **Size**: `ls -lh` (human-readable) or `-size 1033c` in find
- **Permissions**: `ls -l` shows `-rwxr-xr-x`
- **Ownership**: `ls -l` shows user:group

**Search Techniques**:

- **By name**: `find . -name "*.txt"`
- **By size**: `find . -size +1M` (larger than 1MB)
- **By owner**: `find . -user bandit7`
- **By content**: `grep "password" file.txt`

### Security Analyst Applications

**Incident Response Use Cases**:

```bash
# Find recently modified files (attacker left evidence)
find / -type f -mtime -1 2>/dev/null

# Search logs for suspicious IP
grep "203.0.113.45" /var/log/auth.log

# Extract strings from suspicious binary
strings /tmp/malware.bin | grep -i "http"

# Find files owned by compromised user
find / -user compromised_account 2>/dev/null

# Identify Base64-encoded commands in logs
grep "base64" /var/log/syslog | base64 -d
```

### Time Tracking

|Level|Estimated|Actual|Notes|
|---|---|---|---|
|0|5 min|___ min||
|1|5 min|___ min||
|2|5 min|___ min||
|3|10 min|___ min||
|4|10 min|___ min||
|5|15 min|___ min||
|6|15 min|___ min||
|7|5 min|___ min||
|8|10 min|___ min||
|9|5 min|___ min||
|10|5 min|___ min||
|**Total**|**90 min**|**___ min**||

---

## Exam Relevance Notes

### Security+ (SY0-701) Mapping

**Objective 4.4: Data sources for investigations**

- Log files (grep, tail, head)
- File system forensics (find, strings)

**Objective 4.5: Incident response**

- Containment: Identify compromised files
- Eradication: Find malicious binaries
- Recovery: Restore from known-good state

### Linux+ (XK0-006) Mapping

**Objective 1.1: Linux fundamentals**

- File system hierarchy (`/var`, `/etc`, `/home`)
- Hidden files (dot files)

**Objective 1.3: File system navigation**

- `ls`, `cd`, `pwd`, `cat`
- Absolute vs. relative paths

---

## Submission Checklist

Before moving to Levels 11-20, verify:

- [ ] All 12 evidence files documented
- [ ] Passwords saved for Levels 0-11
- [ ] Command cheat sheet completed
- [ ] Lessons learned summary written
- [ ] Time tracking updated

**Final Evidence Count**: 12 files  
**Passwords Collected**: 11 passwords (bandit0 → bandit11)  
**Ready for Next Lab**: Yes / No

---

## Next Steps

After completing Levels 0-10:

1. **Proceed to Bandit Lab 2** (Levels 11-20): File permissions, scripting, networking
2. **Practice in real environments**: Apply commands to your Linux VMs
3. **Study for Security+**: Review incident response frameworks
4. **Optional Challenge**: Complete Bandit Level 11 using ROT13 decoding

**Additional Resources**:

- [OverTheWire Bandit](https://overthewire.org/wargames/bandit/)
- [Linux Command Line Basics](https://www.linuxcommand.org/)
- [SANS Linux CLI Cheat Sheet](https://www.sans.org/posters/linux-command-line/)