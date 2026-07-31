# How to Use the Universal Cybersecurity Lab Generator

Welcome to the **Universal Cybersecurity Lab Generator**. This framework is designed to help students, self-taught practitioners, IT professionals, and security enthusiasts transform raw tutorials, vendor walkthroughs, or custom topic ideas into portfolio-grade lab documentation ready for GitHub publication.

---

## 📖 Quick Start Guide

### Step 1: Prime Your AI Model
1. Copy the entire contents of `master-lab-generator-prompt.md`.
2. Paste it into your AI assistant (Claude, ChatGPT, Gemini, etc.) as the initial message, custom instructions, or system prompt.
3. The model will adopt the persona of an **Elite Lab Documentation Engineer**.

---

### Step 2: Generate a Lab

You can generate labs using two flexible methods:

#### Method A: The Raw Dump (Turn Tutorial Text into Portfolio Documentation)
Paste raw content from Skillsoft, Pluralsight, YouTube transcripts, or vendor docs.

**Example Prompt:**
> "Transform this raw lab dump into a structured lab file:  
> [Paste your unformatted lab text here]"

#### Method B: Plug-and-Play Topic (Build a Custom Lab from Scratch)
Don't have a guide? Specify a topic, target certification, and preferred tools.

**Example Prompt:**
> "Create an Intermediate-level lab on configuring Snort IDS and analyzing alert logs for CompTIA CySA+ (CS0-003).  
> Environment: VMware Workstation with Ubuntu Server and Kali Linux."

---

### Step 3: Execute the Lab & Gather Evidence

Once the AI produces your structured `.md` file:

1. **Save the File:** Store it in your local vault or lab directory (e.g., `labs/intermediate/lab-snort-ids.md`).
2. **Build Your Environment:** Follow Phase 0 requirements for your hypervisor (VMware, VirtualBox, Hyper-V, AWS, etc.).
3. **Set Up Evidence Directories:**
   ```bash
   mkdir -p ~/evidence/lab-[name]
   ```
4. **Work Through Tasks:** Follow the instructions step-by-step. Mark tasks off as you complete them: `- [ ] Complete`.
5. **Record Evidence:** Save screenshots and CLI log exports in your evidence directory using the required naming scheme (`01-description.png`, `02-terminal-history.txt`).

---

### Step 4: Publish to Your GitHub Portfolio

Organize your GitHub repository to showcase your practical experience to recruiters and hiring managers.

#### Recommended Directory Structure
```text
labs/
├── README.md
├── foundation/
│   ├── lab-linux-basics.md
│   └── lab-windows-perf.md
├── intermediate/
│   ├── lab-snort-ids.md
│   └── lab-network-troubleshooting.md
├── advanced/
│   └── lab-security-onion.md
└── evidence/
    ├── lab-linux-basics/
    ├── lab-snort-ids/
    └── lab-security-onion/
```

#### Portfolio Indexing (`README.md`)
Maintain a master `README.md` at the root of your repo categorized by **Difficulty**, **Certification Mapping**, and **Technology Stack**.

---

## 💡 Best Practices for Portfolio Building

* **Document Failure Modes:** Real-world engineering requires troubleshooting. Review the "Observations & Failure Modes" section and document any unexpected issues you solved.
* **Emphasize Hardening:** Pay close attention to the "Security Considerations" section. Show recruiters you understand not just how to make something work, but how to secure it.
* **Keep Evidence Verifiable:** Clear terminal logs and high-resolution screenshots demonstrate authentic hands-on competence.
