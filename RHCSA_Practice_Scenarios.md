# RHCSA EX200 – Practice Scenarios
### 20 Exam-Style Tasks with Full Solutions

> Each scenario is written in the style of the real exam.
> **Verify every task after completion** — points are awarded on outcome, not method.

---

## NFS Scenarios

---

### NFS-01: Export a Share to Multiple Clients with Different Permissions

**Task**
Configure the NFS server so that:
- `/exports/data` is exported **read-write** to host `192.168.1.50`
- `/exports/data` is exported **read-only** to the subnet `192.168.2.0/24`
- The NFS service starts automatically at boot
- The firewall allows NFS traffic

**Solution**
```bash
# Install nfs-utils
dnf install nfs-utils -y

# Create directory
mkdir -p /exports/data
chmod 755 /exports/data

# Configure exports
cat > /etc/exports << 'EOF'
/exports/data  192.168.1.50(rw,sync,no_root_squash)  192.168.2.0/24(ro,sync)
EOF

# Apply and enable
exportfs -rav
systemctl enable --now nfs-server

# Firewall
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-service=mountd
firewall-cmd --reload
```

**Verify**
```bash
exportfs -v                          # Should show both entries
showmount -e localhost               # Should list /exports/data
systemctl is-active nfs-server       # Should return: active
firewall-cmd --list-services         # Should include nfs rpc-bind mountd
```

---

### NFS-02: Persistent NFS Mount via /etc/fstab

**Task**
On the client machine, mount the NFS share `server1:/projects` at `/mnt/projects`.
The mount must:
- Persist across reboots
- Wait for the network before mounting
- Use read-write access

**Solution**
```bash
# Install client tools
dnf install nfs-utils -y

# Create mount point
mkdir -p /mnt/projects

# Verify the export is reachable
showmount -e server1

# Test mount manually first
mount -t nfs server1:/projects /mnt/projects

# If successful, unmount and add to fstab
umount /mnt/projects

# Add to /etc/fstab
echo "server1:/projects  /mnt/projects  nfs  defaults,_netdev,rw  0 0" >> /etc/fstab

# Mount all fstab entries to validate
mount -a

# Confirm
df -h | grep projects
```

**Verify**
```bash
mount | grep projects                # Should show nfs mount
df -h | grep projects                # Should show filesystem info
cat /etc/fstab | grep projects       # Should show entry with _netdev
```

---

### NFS-03: AutoFS with Direct Map

**Task**
Configure autofs on the client so that:
- `/mnt/direct/docs` automatically mounts `nfsserver:/shared/docs`
- Use a **direct map** configuration
- The mount is read-only

**Solution**
```bash
dnf install autofs -y

# Edit /etc/auto.master — add direct map reference
echo "/-  /etc/auto.direct" >> /etc/auto.master

# Create the direct map file
echo "/mnt/direct/docs  -ro,sync  nfsserver:/shared/docs" > /etc/auto.direct

# Create the mount point (autofs needs parent to exist for direct maps)
mkdir -p /mnt/direct/docs

systemctl enable --now autofs

# Test: accessing the path triggers the mount
ls /mnt/direct/docs
```

**Verify**
```bash
systemctl status autofs              # Should be active
ls /mnt/direct/docs                  # Should show contents
mount | grep docs                    # Should show nfs mount after access
```

---

### NFS-04: AutoFS with Wildcard (Home Directories)

**Task**
Configure autofs so that any user home directory under `/home/nfs` is automatically
mounted from `nfsserver:/home/<username>`.
Example: Accessing `/home/nfs/alice` mounts `nfsserver:/home/alice`.

**Solution**
```bash
dnf install autofs -y

# Add to /etc/auto.master
echo "/home/nfs  /etc/auto.home" >> /etc/auto.master

# Create map file with wildcard
cat > /etc/auto.home << 'EOF'
*  -rw,sync  nfsserver:/home/&
EOF
# Note: * is the wildcard key, & substitutes the matched key

systemctl enable --now autofs

# Test
ls /home/nfs/alice    # Mounts nfsserver:/home/alice
ls /home/nfs/bob      # Mounts nfsserver:/home/bob
```

**Verify**
```bash
automount -m                          # Show all automount maps
ls /home/nfs/alice                    # Trigger mount
mount | grep nfs | grep alice         # Should show the mount
```

---

### NFS-05: Diagnose and Fix a Broken NFS Mount

**Task**
A developer reports that `/mnt/appdata` is not mounting the NFS share from `nfsserver`.
Diagnose and fix the issue. The expected share is `nfsserver:/appdata`.

**Solution — Systematic Diagnosis**
```bash
# Step 1: Check if nfsserver is reachable
ping -c 2 nfsserver

# Step 2: Check if NFS service is running on server (from server)
systemctl status nfs-server

# Step 3: Check exports on server
showmount -e nfsserver           # If this fails, check firewall

# Step 4: Check firewall on server
firewall-cmd --list-services     # If nfs/rpc-bind/mountd missing, add them

# Step 5: Check fstab on client
cat /etc/fstab | grep appdata

# Step 6: Check if mount point exists
ls -ld /mnt/appdata              # Create if missing: mkdir -p /mnt/appdata

# Step 7: Try manual mount and check error
mount -t nfs nfsserver:/appdata /mnt/appdata 2>&1

# Step 8: Check if autofs is configured if manual doesn't match fstab
systemctl status autofs

# Fix common issues
mkdir -p /mnt/appdata
exportfs -rav                    # On server if exports missing
systemctl restart nfs-server     # On server
mount -a                         # On client to apply fstab
```

**Verify**
```bash
mount | grep appdata
df -h /mnt/appdata
```

---

## Shell Scripting Scenarios

---

### SCR-01: Disk Usage Alert Script

**Task**
Write a script `/usr/local/bin/diskcheck.sh` that:
- Checks disk usage for every mounted filesystem
- Prints a **WARNING** line for any filesystem over 80% full
- Prints an **OK** line for filesystems under 80%
- Exits with code `1` if any filesystem is over 80%, else exits `0`

**Solution**
```bash
cat > /usr/local/bin/diskcheck.sh << 'SCRIPT'
#!/bin/bash
# Disk usage alert script

THRESHOLD=80
ALERT=0

while IFS= read -r LINE; do
    # Skip header line
    echo "$LINE" | grep -qE "^Filesystem" && continue

    USAGE=$(echo "$LINE" | awk '{print $5}' | tr -d '%')
    MOUNT=$(echo "$LINE" | awk '{print $6}')

    if [ "$USAGE" -ge "$THRESHOLD" ]; then
        echo "WARNING: $MOUNT is at ${USAGE}% capacity"
        ALERT=1
    else
        echo "OK:      $MOUNT is at ${USAGE}% capacity"
    fi
done < <(df -h --output=source,size,used,avail,pcent,target | tail -n +2)

exit $ALERT
SCRIPT

chmod +x /usr/local/bin/diskcheck.sh
```

**Verify**
```bash
/usr/local/bin/diskcheck.sh
echo "Exit code: $?"
```

---

### SCR-02: Backup Script with Timestamps

**Task**
Write a script `/usr/local/bin/backup.sh` that:
- Accepts a source directory as `$1` and backup destination as `$2`
- Creates a compressed tarball named `backup_YYYYMMDD_HHMMSS.tar.gz` in the destination
- Validates that both arguments are provided and that the source directory exists
- Prints the size and path of the created backup

**Solution**
```bash
cat > /usr/local/bin/backup.sh << 'SCRIPT'
#!/bin/bash
# Backup script with timestamps

SRCDIR="$1"
DESTDIR="$2"

# Validate arguments
if [ $# -lt 2 ]; then
    echo "Usage: $0 <source_dir> <dest_dir>" >&2
    exit 1
fi

if [ ! -d "$SRCDIR" ]; then
    echo "ERROR: Source directory '$SRCDIR' does not exist." >&2
    exit 1
fi

if [ ! -d "$DESTDIR" ]; then
    mkdir -p "$DESTDIR" || { echo "ERROR: Cannot create '$DESTDIR'" >&2; exit 1; }
fi

# Create backup
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$DESTDIR/backup_${TIMESTAMP}.tar.gz"

tar -czf "$BACKUP_FILE" -C "$(dirname "$SRCDIR")" "$(basename "$SRCDIR")"

if [ $? -eq 0 ]; then
    SIZE=$(du -sh "$BACKUP_FILE" | awk '{print $1}')
    echo "Backup created: $BACKUP_FILE (Size: $SIZE)"
else
    echo "ERROR: Backup failed!" >&2
    exit 1
fi
SCRIPT

chmod +x /usr/local/bin/backup.sh
```

**Verify**
```bash
/usr/local/bin/backup.sh /etc /tmp/backups
ls -lh /tmp/backups/
```

---

### SCR-03: Process Monitor and Restart

**Task**
Write a script `/usr/local/bin/procwatch.sh` that:
- Accepts a process name as `$1`
- Checks if the process is running
- If not running, attempts to start it using `systemctl start <name>`
- Logs all actions with timestamps to `/var/log/procwatch.log`

**Solution**
```bash
cat > /usr/local/bin/procwatch.sh << 'SCRIPT'
#!/bin/bash
# Process monitor and restart script

SERVICE="${1}"
LOGFILE="/var/log/procwatch.log"

if [ -z "$SERVICE" ]; then
    echo "Usage: $0 <service_name>" >&2
    exit 1
fi

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$LOGFILE"
}

if systemctl is-active --quiet "$SERVICE"; then
    log "OK: $SERVICE is running."
else
    log "WARNING: $SERVICE is NOT running. Attempting to start..."
    systemctl start "$SERVICE"
    sleep 2
    if systemctl is-active --quiet "$SERVICE"; then
        log "SUCCESS: $SERVICE started successfully."
    else
        log "ERROR: Failed to start $SERVICE."
        exit 1
    fi
fi
SCRIPT

chmod +x /usr/local/bin/procwatch.sh
```

**Verify**
```bash
/usr/local/bin/procwatch.sh sshd
cat /var/log/procwatch.log
```

---

### SCR-04: Parse and Report from /etc/passwd

**Task**
Write a script `/usr/local/bin/userreport.sh` that:
- Reads `/etc/passwd`
- Lists only users with a **UID >= 1000** (regular users, not system accounts)
- Outputs: `Username | UID | Home Directory | Shell`
- Formats as a table and saves to `/tmp/userreport.txt`

**Solution**
```bash
cat > /usr/local/bin/userreport.sh << 'SCRIPT'
#!/bin/bash
# User report script

OUTFILE="/tmp/userreport.txt"

printf "%-20s %-8s %-30s %-20s\n" "USERNAME" "UID" "HOME" "SHELL" > "$OUTFILE"
printf "%-20s %-8s %-30s %-20s\n" "--------" "---" "----" "-----" >> "$OUTFILE"

while IFS=: read -r USERNAME _ UID _ _ HOME SHELL; do
    if [ "$UID" -ge 1000 ] 2>/dev/null; then
        printf "%-20s %-8s %-30s %-20s\n" \
            "$USERNAME" "$UID" "$HOME" "$SHELL" >> "$OUTFILE"
    fi
done < /etc/passwd

echo "Report saved to $OUTFILE"
cat "$OUTFILE"
SCRIPT

chmod +x /usr/local/bin/userreport.sh
```

**Verify**
```bash
/usr/local/bin/userreport.sh
cat /tmp/userreport.txt
```

---

### SCR-05: Bulk File Renaming Script

**Task**
Write a script `/usr/local/bin/renamefiles.sh` that:
- Accepts a directory as `$1` and an extension as `$2` (e.g., `.txt`)
- Renames all files with that extension by prepending `archived_` to the filename
- Skips files that already start with `archived_`
- Prints each rename operation

**Solution**
```bash
cat > /usr/local/bin/renamefiles.sh << 'SCRIPT'
#!/bin/bash
# Bulk file rename script

TARGETDIR="${1}"
EXT="${2}"

if [ $# -lt 2 ]; then
    echo "Usage: $0 <directory> <extension>" >&2
    exit 1
fi

if [ ! -d "$TARGETDIR" ]; then
    echo "ERROR: '$TARGETDIR' is not a directory" >&2
    exit 1
fi

COUNT=0
for FILE in "$TARGETDIR"/*"$EXT"; do
    [ -f "$FILE" ] || continue    # Skip if no matches
    BASENAME=$(basename "$FILE")

    # Skip already renamed
    if [[ "$BASENAME" == archived_* ]]; then
        echo "SKIP: $BASENAME (already renamed)"
        continue
    fi

    NEWNAME="$TARGETDIR/archived_$BASENAME"
    mv "$FILE" "$NEWNAME"
    echo "RENAMED: $BASENAME → archived_$BASENAME"
    (( COUNT++ ))
done

echo "Done. $COUNT file(s) renamed."
SCRIPT

chmod +x /usr/local/bin/renamefiles.sh
```

**Verify**
```bash
mkdir /tmp/testfiles
touch /tmp/testfiles/{file1.txt,file2.txt,report.txt}
/usr/local/bin/renamefiles.sh /tmp/testfiles .txt
ls /tmp/testfiles/
```

---

## Container Scenarios (Podman)

---

### CON-01: Run a MariaDB Container with Persistent Storage

**Task**
Run a MariaDB container named `mydb` that:
- Uses image `docker.io/library/mariadb:latest`
- Sets root password to `RedHat1!` and creates database `appdb`
- Stores database files in `/dbdata` on the host (persist across restarts)
- Binds to port `3306` on the host
- Restarts automatically on failure

**Solution**
```bash
# Prepare host storage
mkdir -p /dbdata
chown 27:27 /dbdata         # mysql user UID inside container
chcon -t container_file_t /dbdata  # Or use :Z label below

# Run the container
podman run -d \
  --name mydb \
  -e MYSQL_ROOT_PASSWORD=RedHat1! \
  -e MYSQL_DATABASE=appdb \
  -p 3306:3306 \
  -v /dbdata:/var/lib/mysql:Z \
  --restart=on-failure \
  docker.io/library/mariadb:latest
```

**Verify**
```bash
podman ps                             # mydb should be Up
podman logs mydb | tail -20           # Should show "ready for connections"
podman exec -it mydb mariadb -u root -pRedHat1! -e "SHOW DATABASES;"
ls /dbdata/                           # Should contain DB files
```

---

### CON-02: Build a Custom Container Image

**Task**
Create a custom container image named `mywebapp:1.0` that:
- Is based on `ubi9/ubi-minimal`
- Installs `nginx`
- Copies a custom `index.html` into `/usr/share/nginx/html/`
- Exposes port `80`
- Starts nginx in the foreground

**Solution**
```bash
# Create working directory
mkdir -p /build/mywebapp
cd /build/mywebapp

# Create index.html
echo "<h1>RHCSA Practice - Custom Container</h1>" > index.html

# Create Containerfile
cat > Containerfile << 'EOF'
FROM registry.access.redhat.com/ubi9/ubi-minimal

RUN microdnf install -y nginx && microdnf clean all

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
EOF

# Build the image
podman build -t mywebapp:1.0 .

# Test it
podman run -d --name testapp -p 8080:80 mywebapp:1.0
curl http://localhost:8080
```

**Verify**
```bash
podman images | grep mywebapp         # Should show mywebapp:1.0
podman ps | grep testapp              # Should be running
curl http://localhost:8080            # Should return HTML
```

---

### CON-03: Rootless Container as a Systemd User Service

**Task**
As user `devops`:
- Pull `docker.io/library/nginx:alpine`
- Run a container named `webfront` on port `8080`
- Configure it as a **user systemd service** that:
  - Starts automatically at boot
  - Does NOT require `devops` to be logged in

**Solution**
```bash
# Run as devops user
su - devops

# Pull the image
podman pull docker.io/library/nginx:alpine

# Run the container
podman run -d --name webfront -p 8080:80 nginx:alpine

# Create systemd user directory
mkdir -p ~/.config/systemd/user/

# Generate the service file
podman generate systemd --name webfront --files --new
mv container-webfront.service ~/.config/systemd/user/

# Enable and start
systemctl --user daemon-reload
systemctl --user enable --now container-webfront

# Exit back to root
exit

# Enable linger (allows user services to run without login)
loginctl enable-linger devops
```

**Verify**
```bash
# As devops user:
systemctl --user status container-webfront    # Should be active
# As root:
loginctl show-user devops | grep Linger       # Linger=yes
curl http://localhost:8080                    # Should return nginx page
```

---

### CON-04: Multi-Container Setup with a Shared Network

**Task**
Create two containers:
- `frontend`: runs `nginx:alpine`, mapped to host port `8080`
- `backend`: runs `ubi9/ubi` executing `sleep infinity`

Both containers must be on the same custom network `appnet` so they can communicate.
Verify `frontend` can reach `backend` by name.

**Solution**
```bash
# Create custom network
podman network create appnet

# Run backend
podman run -d \
  --name backend \
  --network appnet \
  registry.access.redhat.com/ubi9/ubi \
  sleep infinity

# Run frontend
podman run -d \
  --name frontend \
  --network appnet \
  -p 8080:80 \
  nginx:alpine

# Verify DNS resolution between containers
podman exec frontend ping -c 2 backend
```

**Verify**
```bash
podman ps                                     # Both containers up
podman network inspect appnet                 # Both containers listed
podman exec frontend ping -c 2 backend        # Should succeed (DNS by name)
```

---

### CON-05: Export, Transfer, and Import a Container Image

**Task**
Export the `mywebapp:1.0` image to a tar file `/tmp/mywebapp.tar`,
then import it on the same system under a new name `mywebapp:imported`.

**Solution**
```bash
# Save (export) the image to a tar file
podman save -o /tmp/mywebapp.tar mywebapp:1.0

# Verify the file exists
ls -lh /tmp/mywebapp.tar

# Load (import) with new name
podman load -i /tmp/mywebapp.tar

# Tag it with a new name
podman tag mywebapp:1.0 mywebapp:imported

# Verify
podman images | grep mywebapp
```

**Alternative: Export/import running container filesystem**
```bash
# Export running container filesystem (different from image save)
podman export mydb > /tmp/mydb_fs.tar

# Import as a new image
podman import /tmp/mydb_fs.tar mydb:backup
podman images | grep mydb
```

**Verify**
```bash
podman images | grep mywebapp             # Shows both :1.0 and :imported
podman run --rm mywebapp:imported echo "works"
```

---

## SELinux Scenarios

---

### SEL-01: Fix SELinux Context for Custom Apache DocumentRoot

**Task**
Apache has been configured to serve files from `/srv/webroot` instead of `/var/www/html`.
The web server starts but returns **403 Forbidden** for all requests.
Fix the problem without disabling SELinux.

**Solution**
```bash
# Step 1: Verify Apache is running
systemctl status httpd

# Step 2: Check SELinux is enforcing
getenforce

# Step 3: Check current context of /srv/webroot
ls -ldZ /srv/webroot
# Likely shows: system_u:object_r:var_t:s0 (WRONG!)
# Should be:    httpd_sys_content_t

# Step 4: Check audit log for denials
ausearch -m avc -ts recent | audit2why

# Step 5: Install tools if needed
dnf install policycoreutils-python-utils -y

# Step 6: Set the correct context permanently
semanage fcontext -a -t httpd_sys_content_t "/srv/webroot(/.*)?"

# Step 7: Apply the context
restorecon -Rv /srv/webroot/

# Step 8: Verify new context
ls -ldZ /srv/webroot
ls -lZ /srv/webroot/

# Step 9: Test
curl http://localhost
```

**Verify**
```bash
ls -lZ /srv/webroot/                       # Shows httpd_sys_content_t
curl -s -o /dev/null -w "%{http_code}" http://localhost  # Should return 200
ausearch -m avc -ts recent                  # Should be empty (no new denials)
```

---

### SEL-02: Enable Home Directory Web Serving

**Task**
A user named `webdev` has a `public_html` directory in their home (`/home/webdev/public_html`).
Apache's `UserDir` module is enabled, but requests to `http://server/~webdev/` return **403 Forbidden**.
Fix this using SELinux without disabling SELinux.

**Solution**
```bash
# Step 1: Check the boolean needed
getsebool httpd_enable_homedirs          # Returns: off

# Step 2: Check audit log
ausearch -m avc -ts recent | audit2why   # Will mention httpd_enable_homedirs

# Step 3: Enable the boolean permanently
setsebool -P httpd_enable_homedirs on

# Step 4: Also ensure the directory context is correct
ls -ldZ /home/webdev/public_html
# Should show: user_home_t or httpd_user_content_t

# If context is wrong:
chcon -t httpd_user_content_t /home/webdev/public_html
# For persistent fix:
semanage fcontext -a -t httpd_user_content_t "/home/webdev/public_html(/.*)?"
restorecon -Rv /home/webdev/public_html

# Step 5: Check execute permission on home dir
chmod o+x /home/webdev
```

**Verify**
```bash
getsebool httpd_enable_homedirs            # Should return: on
curl http://localhost/~webdev/             # Should return 200
```

---

### SEL-03: Configure SELinux for a Non-Standard SSH Port

**Task**
Configure SSH to listen on port **2222** in addition to the default port 22.
Ensure SELinux allows this without disabling it.
Open the new port in the firewall.

**Solution**
```bash
# Step 1: Check what SELinux thinks is allowed for SSH
semanage port -l | grep ssh
# Typically: ssh_port_t  tcp  22

# Step 2: Add port 2222 to allowed SSH ports
semanage port -a -t ssh_port_t -p tcp 2222

# Step 3: Verify
semanage port -l | grep ssh
# Should show: ssh_port_t  tcp  2222, 22

# Step 4: Edit /etc/ssh/sshd_config
sed -i '/^#Port 22/a Port 2222' /etc/ssh/sshd_config
# Or manually add: Port 2222

# Step 5: Restart sshd
systemctl restart sshd

# Step 6: Open firewall
firewall-cmd --permanent --add-port=2222/tcp
firewall-cmd --reload
```

**Verify**
```bash
semanage port -l | grep ssh           # Shows 2222 and 22
ss -tlnp | grep ':2222'              # SSH listening on 2222
firewall-cmd --list-ports            # Should include 2222/tcp
ssh -p 2222 localhost                 # Should prompt for login
```

---

### SEL-04: Diagnose SELinux Denial with audit2why

**Task**
A script attempts to connect to a database from Apache (`httpd` process), but the connection fails.
No firewall rules are blocking the traffic. Diagnose and resolve the SELinux issue.

**Solution**
```bash
# Step 1: Check for AVC denials
ausearch -m avc -ts recent

# Step 2: Get plain-English explanation
ausearch -m avc -ts recent | audit2why
# Expected output suggests: setsebool httpd_can_network_connect_db on

# Step 3: Check the boolean state
getsebool httpd_can_network_connect_db   # Returns: off

# Step 4: Enable it permanently
setsebool -P httpd_can_network_connect_db on

# Step 5: Also check general outbound network if needed
getsebool httpd_can_network_connect
setsebool -P httpd_can_network_connect on    # Only if httpd_can_network_connect_db isn't enough

# Step 6: Verify
getsebool httpd_can_network_connect_db    # Should return: on
```

**Verify**
```bash
getsebool httpd_can_network_connect_db   # on
ausearch -m avc -ts recent               # Should show no new denials after fix
# Test: trigger the application's DB call and verify it succeeds
```

---

### SEL-05: Restore Default SELinux Contexts After Mass Copy

**Task**
A sysadmin copied `/var/www/html` contents to a new location `/var/website` using `cp -a`.
The `cp -a` preserved the original SELinux contexts but now Apache cannot read the files
because the directory `/var/website` has the wrong context.
Fix the context on `/var/website` permanently and apply it.

**Solution**
```bash
# Step 1: View current (wrong) context
ls -ldZ /var/website
# Might show: system_u:object_r:default_t:s0

# Step 2: View expected context (from /var/www/html for reference)
ls -ldZ /var/www/html
# Shows: system_u:object_r:httpd_sys_content_t:s0

# Step 3: Check audit log
ausearch -m avc -ts recent | audit2why

# Step 4: Install semanage if needed
dnf install policycoreutils-python-utils -y

# Step 5: Add permanent policy for /var/website
semanage fcontext -a -t httpd_sys_content_t "/var/website(/.*)?"

# Step 6: Verify the rule was added
semanage fcontext -l | grep website

# Step 7: Apply to filesystem
restorecon -Rv /var/website/

# Step 8: Confirm
ls -lZ /var/website/
```

**Verify**
```bash
ls -lZ /var/website/                   # Should show httpd_sys_content_t
semanage fcontext -l | grep website    # Shows the custom rule
curl http://localhost                  # Should serve content from /var/website
```

---

## Bonus: Mixed Scenario

---

### MIX-01: Full Stack – NFS + SELinux + Containers

**Task**
1. On `server1`, export `/data/shared` read-write to `192.168.1.0/24` via NFS
2. On `client1`, mount `/data/shared` at `/mnt/shared` persistently
3. Run a container that serves `/mnt/shared` via HTTP on port `9090`
4. Ensure SELinux is not blocking the container's volume mount
5. Configure the container as a systemd service that starts at boot

**Solution**
```bash
### SERVER1 ###
dnf install nfs-utils -y
mkdir -p /data/shared
echo "Hello from NFS" > /data/shared/index.html
echo "/data/shared  192.168.1.0/24(rw,sync)" >> /etc/exports
exportfs -rav
systemctl enable --now nfs-server
firewall-cmd --permanent --add-service={nfs,rpc-bind,mountd}
firewall-cmd --reload

### CLIENT1 ###
dnf install nfs-utils -y
mkdir -p /mnt/shared
echo "server1:/data/shared  /mnt/shared  nfs  defaults,_netdev  0 0" >> /etc/fstab
mount -a

# Verify NFS mount works
ls /mnt/shared

# Run container with NFS mount (using :z for shared SELinux label)
podman run -d \
  --name nfsweb \
  -p 9090:80 \
  -v /mnt/shared:/usr/local/apache2/htdocs:z \
  docker.io/library/httpd:latest

# Verify
curl http://localhost:9090

# Create systemd service
mkdir -p ~/.config/systemd/user/
podman generate systemd --name nfsweb --files --new
mv container-nfsweb.service ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now container-nfsweb
loginctl enable-linger $(whoami)
```

**Verify**
```bash
mount | grep shared                          # NFS mounted
podman ps | grep nfsweb                      # Container running
curl http://localhost:9090                   # Returns "Hello from NFS"
systemctl --user status container-nfsweb    # Active
loginctl show-user $(whoami) | grep Linger  # Linger=yes
```

---

## Verification Command Reference

| Area       | Verify Command                                             |
|------------|------------------------------------------------------------|
| NFS export | `exportfs -v` / `showmount -e localhost`                   |
| NFS mount  | `mount \| grep nfs` / `df -h`                             |
| AutoFS     | `automount -m` / `ls <path>` to trigger mount              |
| Script     | `bash -x script.sh` to trace / `echo $?` for exit code    |
| Container  | `podman ps` / `podman logs <name>` / `curl localhost:<port>`|
| Service    | `systemctl status <name>` / `systemctl --user status <name>`|
| SELinux    | `ls -lZ` / `ausearch -m avc -ts recent` / `getsebool`     |
| Firewall   | `firewall-cmd --list-all`                                  |
| Linger     | `loginctl show-user <user> \| grep Linger`                 |

---

*Tip: On the real exam, always run a verification command after each task before moving on. Partial credit is not awarded — the system is checked by scripts that expect exact outcomes.*
