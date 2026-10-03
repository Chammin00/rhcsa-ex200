# RHCSA EX200 Study Kit — RHEL 9 & RHEL 10

An interactive browser-based practice tool for the **Red Hat Certified System Administrator (RHCSA) EX200** exam.

## Live Tool

**[Open the Practice Terminal](https://Chammin00.github.io/rhcsa-ex200/RHCSA_Terminal_Practice.html)**

No install needed — runs entirely in your browser with real-time command evaluation and simulated terminal responses.

---

## 🎯 Featured: codeXchange RHCSA V10 Complete Exam (22 Tasks)
*Directly based on the real-world exam simulation from [codeXchange YouTube](https://www.youtube.com/watch?v=kBsMuA-DA9E)*

| Document | Description |
|---|---|
| **[RHCSA_V10_Complete_Mock_Exam.md](RHCSA_V10_Complete_Mock_Exam.md)** | Full 22-task exam specification with exact prompts, network architecture, and host contexts |
| **[RHCSA_V10_Complete_Exam_Solutions.md](RHCSA_V10_Complete_Exam_Solutions.md)** | Step-by-step solution manual with terminal commands, syntax breakdowns, and verification steps |

### V10 Exam Task Breakdown (22 Tasks)

| Task | Context | Topic | Description |
|:---:|:---:|:---|:---|
| **1** | General | Pre-flight Connectivity | Verify subnet `172.25.250.0/24`, gateway, and node reachability |
| **2** | `node1` | Network Configuration | Configure static IP `172.25.250.11/24`, gateway, DNS, and hostname |
| **3** | `node1` | YUM/DNF Repositories | BaseOS and AppStream repo configuration (`content.example.com`) |
| **4** | `node1` | SELinux & Non-Standard Port | HTTP service on port 82, `semanage port`, firewall, and curl verification |
| **5** | `node1` | User & Group Management | `sysadmin` group, secondary group membership, nologin shell for `sarah` |
| **6** | `node1` | Collaborative Permissions | SGID collaborative directory `/common/admin` (`2770`) for `sysadmin` |
| **7** | `node1` | Cron Scheduling | Daily cron for `natasha` (14:23) and 3-minute logger cron for `harry` |
| **8** | `node1` | Archiving & Compression | Create `/home/backup.tar.bz2` archive of `/etc` |
| **9** | `node1` | NTP Client | Configure chrony client to sync with `classroom.example.com` |
| **10** | `node1` | File Search & Copy | Find all files owned by user `simone` and copy to `/root/found` |
| **11** | `node1` | String Search | Extract `strato` from `/usr/share/dict/words` to `/searchfile` |
| **12** | `node1` | Autofs NFS Automount | Automount NFS home directory for `remoteuser2` from `workstation` |
| **13** | `node1` | Specific UID User | Create user `barry` with UID `2112` and password `atenorth` |
| **14** | `node1` | Scripting & Security | Search script `newsearch` (SUID 30k-50k), `alex` umask (0444), passwordless sudo `elite`, `PASS_MAX_DAYS 20`, app rebuild `ex200` |
| **15** | `node1` | Systemd Timer & ACLs | Script with 1-min systemd timer (`logger.timer`), POSIX ACLs on `/var/tmp/fstab` |
| **16** | `node2` | Root Password Break | GRUB emergency mode (`rd.break`), `/sysroot` remount, reset root to `atenorth`, SELinux autorelabel |
| **17** | `node2` | YUM/DNF Repositories | Configure BaseOS and AppStream repositories on secondary machine |
| **18** | `node2` | Tuned Profile | Identify and apply system recommended tuning profile (`tuned-adm`) |
| **19** | `node2` | Flatpak Management | User-scoped Flatpak repository `extra` and app installation (`codium`) for user `bammbamm` |
| **20** | `node2` | Swap Partition | Create 250 MB persistent swap partition with `/etc/fstab` entry |
| **21** | `node2` | LVM Storage & VFAT | Volume Group `myvol` (8M PE), LV `mydatabase` (50 extents = 400M), `vfat` formatted and mounted on `/database` |
| **22** | `node2` | Logical Volume Resize | Resize LV `home` in VG `myvg` to 152 MB (38 extents), filesystem online resize |

---

## Complete Study Kit Contents

| File | Description |
|---|---|
| `RHCSA_Terminal_Practice.html` | Main interactive web tool — terminal, cheat sheet, drills, 2 exam suites |
| `RHCSA_V10_Complete_Mock_Exam.md` | **NEW:** 22-task codeXchange V10 exam specification |
| `RHCSA_V10_Complete_Exam_Solutions.md` | **NEW:** Comprehensive step-by-step solution manual for all 22 tasks |
| `RHCSA_Mock_Exam.md` | 18-task 300-point standard mock exam paper |
| `RHCSA_Mock_Exam_Answer_Key.md` | Full solutions & grading rubrics for standard exam |
| `RHCSA_Practice_Scenarios.md` | 20 targeted hands-on practice scenarios |
| `RHCSA_EX200_Study_Guide.md` | Deep-dive guide: NFS, SELinux, Storage, Systemd, Scripting |
| `RHCSA_Lab_Setup.md` | VMware Workstation + RHEL two-VM local lab setup guide |

---

## 5 Interactive Modes in the Web Tool

1. **Cheat Sheet** — 100+ commands grouped by topic with instant click-to-paste into the active terminal.
2. **Drill Mode** — Flashcard quizzes with memory validation and immediate feedback.
3. **Practice Exam (Dual Suite)** — Integrated live terminal with simulated RHEL output, task switcher dropdown (**codeXchange V10 22-Task Exam** or **Standard 18-Task Exam**), target node badges (`node1` vs `node2`), and instant answer keys.
4. **Mock Exam Tracker** — Dual-mode countdown timer and grading scorecard to use alongside your actual VMware/KVM lab.
5. **Essential Tools Deep-Dive** — Interactive reference guides for `find`, `grep`, and `Bash Scripting` with 35+ targeted drills.
