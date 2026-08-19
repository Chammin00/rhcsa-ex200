# RHCSA EX200 – RHEL 9 Study Guide
### Focus Areas: NFS · Scripting · Containers · SELinux

---

## Table of Contents
1. [NFS (Network File System)](#1-nfs-network-file-system)
2. [Shell Scripting](#2-shell-scripting)
3. [Containers (Podman)](#3-containers-podman)
4. [SELinux](#4-selinux)
5. [Quick Reference Cheat Sheet](#5-quick-reference-cheat-sheet)
6. [Common Exam Pitfalls](#6-common-exam-pitfalls)
7. [Practice Exam Tasks](#7-practice-exam-tasks)

---

## 1. NFS (Network File System)

### 1.1 NFS Server Setup

#### Install Packages
```bash
dnf install nfs-utils -y
```

#### Create and Export a Share
```bash
# Create the directory to share
mkdir -p /exports/shared

# Set permissions
chmod 755 /exports/shared

# Edit /etc/exports
echo "/exports/shared  192.168.1.0/24(rw,sync,no_root_squash)" >> /etc/exports
# OR for a single host:
echo "/exports/shared  192.168.1.100(rw,sync)" >> /etc/exports
```

#### /etc/exports Options Cheat Sheet
| Option            | Meaning                                       |
|-------------------|-----------------------------------------------|
| `rw`              | Read/Write access                             |
| `ro`              | Read-only access                              |
| `sync`            | Write data to disk before replying            |
| `async`           | Reply before writing (faster, less safe)      |
| `no_root_squash`  | Root on client = root on server               |
| `root_squash`     | Root on client = `nfsnobody` (default/safe)   |
| `no_all_squash`   | Users keep their UIDs (default)               |
| `all_squash`      | All users mapped to `nfsnobody`               |

#### Enable and Start Services
```bash
systemctl enable --now nfs-server
systemctl enable --now rpcbind

# Apply export changes without restart
exportfs -rav

# Verify exports
exportfs -v
showmount -e localhost
```

#### Firewall Rules for NFS Server
```bash
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-service=mountd
firewall-cmd --reload
```

---

### 1.2 NFS Client Setup

#### Install Packages
```bash
dnf install nfs-utils -y
```

#### Manual Mount
```bash
# Discover exports from server
showmount -e 192.168.1.10

# Mount the share
mount -t nfs 192.168.1.10:/exports/shared /mnt/nfs
```

#### Persistent Mount via /etc/fstab
```
192.168.1.10:/exports/shared  /mnt/nfs  nfs  defaults,_netdev  0 0
```
> **IMPORTANT**: Use `_netdev` option so the mount waits for the network to be up.

#### AutoFS (Automounter) – Exam Favourite!
```bash
dnf install autofs -y

# 1. Edit /etc/auto.master (or drop a file in /etc/auto.master.d/)
echo "/mnt/auto  /etc/auto.nfs" >> /etc/auto.master

# 2. Create the map file /etc/auto.nfs
echo "shared  -rw,sync  192.168.1.10:/exports/shared" > /etc/auto.nfs

# 3. Enable and start autofs
systemctl enable --now autofs

# 4. Test (ls triggers the mount)
ls /mnt/auto/shared
```

#### AutoFS Key Concepts
- `/mnt/auto` is the **base directory** (parent key in auto.master)
- `shared` is the **key** that maps to the NFS share
- Full path `/mnt/auto/shared` is auto-created and mounted on access
- Unmounts automatically after idle timeout (default 300s)

---

### 1.3 NFS Troubleshooting
```bash
systemctl status nfs-server     # Check server status
exportfs -v                      # List exports
rpcinfo -p                       # Check RPC services
showmount -e <server-ip>        # Test from client
mount | grep nfs                 # Check active mounts
df -h | grep nfs                 # Show mounted NFS
```

---

## 2. Shell Scripting

### 2.1 Script Structure
```bash
#!/bin/bash
# Description: Script template

# Variables
NAME="World"
COUNT=5

echo "Hello, $NAME"
```

### 2.2 Variables and Input
```bash
#!/bin/bash
echo "Script:    $0"
echo "First arg: $1"
echo "All args:  $@"
echo "Arg count: $#"

read -p "Enter username: " USERNAME

# Default value if arg not provided
LOGDIR="${1:-/var/log}"
```

### 2.3 Conditionals
```bash
#!/bin/bash
if [ "$1" == "start" ]; then
    echo "Starting..."
elif [ "$1" == "stop" ]; then
    echo "Stopping..."
else
    echo "Usage: $0 {start|stop}"
    exit 1
fi
```

#### File Test Operators Quick Reference
| Test  | True When                        |
|-------|----------------------------------|
| `-f`  | Regular file exists              |
| `-d`  | Directory exists                 |
| `-e`  | Exists (any type)                |
| `-r`  | Readable                         |
| `-w`  | Writable                         |
| `-x`  | Executable                       |
| `-s`  | Exists and NOT empty             |
| `-L`  | Symbolic link                    |
| `-z`  | String is empty                  |
| `-n`  | String is NOT empty              |

### 2.4 Loops
```bash
#!/bin/bash

# for – list
for ITEM in apple banana cherry; do
    echo "Fruit: $ITEM"
done

# for – range
for i in {1..10}; do
    echo "$i"
done

# while loop
COUNT=1
while [ $COUNT -le 5 ]; do
    echo "Count: $COUNT"
    (( COUNT++ ))
done

# Loop over lines in a file
while IFS= read -r LINE; do
    echo "Line: $LINE"
done < /etc/passwd

# Loop over files
for FILE in /etc/*.conf; do
    echo "Config: $FILE"
done
```

### 2.5 Functions
```bash
#!/bin/bash

greet() {
    local NAME="$1"    # local variable
    echo "Hello, $NAME!"
    return 0
}

greet "Alice"
greet "Bob"
```

### 2.6 String Operations
```bash
STR="Hello World"
echo "${#STR}"           # Length: 11
echo "${STR,,}"          # Lowercase: hello world
echo "${STR^^}"          # Uppercase: HELLO WORLD
echo "${STR:6}"          # Substring: World
echo "${STR/World/RHEL}" # Replace first: Hello RHEL
echo "${STR#Hello }"     # Strip prefix: World
echo "${STR%World}"      # Strip suffix: Hello 
```

### 2.7 Arithmetic
```bash
RESULT=$(( 5 + 3 ))      # Using $(( ))
(( COUNT++ ))             # Increment
(( COUNT += 5 ))          # Add
RESULT=$(expr 5 + 3)      # Using expr (older)
RESULT=$(echo "scale=2; 10/3" | bc)  # Float with bc
```

### 2.8 Exit Codes and Redirection
```bash
set -e              # Exit on any error
set -u              # Error on unset variables

# Check last exit code
if [ $? -ne 0 ]; then
    echo "ERROR" >&2; exit 1
fi

command 2>/dev/null           # Discard stderr
command &>all.log             # Redirect stdout+stderr to file
command 2>&1 | tee output.log # Both to file AND screen
```

### 2.9 Typical Exam Script Tasks

#### Create users from a file
```bash
#!/bin/bash
USERFILE="${1:-users.txt}"

[ ! -f "$USERFILE" ] && { echo "Error: $USERFILE not found" >&2; exit 1; }

while IFS= read -r USER; do
    if id "$USER" &>/dev/null; then
        echo "User $USER already exists, skipping."
    else
        useradd "$USER"
        echo "$USER:RedHat1!" | chpasswd
        echo "Created: $USER"
    fi
done < "$USERFILE"
```

#### Check and restart a service
```bash
#!/bin/bash
SERVICE="${1:-httpd}"

if ! systemctl is-active --quiet "$SERVICE"; then
    echo "$SERVICE is down. Starting..."
    systemctl start "$SERVICE" && echo "Started." || { echo "Failed!" >&2; exit 1; }
else
    echo "$SERVICE is running."
fi
```

---

## 3. Containers (Podman)

> **RHEL 9 Exam Note**: Uses **Podman**, not Docker. Podman is daemonless and rootless-capable.

### 3.1 Installation and Registry Login
```bash
dnf install container-tools -y

podman login registry.redhat.io
```

### 3.2 Image Operations
```bash
podman search httpd                                # Search
podman pull docker.io/library/httpd:latest         # Pull
podman pull registry.access.redhat.com/ubi9/ubi   # Pull UBI
podman images                                      # List images
podman rmi httpd:latest                            # Remove image
```

### 3.3 Running Containers
```bash
# Detached with port mapping
podman run -d --name myweb -p 8080:80 httpd

# With environment variables
podman run -d --name mydb \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=mydb \
  mysql:8.0

# With volume mount (NOTE :Z for SELinux!)
podman run -d --name myweb \
  -p 8080:80 \
  -v /webdata:/usr/local/apache2/htdocs:Z \
  httpd

# Auto-remove when done
podman run --rm ubi9/ubi echo "Hello"
```

> **CRITICAL**: Use `:Z` on volumes — it automatically sets the correct SELinux context!

### 3.4 Container Management
```bash
podman ps                    # Running containers
podman ps -a                 # All containers
podman stop/start/restart myweb
podman rm myweb              # Remove stopped
podman rm -f myweb           # Force remove running
podman logs myweb            # View logs
podman logs -f myweb         # Follow logs
podman exec -it myweb bash   # Shell into container
podman inspect myweb         # Detailed info
```

### 3.5 Building Images with Containerfile
```dockerfile
FROM ubi9/ubi-minimal

LABEL maintainer="admin@example.com"

RUN microdnf install -y httpd && microdnf clean all

COPY index.html /var/www/html/

EXPOSE 80

CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]
```

```bash
podman build -t myhttpd:v1 .              # Build
podman tag myhttpd:v1 myhttpd:latest      # Tag
podman push myhttpd:v1 registry.example.com/myhttpd:v1
```

### 3.6 Systemd Integration – Container as a Service (EXAM FAVOURITE!)

#### Root-level service
```bash
podman run -d --name myweb httpd
podman generate systemd --name myweb --files --new
mv container-myweb.service /etc/systemd/system/
systemctl daemon-reload
systemctl enable --now container-myweb
```

#### Rootless user service (start at boot without login)
```bash
mkdir -p ~/.config/systemd/user/
podman generate systemd --name myweb --files --new
mv container-myweb.service ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now container-myweb

# CRITICAL: enable linger so service starts at boot without login
loginctl enable-linger <username>
```

#### Quadlet (RHEL 9.3+ modern method)
```ini
# ~/.config/containers/systemd/myweb.container

[Unit]
Description=My Web Container

[Container]
Image=docker.io/library/httpd:latest
PublishPort=8080:80
Volume=/webdata:/usr/local/apache2/htdocs:Z

[Service]
Restart=always

[Install]
WantedBy=default.target
```
```bash
systemctl --user daemon-reload
systemctl --user start myweb
```

### 3.7 Networking and Volumes
```bash
# Networks
podman network ls
podman network create mynet
podman run -d --name db --network mynet mysql:8.0

# Volumes
podman volume create mydata
podman volume ls
podman run -d -v mydata:/data:Z myimage

# SELinux volume labels:
#   :z = shared between multiple containers
#   :Z = private to THIS container
```

---

## 4. SELinux

### 4.1 Core Concepts

| Concept   | Description                                              |
|-----------|----------------------------------------------------------|
| **Mode**  | Enforcing / Permissive / Disabled                        |
| **Label** | Every file, process, port has a security context         |
| **Type**  | Core of label, e.g., `httpd_t`, `httpd_sys_content_t`   |
| **Policy**| Rules defining what types can access what               |
| **AVC**   | Access Vector Cache – where denials are logged           |

#### Security Context Format
```
user:role:type:level
system_u:object_r:httpd_sys_content_t:s0
```

### 4.2 Mode Management
```bash
getenforce               # Check: Enforcing / Permissive / Disabled
sestatus                 # Detailed status

setenforce 0             # Temporarily Permissive
setenforce 1             # Temporarily Enforcing

# Permanently – edit /etc/selinux/config
SELINUX=enforcing        # enforcing | permissive | disabled
SELINUXTYPE=targeted
```

> **CAUTION**: Never disable SELinux on the exam! Use permissive to troubleshoot, then re-enable enforcing.

### 4.3 Viewing Contexts
```bash
ls -lZ /var/www/html/          # File contexts
ls -ldZ /var/www/html/         # Directory context
ps axZ | grep httpd            # Process context
stat -c %C /etc/passwd         # Quick file context
```

### 4.4 Setting File Contexts (Permanent!)
```bash
# Install tools if missing
dnf install policycoreutils-python-utils -y

# Add a permanent rule for a directory
semanage fcontext -a -t httpd_sys_content_t "/webdata(/.*)?"

# Modify existing rule
semanage fcontext -m -t httpd_sys_content_t "/webdata(/.*)?"

# List your custom rules
semanage fcontext -l | grep local

# ALWAYS apply the rules to the filesystem!
restorecon -Rv /webdata/
```

> **WARNING**: `chcon` is NOT persistent across `restorecon` or filesystem relabels. Always use `semanage fcontext` + `restorecon` for exam tasks!

#### Common File Context Types
| Type                       | Used For                           |
|----------------------------|------------------------------------|
| `httpd_sys_content_t`      | Apache web content (read-only)     |
| `httpd_sys_rw_content_t`   | Apache writable content            |
| `samba_share_t`            | Samba shares                       |
| `nfs_t`                    | NFS exports                        |
| `ssh_home_t`               | SSH authorized keys                |
| `user_home_t`              | Regular user home files            |

### 4.5 SELinux Booleans
```bash
getsebool -a                          # List all booleans
getsebool -a | grep httpd             # Filter by service
getsebool httpd_enable_homedirs       # Specific boolean

setsebool httpd_enable_homedirs on    # Temporary
setsebool -P httpd_enable_homedirs on # PERMANENT (-P flag!)

semanage boolean -l | grep httpd      # With descriptions
```

#### Common Booleans
| Boolean                        | Effect                                          |
|--------------------------------|-------------------------------------------------|
| `httpd_enable_homedirs`        | Allow Apache to serve ~/public_html             |
| `httpd_can_network_connect`    | Allow Apache outbound connections               |
| `httpd_can_network_connect_db` | Allow Apache → database connections            |
| `httpd_use_nfs`                | Allow Apache to serve NFS-mounted content       |
| `samba_enable_home_dirs`       | Allow Samba to share home directories           |
| `ftpd_anon_write`              | Allow anonymous FTP write                       |
| `nfs_export_all_rw`            | Allow NFS to export all filesystems rw          |

### 4.6 Port Labeling
```bash
semanage port -l                              # List all port labels
semanage port -l | grep "^http_port"          # Find HTTP ports

# Allow Apache on port 8888
semanage port -a -t http_port_t -p tcp 8888

# Modify existing
semanage port -m -t http_port_t -p tcp 8888

# Remove
semanage port -d -t http_port_t -p tcp 8888
```

### 4.7 Troubleshooting SELinux Denials

```bash
# Read AVC denials
ausearch -m avc -ts recent
ausearch -m avc -ts today

# Plain English explanation
ausearch -m avc -ts recent | audit2why

# Generate allow policy (last resort!)
ausearch -m avc -ts recent | audit2allow -M mypolicy
semodule -i mypolicy.pp

# Human-readable analysis
dnf install setroubleshoot-server -y
sealert -a /var/log/audit/audit.log
grep sealert /var/log/messages
```

#### Systematic Troubleshooting Flow
```
1. getenforce          → confirm Enforcing mode
2. audit2why           → identify what is being denied
3. Choose the fix:
   a. Wrong file context?  → semanage fcontext + restorecon
   b. Wrong boolean?       → setsebool -P <boolean> on
   c. Wrong port?          → semanage port -a -t <type> -p tcp <port>
   d. No standard fix?     → audit2allow -M + semodule -i (last resort)
```

### 4.8 Filesystem Relabeling
```bash
# Trigger full relabel on next boot
touch /.autorelabel && reboot

# Relabel specific path
restorecon -v /path/to/file
restorecon -Rv /path/to/directory/
```

---

## 5. Quick Reference Cheat Sheet

### NFS
```bash
dnf install nfs-utils
echo "/share host(rw,sync)" >> /etc/exports
exportfs -rav
systemctl enable --now nfs-server
firewall-cmd --permanent --add-service={nfs,rpc-bind,mountd} && firewall-cmd --reload
# Client
mount -t nfs server:/share /mnt
echo "server:/share /mnt nfs defaults,_netdev 0 0" >> /etc/fstab
# AutoFS
dnf install autofs && systemctl enable --now autofs
```

### Scripting
```bash
#!/bin/bash
[ -f file ]              # File test
[ "$a" == "$b" ]         # String compare
[ $a -eq $b ]            # Integer compare
for i in {1..10}; do ... done
while [ cond ]; do ... done
$(command)               # Command substitution
$(( expr ))              # Arithmetic
[ $? -eq 0 ]             # Success check
```

### Podman
```bash
podman pull image
podman run -d --name c -p 8080:80 -v /host:/cont:Z image
podman ps / podman ps -a
podman stop/start/restart/rm name
podman logs -f name
podman exec -it name bash
podman build -t name:tag .
podman generate systemd --name c --files --new
loginctl enable-linger user   # Boot without login
```

### SELinux
```bash
getenforce / setenforce 0|1
sestatus
ls -lZ / ps axZ
semanage fcontext -a -t TYPE "PATH(/.*)?"
restorecon -Rv /path
setsebool -P BOOLEAN on
semanage port -a -t TYPE -p tcp PORT
ausearch -m avc -ts recent | audit2why
```

---

## 6. Common Exam Pitfalls

| # | Pitfall                                     | Fix                                                       |
|---|---------------------------------------------|-----------------------------------------------------------|
| 1 | NFS mount lost after reboot                 | Use `/etc/fstab` with `_netdev` or autofs                 |
| 2 | Firewall blocking NFS                       | Add nfs, rpc-bind, mountd to firewall (permanent + reload)|
| 3 | SELinux blocking web content in custom dir  | `semanage fcontext` + `restorecon` (NOT just `chcon`)     |
| 4 | Podman user service not starting at boot    | `loginctl enable-linger <user>`                           |
| 5 | Volume mount denied by SELinux              | Add `:Z` to volume flag (`-v /host:/cont:Z`)              |
| 6 | Boolean reverts after reboot                | Use `setsebool -P` (persistent)                           |
| 7 | `semanage` command not found                | Install `policycoreutils-python-utils`                    |
| 8 | Script won't run                            | `chmod +x script.sh`; check shebang `#!/bin/bash`         |
| 9 | AutoFS not mounting                         | Restart autofs; check `/etc/auto.master` & map file syntax|
| 10| Port blocked by SELinux                     | `semanage port -a -t <type> -p tcp <port>`                |
| 11| Container service dies after logout         | `loginctl enable-linger <user>`                           |
| 12| NFS export changes not taking effect        | Always run `exportfs -rav` after editing `/etc/exports`   |

---

## 7. Practice Exam Tasks

### Task 1 – NFS Server and AutoFS Client
**Setup**: Configure `server1` to export `/nfsdata` read-write to `192.168.1.0/24`.
On `client1`, configure autofs to mount the share automatically at `/mnt/remote/nfsdata`.

**Key steps**:
- Server: install `nfs-utils`, create dir, edit `/etc/exports`, `exportfs -rav`, open firewall
- Client: install `autofs`, edit `/etc/auto.master`, create map file, `systemctl enable --now autofs`

---

### Task 2 – Shell Script: User Audit
**Write** `/usr/local/bin/useraudit.sh` that:
- Accepts a filename as argument (exit 1 if missing)
- Reads usernames line by line
- Creates users that don't exist (password: `RedHat1!`)
- Logs results to `/var/log/useraudit.log`
- Make the script executable

---

### Task 3 – Container Service (Rootless)
**As user `student`**:
- Pull `registry.access.redhat.com/ubi9/ubi`
- Run a container named `myservice` that executes `sleep infinity`
- Generate a systemd unit and enable it as a user service
- Ensure the container starts at boot **without** user login

**Key commands**:
```bash
podman run -d --name myservice ubi9/ubi sleep infinity
mkdir -p ~/.config/systemd/user
podman generate systemd --name myservice --files --new
mv container-myservice.service ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now container-myservice
loginctl enable-linger student
```

---

### Task 4 – SELinux: Custom Web Content Directory
**Configure** Apache to serve content from `/webfiles`.
- Create `/webfiles/index.html`
- Configure Apache `DocumentRoot /webfiles`
- Fix SELinux so Apache can read the content (without disabling SELinux)

**Key commands**:
```bash
semanage fcontext -a -t httpd_sys_content_t "/webfiles(/.*)?"
restorecon -Rv /webfiles/
```

---

### Task 5 – SELinux: Non-Standard Port
**Configure** Apache to listen on port `8888`.
- Update `httpd.conf` with `Listen 8888`
- Fix SELinux to allow port 8888 for Apache
- Open firewall port 8888

**Key commands**:
```bash
semanage port -a -t http_port_t -p tcp 8888
firewall-cmd --permanent --add-port=8888/tcp
firewall-cmd --reload
systemctl restart httpd
```

---

*Good luck on your RHCSA EX200! Key strategy: read tasks carefully, don't skip SELinux fixes, and always verify with `systemctl status` and `curl`/`mount` after configuration.*
