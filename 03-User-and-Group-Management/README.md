# **I. User Management**

Linux is a multi-user operating system, meaning multiple users can operate on a system simultaneously. Proper user management ensures security, controlled access, and system integrity.

**The Rationale for User Management**

In a personal computing context, a single admin user is often sufficient. However, in professional Linux environments where 90% of production workloads reside, multi-user management is critical.

- **Prevention of System Corruption:** Restricting access prevents unauthorized users from deleting critical system folders, such as `/sbin` (system binaries).
- **Accountability:** Assigning unique usernames (e.g., first initial and surname) ensures that actions can be traced to specific individuals rather than a shared root account.

**Key Commands and Files**

Key files involved in user management:

- `/etc/passwd` – Stores user account details.
- `/etc/shadow` – Stores encrypted user passwords.
- `/etc/group` – Stores group information.
- `/etc/gshadow` – Stores secure group details.

The system stores user and group data in specific configuration files within the `/etc` directory.

| **Command/File**        | **Purpose**                                                                                                                                                       |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `useradd`               | A simple, non-interactive command used primarily in scripting to create users. It does not prompt for user details . It wont create /home directory for the user. |
| `adduser`               | An interactive command that prompts for user details (full name, room number) and automatically creates a home directory.                                         |
| `passwd <user>`         | Sets or changes the password for a specified user.                                                                                                                |
| `/etc/passwd`           | Contains a list of all users created on the system.                                                                                                               |
| `/etc/shadow`           | Stores encrypted user passwords using high-level hashing (e.g., SHA-256). Linux employs one-way encryption(decryption is not possible) for passwords.             |
| `userdel <user>`        | Removes a user from the system.                                                                                                                                   |
| `userdel -r <username>` | To remove a user and their home directory:                                                                                                                        |
| `su - <user>`           | Switches the current session to a different user.                                                                                                                 |
| `whoami`                | Displays the current logged-in username.                                                                                                                          |

**Password Security and Policy**

Linux employs one-way encryption(decryption is not possible) for passwords. If a user loses their password, it cannot be decrypted or restored by an administrator; it must be overwritten with a new one. Administrators can enforce security policies using the `chage` command, such as `chage -m 90 <user>`, which forces a password change every 90 days. Accounts can also be locked or unlocked to manage suspicious activity.

- **Security:** Password decryption in Linux is technically impossible due to one-way hashing; lost passwords must be reset rather than recovered. Remote access is standardly managed via SSH, often with password authentication disabled in favor of key-based security.

### **Enforcing Password Policies**

- **Password expiration**: Set password expiry days

  ```plaintext
  chage -M 90 username
  ```

- **Lock a user account**

  ```plaintext
  passwd -l username
  ```

- **Unlock a user account**

  ```plaintext
  passwd -u username
  ```

# Group Management

**Managing Permissions via Groups**

As organizations scale to hundreds or thousands of users, managing permissions individually becomes untenable.

| Task                     | Command                                   | Description                                                                                             |
| ------------------------ | ----------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| **Create a Group**       | `sudo groupadd <groupname>`               | Adds a new group to the system.                                                                         |
| **Add User to Group**    | `sudo usermod -aG <groupname> <username>` | Adds a user to a supplementary group. The `-a` flag is vital to avoid overwriting existing memberships. |
| **View Memberships**     | `groups <username>`                       | Displays all groups a specific user currently belongs to.                                               |
| **Change Primary Group** | `sudo usermod -g <groupname> <username>`  | Changes the user's main group (the one assigned to new files they create).                              |

- **Permissions:** Most of these commands require `sudo` because they modify system configuration files like `/etc/group` and `/etc/passwd`.
- **The** `-a` **Flag:** When using `usermod`, always remind users to include `-a` (append). If you run `usermod -G` without the `-a`, the user will be removed from all their other groups and only remain in the one you just specified.
- **Session Refresh:** After adding a user to a group, the user usually needs to log out and log back in (or use the `newgrp` command) for the changes to take effect in their current session.

- **Efficiency:** Administrators can modify the permissions of a single group (e.g., "devops" or "dev") to instantly update access rights for every user within that group.

## Sudo Access and Privilege Escalation

Granting administrative privileges allows users to perform system-level tasks. This is typically managed by adding users to specific administrative groups or by fine-tuning the `sudoers` configuration file.

| Task                        | Command / Configuration                   | System / Context                                                                                                                                  |
| --------------------------- | ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Grant Sudo Access**       | `sudo usermod -aG sudo <username>`        | **Debian/Ubuntu:** Adds user to the `sudo` group.                                                                                                 |
| **Grant Sudo Access**       | `sudo usermod -aG wheel <username>`       | **RHEL/CentOS/Fedora:** Adds user to the `wheel` group.                                                                                           |
| **Edit Sudo Policies**      | `sudo visudo`                             | **Global:** Opens the `/etc/sudoers` file safely for editing.                                                                                     |
| **Specific Command Access** | `<user> ALL=(ALL) NOPASSWD: /path/to/cmd` | **After running** `sudo visudo`, Add this commad in **Sudoers File \[ This command** Allows a specific command to run without a password prompt\] |

### Critical Best Practices

- **Always use** `visudo`**:** Never edit the `/etc/sudoers` file with a standard text editor like `vim` or `nano` directly. `visudo` checks for syntax errors before saving; a single typo in this file can lock every user (including root) out of administrative access.
- **The Power of** `NOPASSWD`**:** Use this sparingly. While it is useful for automation and CI/CD pipelines, it allows the specified command to be executed by that user without any identity verification, which can be a security risk if the account is compromised.
- **Group Membership Refresh:** For a user to gain sudo privileges after being added to a group, they must log out and log back in to refresh their session tokens.
