# RHCSA EX200 (RHEL 10) Complete Exam Solutions & Answer Key
*All 22 Tasks from codeXchange RHCSA v10 Exam Practice (Video: kBsMuA-DA9E)*

---

## Task 1: Environment & System Information Review
Verify network reachability and connectivity between node1, node2, and workstation:
```bash
ping -c 2 172.25.250.254
ping -c 2 172.25.250.11
```

---

## Task 2: Network Configuration (node1)
**Requirements:**
- Hostname: `primary.net1.example.com`
- IP: `172.25.250.11` (or assigned IP) / `24`
- Gateway: `172.25.250.254`
- DNS: `172.25.250.220`

**Commands:**
```bash
# 1. Set hostname
hostnamectl set-hostname primary.net1.example.com

# 2. Identify connection name
nmcli connection show

# 3. Configure static IP, gateway, and DNS
nmcli connection modify "Wired connection 1" \
  ipv4.method manual \
  ipv4.addresses 172.25.250.11/24 \
  ipv4.gateway 172.25.250.254 \
  ipv4.dns 172.25.250.220 \
  connection.autoconnect yes

# 4. Reactivate connection
nmcli connection down "Wired connection 1" && nmcli connection up "Wired connection 1"

# 5. Verification
ip addr show
ip route show
hostname
```

---

## Task 3: Default YUM / DNF Repositories (node1)
**Requirements:**
- BaseOS: `http://content.example.com/rhel10.0/x86_64/dvd/BaseOS`
- AppStream: `http://content.example.com/rhel10.0/x86_64/dvd/AppStream`

**Commands:**
```bash
cat << 'EOF' > /etc/yum.repos.d/default.repo
[BaseOS]
name=Red Hat Enterprise Linux 10 - BaseOS
baseurl=http://content.example.com/rhel10.0/x86_64/dvd/BaseOS
enabled=1
gpgcheck=0

[AppStream]
name=Red Hat Enterprise Linux 10 - AppStream
baseurl=http://content.example.com/rhel10.0/x86_64/dvd/AppStream
enabled=1
gpgcheck=0
EOF

# Verification
dnf clean all
dnf repolist
```

---

## Task 4: SELinux & Non-Standard Port for HTTPD (node1)
**Requirements:**
- HTTP service on port 82
- Fix SELinux debug issue
- Files under `/var/www/html`
- Starts at boot
- Verify: `curl http://localhost:82/file1`

**Commands:**
```bash
# 1. Install httpd and semanage tools
dnf install -y httpd policycoreutils-python-utils

# 2. Configure Listen port in /etc/httpd/conf/httpd.conf
sed -i 's/^Listen 80/Listen 82/' /etc/httpd/conf/httpd.conf

# 3. Allow port 82 in SELinux policy
semanage port -a -t http_port_t -p tcp 82
# (If port already defined, use: semanage port -m -t http_port_t -p tcp 82)

# 4. Open firewall for port 82
firewall-cmd --permanent --add-port=82/tcp
firewall-cmd --reload

# 5. Create test file if missing
echo "RHCSA Exam Port 82 Test" > /var/www/html/file1
restorecon -Rv /var/www/html/

# 6. Enable and start httpd
systemctl enable --now httpd

# 7. Verification
curl http://localhost:82/file1
systemctl is-active httpd
```

---

## Task 5: User and Group Administration (node1)
**Requirements:**
- Group `sysadmin`
- Users `natasha` and `harry` with secondary group `sysadmin`
- User `sarah` with nologin shell, not in `sysadmin`
- Password `atenorth` for all

**Commands:**
```bash
# 1. Create group
groupadd sysadmin

# 2. Create users with secondary group
useradd -G sysadmin natasha
useradd -G sysadmin harry

# 3. Create non-interactive user
useradd -s /sbin/nologin sarah

# 4. Set passwords
echo "atenorth" | passwd --stdin natasha
echo "atenorth" | passwd --stdin harry
echo "atenorth" | passwd --stdin sarah

# 5. Verification
id natasha
id harry
id sarah
grep sarah /etc/passwd
```

---

## Task 6: Collaborative Directory Permissions (node1)
**Requirements:**
- Directory: `/common/admin`
- Group: `sysadmin`
- Permissions: `2770` (read/write/execute for group, SGID set, no other access)

**Commands:**
```bash
# 1. Create directory
mkdir -p /common/admin

# 2. Set group ownership
chown :sysadmin /common/admin

# 3. Set SGID and 770 permissions
chmod 2770 /common/admin

# 4. Verification
ls -ld /common/admin
# Output must show: drwxrws---. root sysadmin /common/admin
```

---

## Task 7: Scheduled Cron Jobs (node1)
**Requirements:**
- User `natasha`: daily at 14:23 executes `/bin/echo hello`
- User `harry`: every 3 minutes executes `/usr/bin/logger "EX200 in progress"`

**Commands:**
```bash
# 1. Configure for natasha
crontab -u natasha -e
# Add line:
23 14 * * * /bin/echo hello

# 2. Configure for harry
crontab -u harry -e
# Add line:
*/3 * * * * /usr/bin/logger "EX200 in progress"

# 3. Verification
crontab -u natasha -l
crontab -u harry -l
```

---

## Task 8: Archive & Compression (node1)
**Requirements:**
- Create `/home/backup.tar.bz2` containing `/etc`

**Commands:**
```bash
# Create bzip2 compressed archive
tar -cjf /home/backup.tar.bz2 /etc

# Verification
ls -lh /home/backup.tar.bz2
tar -tvjf /home/backup.tar.bz2 | head -15
```

---

## Task 9: Configure NTP Client (node1)
**Requirements:**
- NTP client of `classroom.example.com`

**Commands:**
```bash
# 1. Edit /etc/chrony.conf
# Replace or add server directive:
sed -i '/^server /d' /etc/chrony.conf
echo "server classroom.example.com iburst" >> /etc/chrony.conf

# 2. Restart and enable chronyd
systemctl restart chronyd
systemctl enable chronyd

# 3. Verification
chronyc sources -v
```

---

## Task 10: Find and Copy Files (node1)
**Requirements:**
- Find all files owned by user `simone`
- Copy into `/root/found`

**Commands:**
```bash
# 1. Create target folder
mkdir -p /root/found

# 2. Find and copy files
find / -user simone -type f -exec cp -a {} /root/found/ \; 2>/dev/null

# 3. Verification
ls -la /root/found
```

---

## Task 11: Find Matching String (node1)
**Requirements:**
- Find string `strato` in `/usr/share/dict/words`
- Save results in `/searchfile`

**Commands:**
```bash
grep 'strato' /usr/share/dict/words > /searchfile

# Verification
cat /searchfile
```

---

## Task 12: Configure Autofs NFS Automount (node1)
**Requirements:**
- NFS Server: `workstation.lab.example.com` (`172.25.250.9`)
- Export: `/ourhome/remoteuser2`
- Automount `remoteuser2`'s home directory upon access
- Writable by user

**Commands:**
```bash
# 1. Install autofs and NFS client
dnf install -y autofs nfs-utils

# 2. Configure master map
echo "/ourhome  /etc/auto.ourhome" >> /etc/auto.master

# 3. Configure direct/indirect map
cat << 'EOF' > /etc/auto.ourhome
remoteuser2  -rw,sync  workstation.lab.example.com:/ourhome/remoteuser2
EOF

# 4. Enable and start autofs
systemctl enable --now autofs

# 5. Verification
ls /ourhome/remoteuser2
df -h | grep ourhome
```

---

## Task 13: Create User with Specific UID (node1)
**Requirements:**
- User `barry`
- UID `2112`
- Password `atenorth`

**Commands:**
```bash
useradd -u 2112 barry
echo "atenorth" | passwd --stdin barry

# Verification
id barry
# Output: uid=2112(barry) gid=2112(barry)
```

---

## Task 14: Scripting, Umask, Sudoers, Password Policy (node1)

### a. Script `newsearch`
```bash
cat << 'EOF' > /usr/local/bin/newsearch
#!/bin/bash
find /usr -size +30k -size -50k -perm -4000 > /root/myoutput 2>/dev/null
EOF
chmod +x /usr/local/bin/newsearch

# Run to verify
/usr/local/bin/newsearch
cat /root/myoutput
```

### b. Set Permissions (Umask) for user `alex`
```bash
# Requirement: new files get mode r--r--r-- (0444)
# Default file permissions 0666 - 0444 = 0222 umask
echo "umask 0222" >> /home/alex/.bashrc
```

### c. Group `elite` with Sudo Privileges Without Password
```bash
groupadd elite
cat << 'EOF' > /etc/sudoers.d/elite
%elite ALL=(ALL) NOPASSWD: ALL
EOF
chmod 0440 /etc/sudoers.d/elite
```

### d. Set Default Password Aging Policy (20 Days)
```bash
sed -i 's/^PASS_MAX_DAYS.*/PASS_MAX_DAYS   20/' /etc/login.defs
grep PASS_MAX_DAYS /etc/login.defs
```

### e. Application `ex200`
```bash
cat << 'EOF' > /usr/local/bin/ex200
#!/bin/bash
echo "EX200 Application Executed by $USER"
EOF
chmod +x /usr/local/bin/ex200
```

---

## Task 15: Systemd Timer Service & ACLs (node1)

### a. Logger Script with Systemd Timer
```bash
# 1. Modify or create script
mkdir -p /root/lookup_directory
cat << 'EOF' > /usr/local/bin/logger
#!/bin/bash
cat /var/tmp/ex200* >> /root/lookup_directory/ex200 2>/dev/null
EOF
chmod +x /usr/local/bin/logger

# 2. Create systemd service
cat << 'EOF' > /etc/systemd/system/logger.service
[Unit]
Description=Logger Data Collection Service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/logger
EOF

# 3. Create systemd timer
cat << 'EOF' > /etc/systemd/system/logger.timer
[Unit]
Description=Run Logger Script Every 1 Minute

[Timer]
OnCalendar=*:0/1
Persistent=true

[Install]
WantedBy=timers.target
EOF

# 4. Enable and start timer
systemctl daemon-reload
systemctl enable --now logger.timer

# Verification
systemctl list-timers logger.timer
```

### b. File ACL on `/var/tmp/fstab`
```bash
# 1. Copy file
cp /etc/fstab /var/tmp/fstab

# 2. Ownership and base permissions
chown root:root /var/tmp/fstab
chmod 0644 /var/tmp/fstab

# 3. Set ACLs
setfacl -m u:natasha:rw /var/tmp/fstab
setfacl -m u:harry:--- /var/tmp/fstab

# 4. Verification
getfacl /var/tmp/fstab
```

---

## Task 16: Reset Root Password on Node 2
**Procedure at GRUB screen:**
1. Reboot `node2` and press `e` at the GRUB menu.
2. Locate the line starting with `linux` or `linux16`.
3. Append `rd.break` to the end of the line.
4. Press `Ctrl + x` to boot into emergency shell.
5. Remount `/sysroot` in read-write mode:
   ```bash
   mount -o remount,rw /sysroot
   chroot /sysroot
   ```
6. Set new root password:
   ```bash
   echo "atenorth" | passwd --stdin root
   ```
7. Trigger SELinux relabeling:
   ```bash
   touch /.autorelabel
   ```
8. Exit and reboot:
   ```bash
   exit
   exit
   ```

---

## Task 17: Default YUM Repositories (node2)
```bash
cat << 'EOF' > /etc/yum.repos.d/default.repo
[BaseOS]
name=BaseOS
baseurl=http://content.example.com/rhel10.0/x86_64/dvd/BaseOS
enabled=1
gpgcheck=0

[AppStream]
name=AppStream
baseurl=http://content.example.com/rhel10.0/x86_64/dvd/AppStream
enabled=1
gpgcheck=0
EOF

dnf clean all
dnf repolist
```

---

## Task 18: Configure Tuned Profile (node2)
```bash
# 1. Query recommendation
tuned-adm recommend

# 2. Apply profile (e.g. virtual-guest or throughput-performance)
PROFILE=$(tuned-adm recommend)
tuned-adm profile $PROFILE

# 3. Verification
tuned-adm active
```

---

## Task 19: User-Specific Flatpak Repository & App (node2)
**Requirements:**
- Flatpak remote `extra` available only to user `bammbamm`
- Install `codium` under user scope

**Commands:**
```bash
# 1. Ensure user exists
useradd bammbamm
echo "atenorth" | passwd --stdin bammbamm

# 2. Switch to user bammbamm or execute with su
su - bammbamm -c "flatpak remote-add --user --if-not-exists extra http://content.example.com/flatpak/repo"
su - bammbamm -c "flatpak install --user -y extra codium"

# 3. Verification
su - bammbamm -c "flatpak list --user"
su - bammbamm -c "flatpak remotes --user"
# Verify root cannot see it in system scope:
flatpak list --system
```

---

## Task 20: Configure Swap Space (node2)
**Requirements:**
- 250 MB swap partition, persistent across reboots

**Commands:**
```bash
# 1. Create partition using fdisk or parted (e.g. /dev/vdb1 or /dev/sdb1)
# 2. Format as swap
mkswap /dev/vdb1

# 3. Get UUID
SWAP_UUID=$(blkid -s UUID -o value /dev/vdb1)

# 4. Add to /etc/fstab
echo "UUID=$SWAP_UUID swap swap defaults 0 0" >> /etc/fstab

# 5. Activate swap
swapon -a

# 6. Verification
swapon --show
free -m
```

---

## Task 21: Volume Group & VFAT Filesystem (node2)
**Requirements:**
- Volume group `myvol` with `8 MiB` Physical Extent size
- Logical volume `mydatabase` with `50 P.E.` (400 MB)
- Formatted as `vfat`
- Mounted permanently on `/database`

**Commands:**
```bash
# 1. Install dosfstools if needed
dnf install -y dosfstools

# 2. Create VG with 8M extent size
vgcreate -s 8M myvol /dev/vdb2

# 3. Create LV with 50 extents
lvcreate -l 50 -n mydatabase myvol

# 4. Format with vfat
mkfs.vfat /dev/myvol/mydatabase

# 5. Mount permanently
mkdir -p /database
echo "/dev/myvol/mydatabase /database vfat defaults 0 0" >> /etc/fstab
mount -a

# 6. Verification
df -h /database
vgs myvol
lvs myvol/mydatabase
```

---

## Task 22: Resize Logical Volume (node2)
**Requirements:**
- Resize `home` in `myvg` to `152 MB (38 extents)`
- Size must end between `145 MB and 155 MB`
- Maintain filesystem on `/home`

**Commands:**
```bash
# For ext4 / XFS (using lvresize with -r for automatic filesystem resize):
# Note: XFS can only grow. If ext4, it can grow or shrink.
lvresize -r -L 152M /dev/myvg/home
# Or by extent count:
# lvresize -r -l 38 /dev/myvg/home

# Verification
lvs /dev/myvg/home
df -h /home
```
