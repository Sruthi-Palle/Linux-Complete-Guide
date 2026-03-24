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
