# Linux File System Structure and Directory Hierarchy

## The Shell Prompt and User Identity

The command-line interface (CLI) prompt provides immediate context regarding the user's status and location. For example, in root@ubuntu:/dev#:

- User (root): The first part indicates the logged-in user. The root user is the administrative user with unrestricted privileges. Regular users (e.g., ubuntu or abi) have restricted access.
- Hostname (ubuntu): Identifies the specific host or machine.
- Separator (:): Delimits the identity from the location.
- Present Working Directory (/dev): Indicates the current path within the file system.
- Prompt Symbol: A hashtag # signifies root/administrative access, while a dollar sign $ typically signifies a regular user.

# Understanding the Folder Structure

### Explanation of System Directories

### **Symbolic Links (Less Significant)**

- /usr User Directory The modern parent for binaries and libraries. In many systems, /bin, /sbin, and /lib are shortcuts to /usr/bin, /usr/sbin, and /usr/lib.

| Directory            | Description                                                                                                                                 |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `/sbin -> /usr/sbin` | System binaries for administrative commands (linked to `/usr/sbin`).                                                                        |
| `/bin -> /usr/bin`   | Essential user binaries. Non-administrative, general-purpose commands (e.g., ls, date, diff) accessible to all users (linked to `/usr/bin`) |
| `/lib -> /usr/lib`   | Shared libraries and kernel modules used by the Linux kernel to perform system calls and interact with hardware (linked to `/usr/lib`).     |

### **Important System Directories**

| Directory | Description                                                                                                  |
| --------- | ------------------------------------------------------------------------------------------------------------ |
| `/boot`   | Stores files needed for booting the system (not relevant in containers).                                     |
| `/usr`    | Contains most user-installed applications and libraries.                                                     |
| `/var`    | Stores logs, caches, and temporary files that change frequently.                                             |
| `/etc`    | Stores system configuration files. Key files include passwd (user management) and hosts (local DNS caching). |

### **User & Application-Specific Directories**

| Directory | Description                                                                                                                                                                                                                    |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `/home`   | Default location for user home directories.                                                                                                                                                                                    |
| `/opt`    | Used for installing optional third-party software. or custom dependencies (e.g., a specific version of Java). Using /opt provides a common location accessible to multiple users without cluttering personal home directories. |
| `/srv`    | Holds data for services like web servers (e.g., Apache or HTTPD) (rarely used in containers).                                                                                                                                  |
| `/root`   | Home directory for the root user.                                                                                                                                                                                              |

### **Temporary & Volatile Directories**

| Directory | Description                                             |
| --------- | ------------------------------------------------------- |
| `/tmp`    | Temporary files (cleared on reboot).                    |
| `/run`    | Holds runtime data for processes.                       |
| `/proc`   | Virtual filesystem for process and system information.  |
| `/sys`    | Virtual filesystem for hardware and kernel information. |
| `/dev`    | Contains device files (e.g., `/dev/null`, `/dev/sda`).  |

### **Mount Points**

| Directory | Description                                                                                     |
| --------- | ----------------------------------------------------------------------------------------------- |
| `/mnt`    | A temporary location used by administrators to mount new disks or volumes into the file system. |
| `/media`  | Mount point for removable media (USB, CDs).                                                     |
| `/data`   | Likely your **mounted volume** from Windows (`C:/ubuntu-data`).                                 |

### The PATH Environment Variable

The PATH variable is a critical system configuration that allows users to run commands (like ls or mkdir) from any location without typing the full directory path (e.g., /usr/bin/ls).

**How PATH Functions:**

1.  When a command is entered, the kernel searches through a list of directories defined in the PATH variable.
2.  The directories in the PATH are separated by colons (:).
3.  Standard paths typically include /usr/local/sbin, /usr/local/bin, /usr/sbin, /usr/bin, /sbin, and /bin.
4.  If a binary for the command is found in any of these directories, the command executes. If it is not found, the system returns a "command not found" error.

**Verification Commands:**

- which \[command\]: Identifies the exact file system path of a specific binary.
- echo $PATH: Displays the current list of directories the system is configured to search for executables.

# Detailed Explanation of /var

Unlike `/usr` or `/bin`, which contain static executables, `/var` is the "workspace" for the OS and applications to store logs, caches, and temporary state data.

---

## 📂 Key Subdirectories in `/var`

Understanding the subfolders is the best way to grasp why `/var` is critical for system administration and DevOps:

- `/var/log`: This is arguably the most important directory. It contains system and application log files (e.g., `syslog`, `auth.log`, `dmesg`). When troubleshooting a server crash or a failed login, this is your first stop.
- `/var/lib`: Contains "state" information. This is persistent data that programs modify as they run. For example, Docker images and containers are typically stored in `/var/lib/docker`, and database files for MySQL or PostgreSQL live here.
- `/var/cache`: Used for cached data from applications. This data is locally generated as a result of time-consuming I/O or calculation. If you delete it, the application should still function, though it might be slower while it regenerates the cache.
- `/var/spool`: Holds data waiting for processing, such as print queues or outgoing mail (e.g., Postfix or Sendmail).
- `/var/tmp`: Similar to `/tmp`, but files here are intended to survive a system reboot.
- `/var/run`: Contains information about the system since it was last booted (e.g., Process IDs (PIDs) of running daemons). On modern systems, this is often a symbolic link to `/run`.

---

## SysAdmin Best Practices

Because `/var` is the "junk drawer" that never stops growing, it requires specific management:

### 1\. Disk Space Management

Since logs (`/var/log`) and databases (`/var/lib`) live here, `/var` is the directory most likely to fill up your disk.

- **Log Rotation:** Linux uses a tool called `logrotate` to compress and eventually delete old logs so they don't consume the entire drive.
- **Partitioning:** On production servers, it is a standard best practice to mount `/var` on its own separate partition. This ensures that if an application starts logging excessively and fills up the space, it won't crash the root (`/`) filesystem or prevent the OS from booting.

### 2\. Permissions & Security

Standard users generally cannot write to `/var`. Most subdirectories are owned by `root` or specific service accounts (like `www-data` for web servers). This prevents a regular user from accidentally deleting system logs or database files.

### 3\. Cleanup Commands

If your `/var` directory is getting too large, you can check usage with:

```bash
du -sh /var/* | sort -h
```

Common cleanup tasks include running `apt-get clean` (for Debian/Ubuntu) or `dnf clean all` (for Fedora/RHEL) to clear out the package manager's cache in `/var/cache/`.
