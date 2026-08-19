# RHCSA EX200 – Full Mock Exam
### Simulated 3-Hour Exam · 300 Points Total · Passing Score: 210 (70%)

---

> **READ BEFORE STARTING**
>
> - Total time: **3 hours**
> - Tasks are **independent** — a failed task does not prevent completing others
> - All changes must **survive a reboot** unless stated otherwise
> - Do **NOT** disable SELinux or the firewall at any point
> - The exam environment has **two VMs**: `server1` and `client1`
> - All tasks are graded automatically — exact outcomes are required
> - Where a password is required for a user, use: **`RedHat1!`** unless specified

---

## Environment Setup

Before you begin, note the following about your lab environment:

| Item            | Value                        |
|-----------------|------------------------------|
| Server Hostname | `server1.example.com`        |
| Client Hostname | `client1.example.com`        |
| Server IP       | `192.168.1.10`               |
| Client IP       | `192.168.1.20`               |
| Network         | `192.168.1.0/24`             |
| DNS Domain      | `example.com`                |
| Root Password   | `redhat`                     |

---

## Exam Tasks

---

### TASK 01 — Configure Networking and Hostname `[10 pts]`
**⏱ Estimated time: 5 minutes**

Perform the following on **server1**:

1. Set the system hostname to `server1.example.com` permanently
2. Ensure the primary network interface has the static IP address `192.168.1.10/24`
3. Set the default gateway to `192.168.1.1`
4. Set the DNS server to `8.8.8.8`
5. Confirm the system can ping `client1.example.com`

---

### TASK 02 — User and Group Management `[15 pts]`
**⏱ Estimated time: 10 minutes**

Perform the following on **server1**:

1. Create the following groups: `developers`, `sysadmins`
2. Create the following users with the specified settings:

| Username | Primary Group | Secondary Groups | UID  | Comment         |
|----------|---------------|------------------|------|-----------------|
| `alice`  | `developers`  | `sysadmins`      | 1501 | Alice Developer |
| `bob`    | `developers`  | _(none)_         | 1502 | Bob Developer   |
| `carol`  | `sysadmins`   | `developers`     | 1503 | Carol Sysadmin  |

3. Set the password for all three users to `RedHat1!`
4. Set the password expiry for `bob` so that the account password expires in **30 days**
5. Lock the account `carol` without deleting it

---

### TASK 03 — File Permissions and Ownership `[15 pts]`
**⏱ Estimated time: 10 minutes**

Perform the following on **server1**:

1. Create the directory `/projects/dev` owned by `alice` and group `developers`
2. Set permissions so that:
   - Owner has **read, write, execute**
   - Group has **read, write, execute**
   - Others have **no permissions**
3. Set the **setgid** bit on `/projects/dev` so all new files inherit the `developers` group
4. Create the directory `/projects/shared`
5. Set the **sticky bit** on `/projects/shared` so users can only delete their own files
6. Set permissions on `/projects/shared` to **rwxrwxrwx** + sticky bit

---

### TASK 04 — File Search and Archive `[10 pts]`
**⏱ Estimated time: 8 minutes**

Perform the following on **server1**:

1. Find all files under `/etc` that have been **modified in the last 3 days** and save the list to `/tmp/recent_etc.txt`
2. Find all files under `/usr` that are **larger than 5MB** and save to `/tmp/large_usr.txt`
3. Create a compressed archive of `/etc/ssh` named `/tmp/ssh_backup.tar.gz`
4. List the contents of the archive to verify it was created correctly

---

### TASK 05 — Software Management `[10 pts]`
**⏱ Estimated time: 8 minutes**

Perform the following on **server1**:

1. Install the package groups **"Development Tools"**
2. Install the individual packages: `httpd`, `wget`, `vim`
3. Ensure `httpd` is enabled to start at boot but is NOT currently running
4. Verify all installed packages and save the full list to `/tmp/installed_packages.txt`

---

### TASK 06 — Systemd Service Management `[10 pts]`
**⏱ Estimated time: 8 minutes**

Perform the following on **server1**:

1. Create a custom systemd service unit `/etc/systemd/system/cleanup.service` that:
   - Runs `/usr/local/bin/cleanup.sh` at startup (Type=oneshot)
   - Has description "System Cleanup Service"
   - Runs after `network.target`
2. Create the script `/usr/local/bin/cleanup.sh` that:
   - Deletes all files in `/tmp` older than 7 days
   - Logs "Cleanup complete" with a timestamp to `/var/log/cleanup.log`
3. Enable the service so it runs at boot
4. Test the service by starting it manually and verify the log

---

### TASK 07 — Scheduled Tasks (cron and at) `[10 pts]`
**⏱ Estimated time: 8 minutes**

Perform the following on **server1**:

1. Schedule a cron job for user `alice` that runs `/usr/local/bin/diskcheck.sh` every day at **2:30 AM**
2. Schedule a cron job as `root` that runs `dnf update -y` every **Sunday at 3:00 AM**
3. Schedule a one-time `at` job to write `"Maintenance window complete"` to `/tmp/maint.txt` **5 minutes from now**
4. Ensure the `crond` service is enabled and running

---

### TASK 08 — sudo Configuration `[10 pts]`
**⏱ Estimated time: 5 minutes**

Perform the following on **server1**:

1. Configure sudo so that all members of the `sysadmins` group can run **any command** as root **without a password**
2. Configure sudo so that user `bob` can run only `/usr/bin/systemctl` and `/usr/bin/journalctl` as root, **with password**
3. Verify by testing as `alice` (member of sysadmins): `sudo whoami` should return `root`

---

### TASK 09 — Storage: LVM Configuration `[20 pts]`
**⏱ Estimated time: 15 minutes**

Perform the following on **server1** (assume `/dev/sdb` is an available disk):

1. Create a physical volume on `/dev/sdb`
2. Create a volume group named `vg_data` using the physical volume
3. Create a logical volume named `lv_storage` of size **2G** in `vg_data`
4. Format `lv_storage` with the **XFS** filesystem
5. Create mount point `/mnt/storage` and mount it persistently via `/etc/fstab`
6. Extend `lv_storage` to **3G** (add 1G) without unmounting
7. Grow the XFS filesystem to use the new space
8. Verify the filesystem size reflects the expansion

---

### TASK 10 — NFS Server Configuration `[20 pts]`
**⏱ Estimated time: 15 minutes**

Perform the following on **server1**:

1. Install `nfs-utils` if not already installed
2. Create the directory `/nfsexports/team` owned by root with permissions `755`
3. Create the file `/nfsexports/team/welcome.txt` with content: `"NFS Share Ready"`
4. Export `/nfsexports/team` with the following settings:
   - Read-write access to `192.168.1.0/24`
   - Synchronous writes
   - Root squash enabled (default)
5. Configure the NFS server to start automatically at boot
6. Configure the firewall to allow NFS, rpc-bind, and mountd services

---

### TASK 11 — NFS Client with AutoFS `[20 pts]`
**⏱ Estimated time: 15 minutes**

Perform the following on **client1**:

1. Install `autofs` and `nfs-utils`
2. Configure autofs so that the NFS share `server1:/nfsexports/team` is automatically mounted at `/mnt/nfs/team` when accessed
3. The mount must use **read-write** access with **sync**
4. Configure autofs to start automatically at boot
5. Verify by accessing `/mnt/nfs/team` and confirming the `welcome.txt` file is visible
6. Verify the share **unmounts automatically** after being idle (you can check with `mount` before and after waiting 5 minutes, or just verify autofs config is correct)

---

### TASK 12 — Shell Script: System Health Report `[25 pts]`
**⏱ Estimated time: 20 minutes**

Write the script `/usr/local/bin/healthreport.sh` on **server1** with the following requirements:

1. The script must be executable by all users
2. It accepts an optional argument: a directory path to report on (default: `/`)
3. The report must include ALL of the following sections:
   - **Hostname and Date**: current hostname and timestamp
   - **Uptime**: system uptime
   - **CPU Load**: current load averages (1, 5, 15 min)
   - **Memory**: total, used, and free RAM (in MB)
   - **Disk Usage**: used/total/percent for the argument directory's filesystem
   - **Top 5 Processes**: top 5 processes by CPU usage (name and %CPU)
   - **Failed Services**: list of all systemd services in a `failed` state
4. Save the report to `/var/log/healthreport_YYYYMMDD.log` (dated filename)
5. Print `"Report saved to <filename>"` when complete
6. Exit with code `0` on success, `1` on any error

---

### TASK 13 — Shell Script: User Provisioning `[20 pts]`
**⏱ Estimated time: 15 minutes**

Write the script `/usr/local/bin/provision_users.sh` on **server1** with the following requirements:

1. Reads a CSV file passed as `$1` in the format: `username,group,password`
2. For each line:
   - Creates the group if it doesn't exist
   - Creates the user with that primary group if the user doesn't exist
   - Sets the user's password
   - Forces the user to change their password on first login
   - Logs each action to `/var/log/provision.log` with a timestamp
3. If a user already exists, log a skip message and continue
4. If the input file is missing or no argument is given, print usage and exit with code `1`
5. Make the script executable

**Test the script** with the following CSV file `/tmp/newusers.csv`:
```
dave,qa,DevPass1!
eve,qa,DevPass1!
frank,ops,OpsPass1!
```

---

### TASK 14 — Container: Run and Manage `[20 pts]`
**⏱ Estimated time: 15 minutes**

Perform the following on **server1** as user `alice`:

1. Pull the image `registry.access.redhat.com/ubi9/ubi-minimal`
2. Run a container named `reportgen` that:
   - Mounts `/var/log` from the host at `/hostlogs` inside the container (read-only)
   - Runs indefinitely (`sleep infinity`)
   - Is named `reportgen`
3. Verify the container can see files in `/hostlogs` from inside the container
4. Stop and remove the `reportgen` container
5. Run a new container named `webserver` using `docker.io/library/httpd:latest` that:
   - Binds host port `8888` to container port `80`
   - Mounts `/srv/webdata` (create if needed, add an `index.html`) to `/usr/local/apache2/htdocs` with proper SELinux labeling
6. Verify with `curl http://localhost:8888`

---

### TASK 15 — Container: Build a Custom Image `[20 pts]`
**⏱ Estimated time: 20 minutes**

Perform the following on **server1**:

1. Create a working directory `/opt/containerbuilds/sysinfo`
2. Create a shell script `sysinfo.sh` that prints: hostname, kernel version, and current date
3. Create a `Containerfile` that:
   - Uses `ubi9/ubi-minimal` as the base
   - Copies `sysinfo.sh` into `/usr/local/bin/`
   - Makes the script executable inside the image
   - Sets the default command to run `/usr/local/bin/sysinfo.sh`
4. Build the image and tag it as `sysinfo:1.0`
5. Run the image (non-detached) and verify it prints system information
6. Tag the image as `sysinfo:latest` as well

---

### TASK 16 — Container: Systemd Service `[20 pts]`
**⏱ Estimated time: 15 minutes**

Perform the following on **server1** as user `devops` (create the user first if needed):

1. Create user `devops` with home directory and shell `/bin/bash`
2. As `devops`, pull `docker.io/library/nginx:alpine`
3. Run a container named `nginxsvc` on host port `9090`, mapping to container port `80`
4. Generate a systemd **user** service unit for the container
5. Enable and start the user service
6. Configure the system so that `nginxsvc` starts automatically at boot **even when `devops` is not logged in**
7. Verify the service is active and the web server responds on port `9090`

---

### TASK 17 — SELinux: Context and Boolean Fixes `[20 pts]`
**⏱ Estimated time: 15 minutes**

Perform the following on **server1**:

**Part A:**
1. Create the directory `/webapps/portal` and add a file `index.html` with content `"SELinux Portal"`
2. Configure Apache's `DocumentRoot` to `/webapps/portal` (edit `/etc/httpd/conf/httpd.conf`)
3. Ensure SELinux allows Apache to read content from this custom directory
4. Start Apache and verify `curl http://localhost` returns the portal page

**Part B:**
1. Configure the system so that Apache can **send outbound network connections** (needed for reverse proxy)
2. Enable this using an SELinux boolean **permanently**
3. Verify the boolean is set

**Part C:**
1. Configure Apache to also listen on port **8080**
2. Add `Listen 8080` to the Apache config
3. Ensure SELinux allows Apache to use port `8080`
4. Open port `8080` in the firewall
5. Verify with `curl http://localhost:8080`

---

### TASK 18 — SELinux: Audit and Remediation `[25 pts]`
**⏱ Estimated time: 15 minutes**

Perform the following on **server1**:

**Scenario**: A script `/usr/local/bin/auditme.sh` has been created that attempts to:
- Write to `/var/log/myapp/app.log`
- Bind to port `8443`
- Serve files from `/opt/appdata`

**Your tasks:**

1. Create the directory `/var/log/myapp/` and the file `app.log`
2. Create the directory `/opt/appdata` and place a test file in it
3. Run `getenforce` and confirm SELinux is in **Enforcing** mode
4. Set the SELinux context on `/var/log/myapp/` to `var_log_t` **permanently**
5. Set the SELinux context on `/opt/appdata/` to `httpd_sys_content_t` **permanently**
6. Apply all context changes with `restorecon`
7. Add port `8443` as an allowed HTTPS port in SELinux (`https_port_t`)
8. Verify all three changes using the appropriate `semanage` and `ls -lZ` commands
9. Show that `ausearch -m avc -ts recent` returns no relevant denials after your fixes

---

## Scoring Summary

| Task | Topic                          | Points |
|------|--------------------------------|--------|
| 01   | Networking & Hostname          | 10     |
| 02   | Users & Groups                 | 15     |
| 03   | File Permissions & Special Bits| 15     |
| 04   | File Search & Archive          | 10     |
| 05   | Software Management            | 10     |
| 06   | Systemd Service                | 10     |
| 07   | Scheduling (cron/at)           | 10     |
| 08   | sudo Configuration             | 10     |
| 09   | LVM Storage                    | 20     |
| 10   | NFS Server                     | 20     |
| 11   | NFS Client / AutoFS            | 20     |
| 12   | Script: Health Report          | 25     |
| 13   | Script: User Provisioning      | 20     |
| 14   | Container: Run & Manage        | 20     |
| 15   | Container: Build Image         | 20     |
| 16   | Container: Systemd Service     | 20     |
| 17   | SELinux: Context & Booleans    | 20     |
| 18   | SELinux: Audit & Remediation   | 25     |
| **TOTAL** |                         | **300** |

**Passing Score: 210 / 300 (70%)**

---

## Time Management Guide

| Phase        | Tasks       | Target Time  |
|--------------|-------------|--------------|
| Quick wins   | 01–08       | 0:00 – 0:55  |
| Storage/NFS  | 09–11       | 0:55 – 1:45  |
| Scripting    | 12–13       | 1:45 – 2:25  |
| Containers   | 14–16       | 2:25 – 2:45  |
| SELinux      | 17–18       | 2:45 – 3:00  |

> **Strategy Tips**:
> - Do easy/familiar tasks first to bank points
> - Reboot once mid-exam (after tasks 1–9) to verify persistence
> - Always restart/enable services — a configured but disabled service scores 0
> - Never disable SELinux or firewalld — instant point loss on affected tasks

---

*When finished: type `reboot` and verify all persistent configurations survive. Good luck!*
