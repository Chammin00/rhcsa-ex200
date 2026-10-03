# RHCSA EX200 (RHEL 10) Complete Mock Exam — 22 Tasks
*Source: codeXchange RHCSA v10 Exam Practice (Video: kBsMuA-DA9E)*

---

## Exam Environment & General Information

### System Overview
| System | FQDN | IP Address | Netmask | Gateway | Nameserver |
|---|---|---|---|---|---|
| **System 1 (Primary)** | `node1.domain.example.com` | `172.25.250.10` | `255.255.255.0` | `172.25.250.254` | `172.25.250.220` |
| **System 2 (Secondary)** | `node2.domain.example.com` | `172.25.250.11` | `255.255.255.0` | `172.25.250.254` | `172.25.250.220` |

### Default Credentials
- **Root Password (node1):** `atenorth`
- **Default User/Service Password:** `atenorth` (unless otherwise specified)
- **Container Registry Access:**
  - **Username:** `admin`
  - **Password:** `admin123`

### Work Allocation
- **Tasks 01 – 15:** Perform on **`node1.domain.example.com`**
- **Tasks 16 – 22:** Perform on **`node2.domain.example.com`**

---

## Node 1 Tasks (`node1.domain.example.com`)

### Task 1: Environment & System Configuration Review
- Verify connectivity to the classroom network subnet (`172.25.250.0/24`).
- Ensure root access with password `atenorth`.

### Task 2: Network Configuration
Configure the network settings for the primary machine:
- **Hostname:** `primary.net1.example.com`
- **IP Address:** `172.25.250.11` (or assigned IP)
- **Netmask:** `255.255.255.0`
- **Gateway:** `172.25.250.254`
- **Nameserver:** `172.25.250.220`
- Settings must persist across system reboots.

### Task 3: Configure Default YUM / DNF Repositories
YUM repositories have been made available from:
- `http://content.example.com/rhel10.0/x86_64/dvd/BaseOS`
- `http://content.example.com/rhel10.0/x86_64/dvd/AppStream`
Configure your system to use these locations as default repositories.

### Task 4: Configure SELinux & Non-Standard Web Server Port
- The HTTP service should serve on non-standard port **82** for your machine.
- The system is not able to connect to `httpd` service at port 82; fix the debug/SELinux issue.
- Store the HTML files under `/var/www/html` directory (do not change the docroot).
- Ensure the service is accessible at port 82, starts at boot time, and firewall allows traffic.
- Verify locally: `curl http://localhost:82/file1`

### Task 5: User and Group Administration
Configure the following users and groups:
- Create a group named **`sysadmin`**.
- Create a user **`natasha`** who belongs to `sysadmin` as a secondary group.
- Create a user **`harry`** who belongs to `sysadmin` as a secondary group.
- Create a user **`sarah`** who does not have access to an interactive shell and is not a member of `sysadmin`.
- All above-mentioned users must have the password set to **`atenorth`**.

### Task 6: Collaborative Directory Permissions
Create a collaborative directory **`/common/admin`** with the following characteristics:
- Group ownership of `/common/admin` is **`sysadmin`**.
- The directory should be readable, writable, and accessible to members of `sysadmin`, but not to any other user. (Root has full access).
- Files created in `/common/admin` automatically have group ownership set to the `sysadmin` group (SGID).

### Task 7: Scheduled Cron Tasks
- **a.** The user **`natasha`** must configure a cron job that runs daily at **14:23** local time and executes `/bin/echo hello`.
- **b.** The user **`harry`** must configure a cron job that runs **every 3 minutes** and executes `/usr/bin/logger` with the message `"EX200 in progress"`.

### Task 8: Create an Archive & Backup
- Create a compressed archive named **`backup.tar.bz2`** of the **`/etc`** directory.
- Store the archive in the **`/home`** directory (`/home/backup.tar.bz2`).

### Task 9: Configure NTP Client
- Configure NTP on your system so that it is an NTP client of **`classroom.example.com`**.
- Ensure the `chronyd` service is active and enabled at boot.

### Task 10: Find and Copy Files
- Find all files in the system owned by user **`simone`** (or `Simone`).
- Copy all located files into the **`/root/found`** directory. Preserve attributes.

### Task 11: Find Matching String
- Find the string **`strato`** from **`/usr/share/dict/words`**.
- Save the exact matching output into **`/searchfile`**.

### Task 12: Configure Autofs NFS Automounting
Configure `autofs` to automount the home directories of user `remoteuser2`:
- `remoteuser2` home directory is exported via NFS on **`workstation.lab.example.com`** (`172.25.250.9`).
- NFS export path: **`/ourhome/remoteuser2`**
- `remoteuser2`'s home directory should be automatically mounted upon access using the `autofs` service.
- The home directory must be writable by the user.

### Task 13: Create User with Specific UID
- Create a user named **`barry`**.
- The User ID (UID) of this user must be **`2112`**.
- Set the user password to **`atenorth`**.

### Task 14: System Security, Scripting, and Application Rebuild
- **a. Create a Search Script:**
  - Create a script named **`newsearch`** located in **`/usr/local/bin/newsearch`**.
  - The script must locate all files under **`/usr`** that:
    - Are larger than **30k** and smaller than **50k** in size.
    - Have the **SUID** permission bit set (`-perm -4000`).
  - When executed, `newsearch` must save the list of found files into **`/root/myoutput`**.
  - Make the script executable.
- **b. Set Default Permissions (Umask):**
  - Set default permissions for user **`alex`** for all newly created files and folders so that new files have mode `r--r--r--` (`umask 0222`).
- **c. Sudo Administrative Privileges:**
  - Create a group named **`elite`**.
  - Members of `elite` must be granted administrative sudo permissions without requiring a password.
- **d. Password Aging Policy:**
  - Configure the default password policy for all newly created users to require password changes after **20 days** (`PASS_MAX_DAYS 20`).
- **e. Application Rebuild:**
  - Rebuild/configure an application **`ex200`** that outputs a specific completion message when user **`aslan`** logs in and runs `ex200`.

### Task 15: Systemd Timer Service and File ACLs
- **a. Logger Script with Systemd Timer:**
  - Install the package `logger` from the `timer.repo` repository.
  - Modify the script `/usr/local/bin/logger` to collect logs within `/var/tmp/ex200` and save the output to `/root/lookup_directory/ex200`.
  - Create a custom systemd service (`logger.service`) to run this script.
  - Create a systemd timer (`logger.timer`) to trigger the service **every 1 minute**.
  - Enable and start the service and timer so logs are updated automatically.
- **b. Controlling File Access Permissions with ACLs:**
  - Copy `/etc/fstab` to `/var/tmp/fstab`.
  - The file `/var/tmp/fstab` must be owned by `root:root`.
  - The file `/var/tmp/fstab` must not be executable by anyone.
  - User **`natasha`** must have **read and write** permissions (`rw-`).
  - User **`harry`** must have **no permissions** (`---`).
  - All other current and future users must have **read** permissions (`r--`).

---

## Node 2 Tasks (`node2.domain.example.com`)

### Task 16: Reset Forgotten Root Password
- Reboot `node2.domain.example.com` into GRUB emergency/debug mode (`rd.break`).
- Reset the root password to **`atenorth`**.
- Ensure SELinux relabeling completes successfully so the system boots cleanly.

### Task 17: Configure Default Repositories on Node 2
- Configure default YUM/DNF repositories on node2 pointing to:
  - BaseOS: `http://content.example.com/rhel10.0/x86_64/dvd/BaseOS`
  - AppStream: `http://content.example.com/rhel10.0/x86_64/dvd/AppStream`
- Verify that `dnf repolist` functions properly.

### Task 18: Configure Recommended Tuned Profile
- Identify and activate the recommended `tuned` profile for this virtualized system.
- Verify that the active profile matches the recommendation.

### Task 19: User-Specific Flatpak Repository & Application
- Configure a Flatpak remote repository named **`extra`** that is available **only to the user `bammbamm`**.
- Using the `extra` repository, install the Flatpak application **`codium`** under the user's scope.
- Verify that the repository `extra` and application `codium` are accessible only for `bammbamm`.

### Task 20: Configure Swap Space
- Create a SWAP partition of **250 MB** on an available disk partition.
- Ensure the swap space is automatically activated at system reboot via `/etc/fstab`.

### Task 21: LVM Volume Group & VFAT Filesystem
- Create a Volume Group named **`myvol`** with a Physical Extent (PE) size of **8 MiB**.
- Create a Logical Volume named **`mydatabase`** using **50 Physical Extents** (400 MiB).
- Format the logical volume with a **`vfat`** filesystem.
- Create a directory **`/database`** and mount this volume permanently on `/database`.

### Task 22: Resize Logical Volume
- Resize the existing logical volume **`home`** in volume group **`myvg`** to **152 MB (38 extents)**.
- The final filesystem size must be between **145 MB and 155 MB**.
- Ensure that the underlying filesystem is resized without data loss and remains mounted on `/home`.
