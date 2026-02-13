---
title: "Software-Defined Networking: Ansible and Git for Network Automation"
difficulty: Intermediate → Advanced
Cert:
  - Network+
  - NT10-009
Exam-Objectives:
  - Net+ 1.7
  - Net+ 3.5
created: 2026-02-13
completed:
time-hours: 4
complete: false
GitHub-ready: false
evidence-files: 13
notes:
tags:
  - sdn
  - ansible
  - git
  - automation
  - infrastructure-as-code
  - IaC
  - pfsense
  - version-control
---
---
## Objective Mapping

**1.7 Explain basic corporate and datacenter network architecture**

- Software-defined networking (SDN)
- Automation/orchestration

**3.5 Compare and contrast network access and management methods**

- Connection methods (SSH for automation)

## Scenario

You're a network automation engineer at a cloud service provider managing 200+ pfSense firewall instances. Manual configuration changes are error-prone and don't scale. Leadership has mandated **Infrastructure as Code (IaC)** practices using:

1. **Ansible** for automated configuration management (deploy DNS server changes across all firewalls)
2. **Git** for version control (track all network configuration changes)

Your task: **Demonstrate automated pfSense configuration** using an Ansible playbook and **version control** using Git to track changes. This lab simulates modern **DevOps/NetOps workflows** critical for SOC analysts supporting cloud infrastructure.

## Learning Objectives

- Install and configure Ansible on Linux
- Use third-party Ansible modules (pfsense)
- Establish SSH connections from Ansible to managed nodes
- Deploy Ansible playbooks to configure network devices
- Initialize Git repositories for version control
- Track configuration file changes with Git commits
- Manage version history with Git checkout and log

## Required Environment

### Hardware/Software

- **VMs**: ACIALMA (Alma Linux 9.3), ACIDC01 (Windows Server 2022), ACIWIN11 (Windows 11), ACIPFSENSE (pfSense 2.7.2)
- **Tools**: Ansible, Git, nano text editor, SSH, Python pip

### Host System

- **OS**: Windows 11 Pro or Windows Server 2022
- **RAM**: 12GB minimum
- **Network**: Internal virtual network with SSH access

## Deliverables

### Evidence Files (Required)

1. `01-alma-ansible-installation.txt` – `pip install ansible` output
2. `02-pfsense-ansible-module-installation.txt` – `ansible-galaxy collection install` output
3. `03-ssh-keygen-output.txt` – SSH key generation for passwordless auth
4. `04-ssh-copy-id-to-pfsense.txt` – SSH key copied to pfSense
5. `05-ansible-inventory-file.txt` – Inventory listing pfSense host
6. `06-pfsense-update-dns-playbook.yml` – Ansible playbook YAML file
7. `07-ansible-playbook-execution.txt` – Playbook run output
8. `08-git-init-repository.txt` – Git repository initialization
9. `09-git-status-before-commit.txt` – Untracked files shown
10. `10-git-commit-initial.txt` – First commit message
11. `11-git-log-history.txt` – Commit history showing initial and modified commits
12. `12-git-checkout-file.txt` – Restoring file from Git repository
13. `13-git-commit-modified.txt` – Second commit after DNS server change

### Documentation (Required)

- **Ansible playbook explanation**: Breakdown of each task in YAML
- **Git workflow diagram**: Staging → Commit → Push workflow
- **SSH key management notes**: Security implications of passwordless auth

## Lab Tasks

### Phase 1: Ansible Installation and Configuration (70 min)

#### Task 1.1: Install Ansible on Alma Linux

- [ ] Connect to ACIALMA → Open Terminal
- [ ] Install Ansible via pip:
    
    ```bash
    sudo dnf install python3-pip -ypip3 install ansible --break-system-packagesansible --version > ~/01-alma-ansible-installation.txtcat ~/01-alma-ansible-installation.txt
    ```
    
- [ ] **Evidence**: `01-alma-ansible-installation.txt`

#### Task 1.2: Install pfSense Ansible Module

- [ ] Install third-party collection:
    
    ```bash
    ansible-galaxy collection install pfsensible.core > ~/02-pfsense-ansible-module-installation.txt 2>&1cat ~/02-pfsense-ansible-module-installation.txt
    ```
    
- [ ] **Evidence**: `02-pfsense-ansible-module-installation.txt`

#### Task 1.3: Configure SSH Passwordless Authentication

- [ ] Generate SSH key pair:
    
    ```bash
    ssh-keygen -t rsa -b 4096 -C "ansible@acialma" -f ~/.ssh/id_rsa -N ""
    cat ~/.ssh/id_rsa.pub > ~/03-ssh-keygen-output.txt
    ```
    
- [ ] **Evidence**: `03-ssh-keygen-output.txt`
    
- [ ] Copy public key to pfSense:
    
    ```bash
    ssh-copy-id -i ~/.ssh/id_rsa.pub admin@192.168.0.5
    # Enter pfSense password: Passw0rd
    
    # Verify passwordless login
    ssh admin@192.168.0.5 "uname -a" > ~/04-ssh-copy-id-to-pfsense.txt
    cat ~/04-ssh-copy-id-to-pfsense.txt
    ```
    
- [ ] **Evidence**: `04-ssh-copy-id-to-pfsense.txt`
    

#### Task 1.4: Create Ansible Inventory

- [ ] Create inventory file:
    
    ```bash
    nano ~/inventory.ini
    ```
    
    Content:
    
    ```ini
    [pfsense_firewalls]
    pfsense01 ansible_host=192.168.0.5 ansible_user=admin ansible_python_interpreter=/usr/local/bin/python3.11
    
    [pfsense_firewalls:vars]
    ansible_connection=ssh
    ansible_ssh_common_args='-o StrictHostKeyChecking=no'
    ```
    
- [ ] Save and export:
    
    ```bash
    cat ~/inventory.ini > ~/05-ansible-inventory-file.txt
    ```
    
- [ ] **Evidence**: `05-ansible-inventory-file.txt`
    

#### Task 1.5: Create Ansible Playbook

- [ ] Create playbook:
    
    ```bash
    nano ~/pfsense_update_dns_server.yml
    ```
    
    Content:
    
    ```yaml
    ---
    - name: Update pfSense DNS Server Configuration
      hosts: pfsense_firewalls
      gather_facts: no
      
      tasks:
        - name: Configure DNS servers
          pfsensible.core.pfsense_setup:
            dns_addresses:
              - 8.8.8.8
              - 8.8.4.4
            dns_hostnames:
              - google-dns1
              - google-dns2
          register: dns_result
        
        - name: Display DNS configuration result
          debug:
            var: dns_result
    ```
    
- [ ] Save and export:
    
    ```bash
    cp ~/pfsense_update_dns_server.yml ~/06-pfsense-update-dns-playbook.yml
    ```
    
- [ ] **Evidence**: `06-pfsense-update-dns-playbook.yml`
    

#### Task 1.6: Execute Ansible Playbook

- [ ] Run playbook:
    
    ```bash
    ansible-playbook -i ~/inventory.ini ~/pfsense_update_dns_server.yml > ~/07-ansible-playbook-execution.txt 2>&1cat ~/07-ansible-playbook-execution.txt
    ```
    
- [ ] Verify output shows:
    
    ```
    PLAY RECAP ***********************pfsense01 : ok=2 changed=1 unreachable=0 failed=0
    ```
    
- [ ] **Evidence**: `07-ansible-playbook-execution.txt`

---

### Phase 2: Git Version Control (50 min)

#### Task 2.1: Initialize Git Repository

- [ ] Create Git repo:
    
    ```bash
    mkdir ~/network-configscd ~/network-configsgit init > ~/08-git-init-repository.txt 2>&1cat ~/08-git-init-repository.txt
    ```
    
- [ ] **Evidence**: `08-git-init-repository.txt`

#### Task 2.2: Stage and Commit Initial Configuration

- [ ] Copy playbook to Git repo:
    
    ```bash
    cp ~/pfsense_update_dns_server.yml ~/network-configs/
    cd ~/network-configs
    git status > ~/09-git-status-before-commit.txt
    cat ~/09-git-status-before-commit.txt
    ```
    
- [ ] **Evidence**: `09-git-status-before-commit.txt`
    
- [ ] Add and commit:
    
    ```bash
    git add pfsense_update_dns_server.yml
    git commit -m "Initial commit: Add pfsense_update_dns_server.yml playbook" > ~/10-git-commit-initial.txt 2>&1
    cat ~/10-git-commit-initial.txt
    ```
    
- [ ] **Evidence**: `10-git-commit-initial.txt`
    

#### Task 2.3: Modify Configuration and Track Changes

- [ ] Delete local file (simulate accidental deletion):
    
    ```bash
    rm pfsense_update_dns_server.yml
    ls
    # File should be gone
    ```
    
- [ ] Restore from Git:
    
    ```bash
    git checkout pfsense_update_dns_server.yml > ~/12-git-checkout-file.txt 2>&1
    ls
    cat ~/12-git-checkout-file.txt
    ```
    
- [ ] **Evidence**: `12-git-checkout-file.txt`
    
- [ ] Modify DNS servers:
    
    ```bash
    nano pfsense_update_dns_server.yml
    # Change dns_addresses to:
    #   - 192.168.255.13
    ```
    
- [ ] Commit modification:
    
    ```bash
    git add pfsense_update_dns_server.yml
    git commit -m "Modified pfsense_update_dns_server.yml: Changed DNS server address" > ~/13-git-commit-modified.txt 2>&1
    cat ~/13-git-commit-modified.txt
    ```
    
- [ ] **Evidence**: `13-git-commit-modified.txt`
    

#### Task 2.4: View Commit History

- [ ] Display Git log:
    
    ```bash
    git log > ~/11-git-log-history.txtcat ~/11-git-log-history.txt
    ```
    
- [ ] Verify output shows both commits:
    
    ```
    commit [hash2] (HEAD -> master)Author: ...Date: ...Modified pfsense_update_dns_server.yml: Changed DNS server addresscommit [hash1]Author: ...Date: ...Initial commit: Add pfsense_update_dns_server.yml playbook
    ```
    
- [ ] **Evidence**: `11-git-log-history.txt`

---

### Phase 3: Validation and Cleanup (60 min)

#### Task 3.1: Verify pfSense Configuration Changed

- [ ] On ACIWIN11 → Open browser → Navigate to `http://192.168.0.5`
- [ ] Login to pfSense → **System → General Setup**
- [ ] Verify DNS servers show:
    
    ```
    DNS Server 1: 192.168.255.13
    ```
    
- [ ] **Screenshot** (optional): pfSense DNS configuration page

#### Task 3.2: Test Idempotency

- [ ] Rerun Ansible playbook:
    
    ```bash
    ansible-playbook -i ~/inventory.ini ~/network-configs/pfsense_update_dns_server.yml
    ```
    
- [ ] Verify output shows:
    
    ```
    changed=0
    ```
    
    (No changes made because configuration already matches)

---

## Observations & Failure Modes

### Issue 1: Ansible Module Not Found

**Cause**: pfsensible.core collection not installed  
**Solution**:

```bash
ansible-galaxy collection install pfsensible.core --force
```

### Issue 2: SSH Connection Refused

**Cause**: pfSense SSH service not enabled  
**Solution**:

- pfSense WebGUI → **System → Advanced → Secure Shell**
- Check "Enable Secure Shell"
- Save

### Issue 3: Git Commit Shows "Nothing to Commit"

**Cause**: File not staged with `git add`  
**Solution**:

```bash
git add [filename]
git commit -m "message"
```

### Issue 4: Ansible Playbook Syntax Error

**Cause**: YAML indentation incorrect (use 2 spaces, not tabs)  
**Solution**:

```bash
# Validate YAML syntax
ansible-playbook --syntax-check ~/pfsense_update_dns_server.yml
```

---

## Analysis & Reflection

### Infrastructure as Code Benefits

**Advantages**:

- **Consistency**: Same playbook deploys identical configs across 200 firewalls
- **Auditability**: Git log shows who changed what and when
- **Rollback**: `git revert` undoes bad changes
- **Testing**: Run playbooks in dev before prod

**Real-World Example**:

```yaml
# Single playbook updates firewall rules across entire fleet
- name: Block malicious IP range
  hosts: all_firewalls
  tasks:
    - name: Add blocking rule
      pfsensible.core.pfsense_rule:
        name: "Block Malware C2"
        source: "203.0.113.0/24"
        action: block
```

### Git Workflow for Network Configs

```
Developer Workstation:
  git clone → Edit configs → git commit → git push

Central Repository (GitHub/GitLab):
  Code review → Merge → CI/CD pipeline

Automation Server:
  git pull → ansible-playbook → Deploy to production
```

### Security Considerations

**SSH Key Management**:

- **Risk**: Compromised private key = full network access
- **Mitigation**:
    - Use passphrase-protected keys
    - Rotate keys quarterly
    - Store in secrets manager (HashiCorp Vault)

**Ansible Vault**:

```yaml
# Encrypt sensitive variables
ansible-vault encrypt inventory.ini
ansible-playbook --ask-vault-pass playbook.yml
```

### Real-World Applications

**SOC Analyst Use Case**:

- Ansible playbooks deploy security patches across firewall fleet
- Git log provides audit trail for compliance (HIPAA, PCI-DSS)
- Monitor for unauthorized Ansible executions (SIEM correlation)

**Network Engineer Use Case**:

- Automate VLAN provisioning across 100+ switches
- Version control for firewall rulesets (compare current vs. last known good)
- CI/CD pipeline tests configs in lab before prod deployment

### Time Tracking

|Phase|Estimated|Actual|
|---|---|---|
|Phase 1: Ansible|70 min|___ min|
|Phase 2: Git|50 min|___ min|
|Phase 3: Validation|60 min|___ min|
|**Total**|**180 min**|**___ min**|

---

## Exam Relevance Notes

**N10-009 Key Concepts**:

- **Software-Defined Networking (SDN)**: Separates control plane from data plane
    - Control Plane: Makes decisions (where packets go)
    - Data Plane: Forwards packets based on decisions
- **Automation Tools**: Ansible, Puppet, Chef, SaltStack
- **Version Control**: Git, SVN, Mercurial

**PBQ Scenarios**:

- "Explain how SDN improves network agility" → Answer: Centralized policy management
- "Which tool automates network device configuration?" → Answer: Ansible

**Multiple Choice Patterns**:

- "What is the primary benefit of Infrastructure as Code?" → Consistency/Repeatability
- "Git is used for:" → Version control of configuration files

---

## Submission Checklist

- [ ] All 13 evidence files collected
- [ ] Ansible playbook explanation documented
- [ ] Git workflow diagram created
- [ ] SSH key management notes included
- [ ] Time tracking updated

**Final Evidence Count**: 13 files  
**Ready for GitHub**: Yes / No

---

## Next Steps

1. **Review**: Study SDN controllers (OpenDaylight, ONOS)
2. **Practice**: Create Ansible role for multi-vendor network devices (Cisco, Juniper)
3. **Advance**: Integrate Git with CI/CD pipeline (GitLab CI, Jenkins)
4. **Challenge**: Use Terraform for network infrastructure provisioning (AWS VPC, Azure VNet)

**Additional Resources**:

- [Ansible Network Automation Guide](https://docs.ansible.com/ansible/latest/network/index.html)
- [Git Official Documentation](https://git-scm.com/doc)
- CompTIA Network+ N10-009 Official Study Guide, Chapter 1.7 (SDN/Automation)