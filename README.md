# RHCSA EX200 Study Kit - RHEL 9

An interactive browser-based practice tool for the **Red Hat Certified System Administrator (RHCSA) EX200** exam on **RHEL 9**.

## Live Tool

**[Open the Practice Terminal](https://Chammin00.github.io/rhcsa-ex200/RHCSA_Terminal_Practice.html)**

No install needed - runs entirely in your browser.

## Contents

| File | Description |
|------|-------------|
| `RHCSA_Terminal_Practice.html` | Main tool - interactive terminal + 4-tab study interface |
| `RHCSA_EX200_Study_Guide.md` | Full study guide: NFS, SELinux, Containers, Scripting |
| `RHCSA_Practice_Scenarios.md` | 20 practice scenarios with solutions |
| `RHCSA_Mock_Exam.md` | 18-task 300-point mock exam paper |
| `RHCSA_Mock_Exam_Answer_Key.md` | Full solutions + grading rubrics |
| `RHCSA_Lab_Setup.md` | VMware + RHEL 9 two-VM lab setup guide |

## 4 Study Modes

**Cheat Sheet** - 100+ commands grouped by topic. Click to paste. Includes Required Installs per topic.

**Drill Mode** - Flashcard-style quizzes. Type the exact command from memory. Scored.

**Practice Exam** - Full 3-hour timed exam with all 18 tasks + integrated terminal:
- Task description with dnf install requirements shown at load
- Live terminal - type commands, get simulated RHEL 9 output
- Key commands highlighted for each task
- Mark Complete to score and advance
- Sidebar task list with status dots

**Mock Exam Tracker** - Use alongside your real VM. 3-hour timer, mark tasks done, track score.

## Exam Coverage

| Task | Topic | Points | Install Needed |
|------|-------|--------|----------------|
| 1 | Networking and Hostname | 10 | none |
| 2 | User and Group Management | 15 | none |
| 3 | File Permissions and Special Bits | 15 | none |
| 4 | File Search and Archive | 10 | none |
| 5 | Software Management | 10 | none |
| 6 | Systemd Service Unit | 10 | none |
| 7 | Cron and at Scheduling | 10 | dnf install at cronie -y |
| 8 | sudo Configuration | 10 | none |
| 9 | LVM Storage | 20 | dnf install lvm2 -y |
| 10 | NFS Server | 20 | dnf install nfs-utils -y |
| 11 | NFS Client and AutoFS | 20 | dnf install autofs nfs-utils -y |
| 12 | Shell Script Health Report | 25 | none |
| 13 | Shell Script CSV Provisioning | 20 | none |
| 14 | Containers Run and Manage | 20 | dnf install container-tools -y |
| 15 | Containers Build Image | 20 | dnf install container-tools -y |
| 16 | Containers Systemd Service | 20 | dnf install container-tools -y |
| 17 | SELinux Contexts and Booleans | 20 | dnf install httpd policycoreutils-python-utils -y |
| 18 | SELinux Audit and Remediation | 25 | dnf install policycoreutils-python-utils -y |
| | **Total** | **300** | Pass = 210 (70%) |

## Quick Start Local

Open RHCSA_Terminal_Practice.html in any modern browser. No server or install needed.

## Lab Setup

VMware Workstation with 2 RHEL 9 VMs. Local DNF repo from ISO - no subscription needed.
See RHCSA_Lab_Setup.md for full details.
