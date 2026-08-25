# RHCSA EX200 – Mock Exam Answer Key
### Full Solutions for All 18 Tasks

> ⚠️ **Try the exam first before reading this!**
> Reference doc: [RHCSA_Mock_Exam.md](file:///C:/Users/chase/.gemini/antigravity/brain/9d871169-caaa-465e-95c0-1387ebde384a/RHCSA_Mock_Exam.md)

---

## TASK 01 — Networking and Hostname `[10 pts]`

```bash
# Set hostname permanently
hostnamectl set-hostname server1.example.com

# Configure static IP using nmcli
nmcli con show                                             # Note connection name (e.g., enp0s3)
nmcli con mod enp0s3 ipv4.method manual \
    ipv4.addresses 192.168.1.10/24 \
    ipv4.gateway 192.168.1.1 \
    ipv4.dns 8.8.8.8
nmcli con down enp0s3 && nmcli con up enp0s3

# Verify
hostnamectl                                                # Shows server1.example.com
ip addr show enp0s3                                        # Shows 192.168.1.10/24
ping -c 2 192.168.1.20                                    # Ping client1
```

**Grading Rubric**:
- Hostname set correctly: 2 pts
- Static IP applied: 3 pts
- Gateway set: 2 pts
- DNS set: 1 pt
- Connectivity verified: 2 pts

---

## TASK 02 — User and Group Management `[15 pts]`

```bash
# Create groups
groupadd developers
groupadd sysadmins

# Create users with specific UIDs and comments
useradd -u 1501 -g developers -G sysadmins -c "Alice Developer" alice
useradd -u 1502 -g developers -c "Bob Developer" bob
useradd -u 1503 -g sysadmins -G developers -c "Carol Sysadmin" carol

# Set passwords
echo "alice:RedHat1!" | chpasswd
echo "bob:RedHat1!" | chpasswd
echo "carol:RedHat1!" | chpasswd

# Set password expiry for bob (30 days)
chage -M 30 bob
chage -l bob                     # Verify: Maximum number of days = 30

# Lock carol's account
usermod -L carol
passwd -l carol                  # Alternative
grep carol /etc/shadow           # Should show ! prefix on password hash
```

**Verify**
```bash
id alice                         # uid=1501 gid=developers groups=developers,sysadmins
id bob                           # uid=1502 gid=developers
id carol                         # uid=1503 gid=sysadmins groups=sysadmins,developers
chage -l bob                     # Maximum: 30
grep carol /etc/shadow | cut -d: -f2  # Starts with !
```

**Grading Rubric**:
- Groups created: 2 pts
- Users with correct UID/group: 6 pts (2 each)
- Passwords set: 2 pts
- bob expiry: 3 pts
- carol locked: 2 pts

---

## TASK 03 — File Permissions and Special Bits `[15 pts]`

```bash
# Create /projects/dev
mkdir -p /projects/dev
chown alice:developers /projects/dev
chmod 770 /projects/dev           # rwxrwx--- = 770

# Set setgid bit on /projects/dev
chmod g+s /projects/dev
# OR: chmod 2770 /projects/dev

# Create /projects/shared with sticky bit
mkdir -p /projects/shared
chmod 1777 /projects/shared       # rwxrwxrwx + sticky = 1777
# OR:
chmod 777 /projects/shared
chmod +t /projects/shared
```

**Verify**
```bash
ls -ld /projects/dev
# drwxrws--- 2 alice developers ... (s in group execute position = setgid)

ls -ld /projects/shared
# drwxrwxrwt ... (t in other execute position = sticky bit)

# Test setgid - new files should inherit developers group
su - alice
touch /projects/dev/testfile
ls -l /projects/dev/testfile     # Group should be developers
```

**Grading Rubric**:
- `/projects/dev` correct owner: 2 pts
- `/projects/dev` correct permissions (770): 3 pts
- setgid on `/projects/dev`: 3 pts
- `/projects/shared` sticky bit: 4 pts
- `/projects/shared` permissions (1777): 3 pts

---

## TASK 04 — File Search and Archive `[10 pts]`

```bash
# Files in /etc modified in last 3 days
find /etc -mtime -3 -type f > /tmp/recent_etc.txt
cat /tmp/recent_etc.txt          # Verify

# Files in /usr larger than 5MB
find /usr -type f -size +5M > /tmp/large_usr.txt
cat /tmp/large_usr.txt

# Create compressed archive of /etc/ssh
tar -czf /tmp/ssh_backup.tar.gz /etc/ssh

# List archive contents
tar -tzvf /tmp/ssh_backup.tar.gz
```

**Verify**
```bash
[ -s /tmp/recent_etc.txt ] && echo "OK" || echo "EMPTY"
[ -s /tmp/large_usr.txt ]  && echo "OK" || echo "EMPTY"
ls -lh /tmp/ssh_backup.tar.gz   # Should exist with size
tar -tzf /tmp/ssh_backup.tar.gz | head  # Should list etc/ssh/...
```

**Grading Rubric**:
- find by mtime: 3 pts
- find by size: 3 pts
- tar create: 2 pts
- tar list verify: 2 pts

---

## TASK 05 — Software Management `[10 pts]`

```bash
# Install Development Tools group
dnf group install "Development Tools" -y

# Install individual packages
dnf install httpd wget vim -y

# Enable httpd at boot but don't start it now
systemctl enable httpd
systemctl stop httpd             # Ensure it's stopped

# Verify
systemctl is-enabled httpd       # Should return: enabled
systemctl is-active httpd        # Should return: inactive

# Save installed package list
dnf list installed > /tmp/installed_packages.txt
wc -l /tmp/installed_packages.txt    # Should be hundreds of lines
```

**Grading Rubric**:
- Development Tools installed: 2 pts
- httpd/wget/vim installed: 3 pts
- httpd enabled but not running: 3 pts
- Package list saved: 2 pts

---

## TASK 06 — Systemd Service `[10 pts]`

```bash
# Create the script first
cat > /usr/local/bin/cleanup.sh << 'EOF'
#!/bin/bash
find /tmp -type f -mtime +7 -delete 2>/dev/null
echo "$(date '+%Y-%m-%d %H:%M:%S') - Cleanup complete" >> /var/log/cleanup.log
EOF
chmod +x /usr/local/bin/cleanup.sh

# Create the systemd unit
cat > /etc/systemd/system/cleanup.service << 'EOF'
[Unit]
Description=System Cleanup Service
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/cleanup.sh

[Install]
WantedBy=multi-user.target
EOF

# Enable and test
systemctl daemon-reload
systemctl enable cleanup.service
systemctl start cleanup.service
systemctl status cleanup.service
cat /var/log/cleanup.log
```

**Grading Rubric**:
- Script created and executable: 2 pts
- Service file correct (Type=oneshot): 3 pts
- After=network.target: 1 pt
- Service enabled: 2 pts
- Log entry created on start: 2 pts

---

## TASK 07 — Scheduled Tasks `[10 pts]`

```bash
# Cron job for alice at 2:30 AM daily
crontab -u alice -e
# Add: 30 2 * * * /usr/local/bin/diskcheck.sh

# Cron job for root every Sunday at 3:00 AM
crontab -u root -e
# Add: 0 3 * * 0 /usr/bin/dnf update -y

# OR: use /etc/crontab or /etc/cron.d/
cat > /etc/cron.d/maintenance << 'EOF'
30 2 * * * alice /usr/local/bin/diskcheck.sh
0 3 * * 0 root /usr/bin/dnf update -y
EOF

# One-time at job (5 minutes from now)
echo 'echo "Maintenance window complete" > /tmp/maint.txt' | at now + 5 minutes

# Enable crond
systemctl enable --now crond
```

**Verify**
```bash
crontab -u alice -l                # Shows alice's cron job
crontab -u root -l                 # Shows root's cron job
atq                                # Shows pending at jobs
systemctl is-active crond          # active
```

**Grading Rubric**:
- alice cron (correct time): 3 pts
- root cron (correct time + day): 3 pts
- at job created: 2 pts
- crond enabled and active: 2 pts

---

## TASK 08 — sudo Configuration `[10 pts]`

```bash
# Method: create a drop-in file in /etc/sudoers.d/ (safest approach)

# All sysadmins — any command, no password
cat > /etc/sudoers.d/sysadmins << 'EOF'
%sysadmins ALL=(ALL) NOPASSWD: ALL
EOF
chmod 440 /etc/sudoers.d/sysadmins

# bob — only systemctl and journalctl, with password
cat > /etc/sudoers.d/bob << 'EOF'
bob ALL=(ALL) /usr/bin/systemctl, /usr/bin/journalctl
EOF
chmod 440 /etc/sudoers.d/bob

# Verify syntax
visudo -c

# Test as alice
su - alice -c "sudo whoami"      # Should return: root (no password prompt)
```

**Grading Rubric**:
- sysadmins group NOPASSWD sudo: 4 pts
- bob restricted to two commands: 4 pts
- Correct file permissions (440): 1 pt
- Verification test passes: 1 pt

---

## TASK 09 — LVM Storage `[20 pts]`

```bash
# Create PV
pvcreate /dev/sdb
pvs                              # Verify PV

# Create VG
vgcreate vg_data /dev/sdb
vgs                              # Verify VG

# Create LV (2G)
lvcreate -L 2G -n lv_storage vg_data
lvs                              # Verify LV: /dev/vg_data/lv_storage

# Format with XFS
mkfs.xfs /dev/vg_data/lv_storage

# Mount persistently
mkdir -p /mnt/storage
echo "/dev/vg_data/lv_storage  /mnt/storage  xfs  defaults  0 0" >> /etc/fstab
mount -a
df -h /mnt/storage               # Verify ~2G

# Extend LV by 1G (total 3G) — while mounted!
lvextend -L +1G /dev/vg_data/lv_storage
lvs                              # Should now show 3G

# Grow XFS filesystem (online — no unmount needed)
xfs_growfs /mnt/storage

# Verify
df -h /mnt/storage               # Should now show ~3G
```

**Grading Rubric**:
- PV created: 2 pts
- VG created: 2 pts
- LV created at 2G: 3 pts
- XFS formatted: 3 pts
- fstab entry correct: 3 pts
- Mount working: 2 pts
- Extended to 3G: 3 pts
- xfs_growfs applied: 2 pts

---

## TASK 10 — NFS Server `[20 pts]`

```bash
# Install nfs-utils
dnf install nfs-utils -y

# Create export directory
mkdir -p /nfsexports/team
chmod 755 /nfsexports/team

# Create test file
echo "NFS Share Ready" > /nfsexports/team/welcome.txt

# Configure export
cat > /etc/exports << 'EOF'
/nfsexports/team  192.168.1.0/24(rw,sync,root_squash)
EOF

# Enable and start NFS server
systemctl enable --now nfs-server

# Apply exports
exportfs -rav

# Firewall
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-service=mountd
firewall-cmd --reload

# Verify
exportfs -v
showmount -e localhost
systemctl is-enabled nfs-server    # enabled
systemctl is-active nfs-server     # active
firewall-cmd --list-services       # includes nfs rpc-bind mountd
```

**Grading Rubric**:
- nfs-utils installed: 1 pt
- Directory created (correct perms): 2 pts
- welcome.txt created: 1 pt
- /etc/exports correct (rw, sync, root_squash): 6 pts
- nfs-server enabled + active: 4 pts
- exportfs -rav applied: 2 pts
- Firewall (all 3 services): 4 pts

---

## TASK 11 — NFS Client / AutoFS `[20 pts]`

**On client1:**
```bash
# Install packages
dnf install autofs nfs-utils -y

# Configure auto.master
echo "/mnt/nfs  /etc/auto.nfs" >> /etc/auto.master

# Create the map file
cat > /etc/auto.nfs << 'EOF'
team  -rw,sync  192.168.1.10:/nfsexports/team
EOF

# Create base directory (autofs needs the parent)
mkdir -p /mnt/nfs

# Enable and start autofs
systemctl enable --now autofs

# Test: accessing the path triggers the mount
ls /mnt/nfs/team
cat /mnt/nfs/team/welcome.txt    # Should show: NFS Share Ready

# Verify mount
mount | grep nfs
```

**Grading Rubric**:
- autofs/nfs-utils installed: 2 pts
- auto.master entry correct: 4 pts
- /etc/auto.nfs map correct (key, options, server:path): 6 pts
- autofs enabled + active: 4 pts
- welcome.txt readable via autofs path: 4 pts

---

## TASK 12 — Script: Health Report `[25 pts]`

```bash
cat > /usr/local/bin/healthreport.sh << 'SCRIPT'
#!/bin/bash
# System Health Report Script

TARGETDIR="${1:-/}"
LOGDATE=$(date '+%Y%m%d')
LOGFILE="/var/log/healthreport_${LOGDATE}.log"

# Validate directory
if [ ! -d "$TARGETDIR" ]; then
    echo "ERROR: '$TARGETDIR' is not a valid directory" >&2
    exit 1
fi

{
    echo "========================================"
    echo "  SYSTEM HEALTH REPORT"
    echo "  Generated: $(date '+%Y-%m-%d %H:%M:%S')"
    echo "========================================"
    echo ""

    echo "--- HOSTNAME ---"
    hostname -f
    echo ""

    echo "--- UPTIME ---"
    uptime
    echo ""

    echo "--- CPU LOAD ---"
    uptime | awk -F'load average:' '{print "Load averages:" $2}'
    echo ""

    echo "--- MEMORY (MB) ---"
    free -m | awk '/^Mem:/ {printf "Total: %s MB | Used: %s MB | Free: %s MB\n", $2, $3, $4}'
    echo ""

    echo "--- DISK USAGE ($TARGETDIR) ---"
    df -h "$TARGETDIR" | tail -1 | awk '{printf "Used: %s / Total: %s (%s used)\n", $3, $2, $5}'
    echo ""

    echo "--- TOP 5 PROCESSES BY CPU ---"
    ps aux --sort=-%cpu | awk 'NR==1 || NR<=6 {printf "%-15s %s\n", $11, $3}' | head -6
    echo ""

    echo "--- FAILED SYSTEMD SERVICES ---"
    systemctl --failed --no-legend || echo "None"
    echo ""

    echo "========================================"
} > "$LOGFILE" 2>&1

if [ $? -eq 0 ]; then
    echo "Report saved to $LOGFILE"
    exit 0
else
    echo "ERROR: Report generation failed" >&2
    exit 1
fi
SCRIPT

chmod +x /usr/local/bin/healthreport.sh
```

**Test**
```bash
/usr/local/bin/healthreport.sh /var
cat /var/log/healthreport_$(date +%Y%m%d).log
/usr/local/bin/healthreport.sh /nonexistent   # Should exit 1
echo "Exit: $?"
```

**Grading Rubric**:
- Script executable, correct shebang: 2 pts
- Default arg handling (`${1:-/}`): 2 pts
- Hostname/date section: 2 pts
- Uptime section: 2 pts
- CPU load section: 3 pts
- Memory section (total/used/free): 3 pts
- Disk usage for argument dir: 3 pts
- Top 5 processes: 4 pts
- Failed services section: 2 pts
- Dated logfile + success message: 2 pts

---

## TASK 13 — Script: User Provisioning `[20 pts]`

```bash
cat > /usr/local/bin/provision_users.sh << 'SCRIPT'
#!/bin/bash
# User provisioning script from CSV
# Format: username,group,password

CSV_FILE="$1"
LOGFILE="/var/log/provision.log"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$LOGFILE"
}

# Validate input
if [ -z "$CSV_FILE" ]; then
    echo "Usage: $0 <csv_file>" >&2
    exit 1
fi

if [ ! -f "$CSV_FILE" ]; then
    echo "ERROR: File '$CSV_FILE' not found" >&2
    exit 1
fi

# Process each line
while IFS=, read -r USERNAME GROUP PASSWORD; do
    # Skip empty lines or comment lines
    [[ -z "$USERNAME" || "$USERNAME" == \#* ]] && continue

    # Create group if it doesn't exist
    if ! getent group "$GROUP" &>/dev/null; then
        groupadd "$GROUP"
        log "Group created: $GROUP"
    fi

    # Check if user already exists
    if id "$USERNAME" &>/dev/null; then
        log "SKIP: User '$USERNAME' already exists"
        continue
    fi

    # Create user
    useradd -g "$GROUP" -m "$USERNAME"
    echo "$USERNAME:$PASSWORD" | chpasswd

    # Force password change on first login
    chage -d 0 "$USERNAME"

    log "Created user: $USERNAME (group: $GROUP, must change password)"

done < "$CSV_FILE"

log "Provisioning complete."
SCRIPT

chmod +x /usr/local/bin/provision_users.sh

# Create test CSV
cat > /tmp/newusers.csv << 'EOF'
dave,qa,DevPass1!
eve,qa,DevPass1!
frank,ops,OpsPass1!
EOF

# Run it
/usr/local/bin/provision_users.sh /tmp/newusers.csv
```

**Verify**
```bash
id dave                          # uid exists, group=qa
id eve                           # uid exists, group=qa
id frank                         # uid exists, group=ops
chage -l dave | grep "Last pass" # Should show Jan 01, 1970 (force change)
cat /var/log/provision.log       # Should show all actions with timestamps
/usr/local/bin/provision_users.sh  # No arg — should exit 1 with usage
```

**Grading Rubric**:
- Argument validation + usage: 2 pts
- File existence check: 2 pts
- Group creation logic: 3 pts
- User existence check (skip): 2 pts
- useradd with correct group: 3 pts
- Password set: 2 pts
- Force password change (chage -d 0): 3 pts
- Timestamped logging: 3 pts

---

## TASK 14 — Container: Run and Manage `[20 pts]`

**As alice:**
```bash
su - alice

# Pull the image
podman pull registry.access.redhat.com/ubi9/ubi-minimal

# Run reportgen with read-only /var/log mount
podman run -d \
  --name reportgen \
  -v /var/log:/hostlogs:ro,Z \
  registry.access.redhat.com/ubi9/ubi-minimal \
  sleep infinity

# Verify access inside container
podman exec reportgen ls /hostlogs

# Stop and remove
podman stop reportgen
podman rm reportgen

# Prepare webdata directory
mkdir -p /srv/webdata
echo "<h1>RHCSA Web Server</h1>" > /srv/webdata/index.html

# Run webserver container
podman run -d \
  --name webserver \
  -p 8888:80 \
  -v /srv/webdata:/usr/local/apache2/htdocs:Z \
  docker.io/library/httpd:latest

# Verify
curl http://localhost:8888
```

**Grading Rubric**:
- ubi-minimal pulled: 2 pts
- reportgen with ro volume and :Z: 4 pts
- exec into container verifies files: 2 pts
- stop + rm reportgen: 2 pts
- /srv/webdata created with index.html: 2 pts
- webserver on port 8888 with :Z: 4 pts
- curl returns 200: 4 pts

---

## TASK 15 — Container: Build Custom Image `[20 pts]`

```bash
# Create working directory
mkdir -p /opt/containerbuilds/sysinfo
cd /opt/containerbuilds/sysinfo

# Create the sysinfo script
cat > sysinfo.sh << 'EOF'
#!/bin/bash
echo "Hostname:       $(hostname)"
echo "Kernel:         $(uname -r)"
echo "Date/Time:      $(date)"
EOF

# Create Containerfile
cat > Containerfile << 'EOF'
FROM registry.access.redhat.com/ubi9/ubi-minimal

COPY sysinfo.sh /usr/local/bin/sysinfo.sh

RUN chmod +x /usr/local/bin/sysinfo.sh

CMD ["/usr/local/bin/sysinfo.sh"]
EOF

# Build the image
podman build -t sysinfo:1.0 .

# Test the image (runs and exits)
podman run --rm sysinfo:1.0

# Tag as latest
podman tag sysinfo:1.0 sysinfo:latest

# Verify
podman images | grep sysinfo
```

**Grading Rubric**:
- Correct working directory: 1 pt
- sysinfo.sh with hostname, kernel, date: 4 pts
- Containerfile FROM correct base: 2 pts
- COPY instruction: 2 pts
- RUN chmod +x: 2 pts
- CMD correct: 2 pts
- Image built successfully: 3 pts
- Runs and prints system info: 2 pts
- Tagged as :latest: 2 pts

---

## TASK 16 — Container: Systemd User Service `[20 pts]`

```bash
# Create devops user (as root)
useradd -m -s /bin/bash devops
echo "devops:RedHat1!" | chpasswd

# Switch to devops
su - devops

# Pull nginx:alpine
podman pull docker.io/library/nginx:alpine

# Run container
podman run -d --name nginxsvc -p 9090:80 nginx:alpine

# Create systemd user directory
mkdir -p ~/.config/systemd/user/

# Generate service unit
podman generate systemd --name nginxsvc --files --new
mv container-nginxsvc.service ~/.config/systemd/user/

# Enable user service
systemctl --user daemon-reload
systemctl --user enable --now container-nginxsvc

# Verify service is active
systemctl --user status container-nginxsvc

# Exit back to root
exit

# Enable linger for boot startup without login (as root)
loginctl enable-linger devops

# Verify
loginctl show-user devops | grep Linger   # Linger=yes
curl http://localhost:9090                # nginx welcome page
```

**Grading Rubric**:
- devops user created: 2 pts
- nginx:alpine pulled: 2 pts
- Container running on port 9090: 3 pts
- Systemd unit generated correctly: 3 pts
- User service enabled and active: 4 pts
- loginctl enable-linger: 4 pts
- curl localhost:9090 returns 200: 2 pts

---

## TASK 17 — SELinux: Context and Booleans `[20 pts]`

**Part A: Custom DocumentRoot**
```bash
# Create directory and file
mkdir -p /webapps/portal
echo "SELinux Portal" > /webapps/portal/index.html

# Update Apache config (edit /etc/httpd/conf/httpd.conf)
sed -i 's|DocumentRoot "/var/www/html"|DocumentRoot "/webapps/portal"|' /etc/httpd/conf/httpd.conf
sed -i 's|<Directory "/var/www/html">|<Directory "/webapps/portal">|' /etc/httpd/conf/httpd.conf

# Fix SELinux context (permanent)
dnf install policycoreutils-python-utils -y
semanage fcontext -a -t httpd_sys_content_t "/webapps/portal(/.*)?"
restorecon -Rv /webapps/portal/

# Verify context
ls -lZ /webapps/portal/          # Should show httpd_sys_content_t

# Start Apache
systemctl enable --now httpd
curl http://localhost             # Should return "SELinux Portal"
```

**Part B: Outbound Connections Boolean**
```bash
getsebool httpd_can_network_connect    # Confirm it's off
setsebool -P httpd_can_network_connect on
getsebool httpd_can_network_connect    # Should return: on
```

**Part C: Non-Standard Port**
```bash
# Add Listen directive
echo "Listen 8080" >> /etc/httpd/conf/httpd.conf

# Add port to SELinux
semanage port -a -t http_port_t -p tcp 8080

# Verify
semanage port -l | grep "8080"

# Open firewall
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload

# Restart Apache
systemctl restart httpd

# Test
curl http://localhost:8080
```

**Grading Rubric**:
- Part A - Directory/file created: 1 pt
- Part A - httpd.conf updated: 2 pts
- Part A - semanage fcontext correct: 3 pts
- Part A - restorecon applied: 2 pts
- Part A - curl returns content: 2 pts
- Part B - boolean set permanently: 4 pts
- Part C - semanage port 8080: 3 pts
- Part C - firewall port 8080: 2 pts
- Part C - curl 8080 works: 1 pt

---

## TASK 18 — SELinux: Audit and Remediation `[25 pts]`

```bash
# Step 1: Create directories and files
mkdir -p /var/log/myapp
touch /var/log/myapp/app.log
mkdir -p /opt/appdata
echo "test data" > /opt/appdata/testfile.txt

# Step 2: Confirm Enforcing
getenforce                                  # Should return: Enforcing

# Step 3: Set context on /var/log/myapp/ permanently
semanage fcontext -a -t var_log_t "/var/log/myapp(/.*)?"
restorecon -Rv /var/log/myapp/
ls -ldZ /var/log/myapp/                     # Verify: var_log_t

# Step 4: Set context on /opt/appdata/ permanently
semanage fcontext -a -t httpd_sys_content_t "/opt/appdata(/.*)?"
restorecon -Rv /opt/appdata/
ls -ldZ /opt/appdata/                       # Verify: httpd_sys_content_t

# Step 5: Add port 8443 as allowed HTTPS port
semanage port -l | grep https_port          # See current https ports
semanage port -a -t https_port_t -p tcp 8443

# Step 6: Verify all changes
semanage fcontext -l | grep myapp           # Shows var_log_t rule
semanage fcontext -l | grep appdata         # Shows httpd_sys_content_t rule
semanage port -l | grep 8443               # Shows https_port_t for 8443

ls -ldZ /var/log/myapp/                    # var_log_t
ls -ldZ /opt/appdata/                      # httpd_sys_content_t

# Step 7: Check for AVC denials
ausearch -m avc -ts recent                  # Should be clean after fixes
```

**Grading Rubric**:
- Directories and files created: 2 pts
- getenforce confirms Enforcing: 2 pts
- semanage fcontext for /var/log/myapp: 4 pts
- restorecon applied for /var/log/myapp: 2 pts
- semanage fcontext for /opt/appdata: 4 pts
- restorecon applied for /opt/appdata: 2 pts
- semanage port for 8443 (https_port_t): 5 pts
- ls -lZ verification correct contexts: 2 pts
- ausearch shows no denials: 2 pts

---

## Post-Reboot Verification Checklist

After completing all tasks, **reboot the system** and verify:

```bash
# Boot verification
reboot

# After reboot — run these checks:
hostnamectl                              # Task 01: hostname persists
id alice && id bob && id carol           # Task 02: users exist
ls -ld /projects/dev /projects/shared   # Task 03: perms persist
systemctl is-enabled cleanup.service     # Task 06: enabled
systemctl is-enabled crond               # Task 07: crond active
systemctl is-enabled nfs-server          # Task 10: NFS enabled
ls /mnt/nfs/team/welcome.txt            # Task 11: AutoFS works
df -h /mnt/storage                       # Task 09: LVM mounted
podman ps                               # Task 14/16: containers running (if root service)
systemctl --user status container-nginxsvc  # Task 16: user service
curl http://localhost                    # Task 17: Apache serving
curl http://localhost:8080              # Task 17 Part C
curl http://localhost:9090              # Task 16: nginx
getenforce                              # Always: Enforcing!
```

---

## Score Self-Assessment Sheet

| Task | Max  | Self-Score | Notes                              |
|------|------|------------|------------------------------------|
| 01   | 10   |            |                                    |
| 02   | 15   |            |                                    |
| 03   | 15   |            |                                    |
| 04   | 10   |            |                                    |
| 05   | 10   |            |                                    |
| 06   | 10   |            |                                    |
| 07   | 10   |            |                                    |
| 08   | 10   |            |                                    |
| 09   | 20   |            |                                    |
| 10   | 20   |            |                                    |
| 11   | 20   |            |                                    |
| 12   | 25   |            |                                    |
| 13   | 20   |            |                                    |
| 14   | 20   |            |                                    |
| 15   | 20   |            |                                    |
| 16   | 20   |            |                                    |
| 17   | 20   |            |                                    |
| 18   | 25   |            |                                    |
|**TOTAL**|**300**|         |**Pass = 210+**                     |

---

## Top Point-Loss Mistakes

| Mistake                                           | Points at Risk |
|---------------------------------------------------|----------------|
| Forgetting `systemctl enable` (service not persistent) | 2–4 pts per task |
| Using `chcon` instead of `semanage + restorecon`  | 3–5 pts        |
| Missing `exportfs -rav` after editing /etc/exports| 2–4 pts        |
| Missing `_netdev` in fstab NFS entry             | 2 pts          |
| Forgetting `loginctl enable-linger` for containers| 4 pts          |
| Missing `:Z` on container volume mounts          | 2–4 pts        |
| `setsebool` without `-P` flag (not persistent)   | 2–3 pts        |
| Firewall not updated after service config        | 2–4 pts        |
| Script not made executable (`chmod +x`)          | 2 pts          |
| Not running `firewall-cmd --reload` after changes| 1–2 pts        |
