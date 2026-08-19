# RHCSA Lab Setup – VMware Workstation + RHEL 9
### Get Your Two-VM Exam Environment Ready in ~45 Minutes

---

## What You'll Build

```
┌──────────────────────────────────────────────────────────┐
│                   VMware Workstation                      │
│                                                          │
│  ┌─────────────────────┐    ┌──────────────────────────┐ │
│  │      server1        │    │        client1           │ │
│  │  192.168.1.10/24   │◄──►│    192.168.1.20/24      │ │
│  │  RHEL 9             │    │    RHEL 9                │ │
│  │  2 vCPU / 2GB RAM  │    │  1 vCPU / 1GB RAM        │ │
│  │  20GB disk          │    │  20GB disk               │ │
│  └─────────────────────┘    └──────────────────────────┘ │
│                                                          │
│              VMnet (Host-Only or NAT)                    │
└──────────────────────────────────────────────────────────┘
```

---

## Step 1 — Configure VMware Network

> Do this ONCE before creating any VMs.

1. Open VMware Workstation → **Edit → Virtual Network Editor**
2. Click **"Change Settings"** (Run as Administrator)
3. Select **VMnet1** (Host-only) or create a new host-only network
4. Set subnet: `192.168.1.0` / Subnet mask: `255.255.255.0`
5. **Uncheck** "Use local DHCP service" (we'll use static IPs)
6. Click **Apply → OK**

> **Tip**: Using Host-Only keeps your VMs isolated from your real network — ideal for exam lab.

---

## Step 2 — Create server1 VM

### VM Settings
| Setting          | Value                            |
|------------------|----------------------------------|
| VM Name          | `server1`                        |
| Guest OS         | Red Hat Enterprise Linux 9 64-bit|
| CPUs             | 2 vCPUs                          |
| RAM              | **2048 MB** (2 GB minimum)       |
| Hard Disk 1      | **20 GB** (SCSI, thin provision) |
| Hard Disk 2      | **5 GB** (for LVM practice)      |
| Network Adapter  | **Host-Only (VMnet1)**           |
| CD/DVD           | Point to your RHEL 9 ISO         |

### How to Create
```
1. File → New Virtual Machine → Custom (Advanced)
2. Hardware compatibility: Workstation 17.x (or your version)
3. Installer disc image: Browse to your RHEL 9 .iso
4. Guest OS: Linux → Red Hat Enterprise Linux 9 64-bit
5. VM Name: server1
6. CPUs: 2 processors, 1 core each
7. Memory: 2048 MB
8. Network: Host-only
9. Disk: SCSI, 20 GB, "Store as single file"
10. Finish
```

### Add the 2nd disk for LVM (Task 09)
```
1. VM Settings → Add → Hard Disk
2. SCSI, 5 GB, thin
3. This will appear as /dev/sdb inside the VM
```

---

## Step 3 — Create client1 VM

### VM Settings
| Setting          | Value                            |
|------------------|----------------------------------|
| VM Name          | `client1`                        |
| Guest OS         | Red Hat Enterprise Linux 9 64-bit|
| CPUs             | 1 vCPU                           |
| RAM              | **1024 MB** (1 GB)               |
| Hard Disk        | **20 GB**                        |
| Network Adapter  | **Host-Only (VMnet1)**           |
| CD/DVD           | Point to your RHEL 9 ISO         |

> **Shortcut**: Clone `server1` after install, then change hostname/IP on the clone.

---

## Step 4 — Install RHEL 9 (Both VMs)

### Installation Destination Settings
Use these choices during the graphical installer:

| Option                  | Choose                                    |
|-------------------------|-------------------------------------------|
| Language                | English (United States)                   |
| Keyboard                | English (US)                              |
| Time Zone               | Your local timezone                       |
| Software Selection      | **Minimal Install** (exam uses CLI only)  |
| Installation Destination| Automatic (let it partition)              |
| Root Password           | `redhat`                                  |
| Create User             | Optional — you'll create users in practice|
| Network                 | Leave OFF for now (configure post-install)|

> **Important**: Select **Minimal Install** — the real exam environment is minimal. This trains you to install what you need via `dnf`.

---

## Step 5 — Post-Install: Configure server1

Boot into server1 and log in as `root`.

### 5.1 — Set Hostname
```bash
hostnamectl set-hostname server1.example.com
hostnamectl                          # Verify
```

### 5.2 — Configure Static IP
```bash
# List network interfaces
nmcli con show
ip link show
# Note your interface name (likely: ens33 or ens160 in VMware)

# Set static IP (replace ens33 with your interface name)
nmcli con mod ens33 \
    ipv4.method manual \
    ipv4.addresses 192.168.1.10/24 \
    ipv4.gateway 192.168.1.1 \
    ipv4.dns 8.8.8.8 \
    connection.autoconnect yes

nmcli con down ens33 && nmcli con up ens33

# Verify
ip addr show ens33
ping -c 2 8.8.8.8
```

### 5.3 — Register RHEL (Option A: Red Hat Subscription)
```bash
# If you have a free Red Hat Developer account:
subscription-manager register --username=<your_email> --password=<password>
subscription-manager attach --auto
dnf repolist                        # Should show repos
```

### 5.3 — Alternative (Option B: No Subscription — Use ISO as Repo)
```bash
# Mount the ISO
mkdir -p /mnt/rhel9iso

# In VMware: VM Settings → CD/DVD → Connected to ISO
mount /dev/cdrom /mnt/rhel9iso

# Create a local repo
cat > /etc/yum.repos.d/local.repo << 'EOF'
[BaseOS]
name=RHEL 9 BaseOS
baseurl=file:///mnt/rhel9iso/BaseOS
enabled=1
gpgcheck=0

[AppStream]
name=RHEL 9 AppStream
baseurl=file:///mnt/rhel9iso/AppStream
enabled=1
gpgcheck=0
EOF

dnf clean all
dnf repolist                        # Should show BaseOS and AppStream
```

> **Note**: The ISO repo covers ~90% of exam packages. A developer subscription gives full access.

### 5.4 — Install Exam-Relevant Packages
```bash
dnf install -y \
    bash-completion \
    nfs-utils \
    autofs \
    httpd \
    vim \
    wget \
    curl \
    tar \
    policycoreutils-python-utils \
    setroubleshoot-server \
    container-tools \
    lvm2 \
    tmux
```

### 5.5 — Configure /etc/hosts (for name resolution between VMs)
```bash
cat >> /etc/hosts << 'EOF'
192.168.1.10  server1.example.com  server1
192.168.1.20  client1.example.com  client1
EOF
```

### 5.6 — Disable Annoying MOTD / Subscription Warnings
```bash
# Only if not subscribed and you keep seeing warnings:
sed -i 's/^enabled=1/enabled=0/' /etc/yum/pluginconf.d/subscription-manager.conf
```

### 5.7 — Take a Snapshot! 📸
```
In VMware: VM → Snapshot → Take Snapshot
Name: "Fresh Install - server1"
```
> This lets you reset to a clean state between practice runs!

---

## Step 6 — Post-Install: Configure client1

```bash
# Same steps but with client1's values:
hostnamectl set-hostname client1.example.com

nmcli con mod ens33 \
    ipv4.method manual \
    ipv4.addresses 192.168.1.20/24 \
    ipv4.gateway 192.168.1.1 \
    ipv4.dns 8.8.8.8 \
    connection.autoconnect yes

nmcli con down ens33 && nmcli con up ens33

dnf install -y nfs-utils autofs curl vim

cat >> /etc/hosts << 'EOF'
192.168.1.10  server1.example.com  server1
192.168.1.20  client1.example.com  client1
EOF

# Snapshot!
# VMware: VM → Snapshot → Take Snapshot → "Fresh Install - client1"
```

---

## Step 7 — Verify Both VMs Can Talk to Each Other

**On server1:**
```bash
ping -c 3 192.168.1.20             # Ping client1
ping -c 3 client1.example.com      # By hostname
```

**On client1:**
```bash
ping -c 3 192.168.1.10             # Ping server1
ping -c 3 server1.example.com      # By hostname
ssh root@server1.example.com       # Test SSH
```

---

## Lab Readiness Checklist

Before starting any practice exam, confirm:

```
[ ] server1 has hostname server1.example.com
[ ] server1 has IP 192.168.1.10/24
[ ] client1 has hostname client1.example.com
[ ] client1 has IP 192.168.1.20/24
[ ] Both VMs can ping each other by hostname
[ ] dnf repolist shows repos on both VMs
[ ] SELinux is Enforcing on both (getenforce = Enforcing)
[ ] firewalld is running on both (systemctl status firewalld)
[ ] Snapshots taken for easy reset
[ ] /dev/sdb exists on server1 (for LVM tasks)
```

---

## Snapshot Strategy (Highly Recommended)

| Snapshot Name              | When to Take                          |
|----------------------------|---------------------------------------|
| `Fresh Install`            | Right after OS install + basic config |
| `Pre-Exam-Run-1`           | Before each mock exam attempt         |
| `Mid-Exam-Checkpoint`      | After completing tasks 1–9            |
| `Post-Exam-Run-1`          | After finishing — review your work    |

**To restore for a fresh attempt:**
```
VMware → VM → Snapshot → Snapshot Manager → Revert to "Pre-Exam-Run-1"
```

---

## Pro Tips for VMware Practice

```bash
# Open two terminal windows — one per VM
# Use tmux to split your server1 terminal:
tmux
# Ctrl+B, % = vertical split
# Ctrl+B, " = horizontal split
# Ctrl+B, arrow = switch pane

# Enable SSH from your Windows host to VMs
# Windows Terminal / PuTTY → 192.168.1.10 (server1)
# This is more comfortable than the VMware console

# Copy-paste works better over SSH than the VM console
```

---

## You're Ready! Start Here:

1. 📖 Review: [Study Guide](file:///C:/Users/chase/.gemini/antigravity/brain/9d871169-caaa-465e-95c0-1387ebde384a/RHCSA_EX200_Study_Guide.md)
2. 🔧 Practice: [Practice Scenarios](file:///C:/Users/chase/.gemini/antigravity/brain/9d871169-caaa-465e-95c0-1387ebde384a/RHCSA_Practice_Scenarios.md)
3. ⏱️ Exam: [Mock Exam](file:///C:/Users/chase/.gemini/antigravity/brain/9d871169-caaa-465e-95c0-1387ebde384a/RHCSA_Mock_Exam.md) ← Start a 3-hour timer!
4. 🗝️ Check: [Answer Key](file:///C:/Users/chase/.gemini/antigravity/brain/9d871169-caaa-465e-95c0-1387ebde384a/RHCSA_Mock_Exam_Answer_Key.md)
