# Lesson 12: Users & Permissions in Linux

> Linux is a **multi-user operating system**. Understanding how users, groups, and permissions are structured is foundational to system security, access control, and enterprise server administration. This lesson covers user categories, Linux vs. Windows account management models, group-based permission strategies, and the essential CLI commands for managing users in production environments.

---

## Table of Contents

1. [The Three Categories of Linux Users](#1-the-three-categories-of-linux-users)
2. [Linux vs. Windows: Account Management Models](#2-linux-vs-windows-account-management-models)
3. [User & Group Permission Levels on Servers](#3-user--group-permission-levels-on-servers)
4. [System Files: Where Users & Groups Are Stored](#4-system-files-where-users--groups-are-stored)
5. [Managing Users — Commands](#5-managing-users--commands)
6. [Managing Groups — Commands](#6-managing-groups--commands)
7. [File Permissions (rwx Model)](#7-file-permissions-rwx-model)
8. [Changing Ownership & Permissions](#8-changing-ownership--permissions)
9. [Quick Reference Cheat Sheet](#9-quick-reference-cheat-sheet)

---

## 1. The Three Categories of Linux Users

```
┌──────────────────────────────────────────────────────────────┐
│                  LINUX USER CATEGORIES                       │
│                                                              │
│  ┌──────────────┐  ┌──────────────────┐  ┌───────────────┐   │
│  │  SUPERUSER   │  │  STANDARD USER   │  │ SERVICE USER  │   │
│  │   (Root)     │  │  (User Account)  │  │               │   │
│  │              │  │                  │  │ mysql         │   │
│  │  UID: 0      │  │  UID: 1000+      │  │ www-data      │   │
│  │  No limits   │  │  Own home dir    │  │ nginx         │   │
│  │  Full access │  │  Scoped access   │  │ UID: 100-999  │   │
│  └──────────────┘  └──────────────────┘  └───────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

### Superuser (Root User)
- The **unrestricted, all-powerful** administrative account on any Linux system.
- Has **UID (User ID) of 0** — reserved exclusively for root.
- Can read, write, and execute any file on the system regardless of permissions.
- Direct login as root is discouraged in production — use `sudo` instead to execute privileged commands selectively and maintain an audit trail.

### Standard User (User Account)
- Regular accounts created for individuals to log in and perform their work.
- Each user has a **dedicated home directory** (e.g., `/home/alice`) where they store personal files and configs.
- **Scoped access** — cannot modify system files or other users' data without elevated privileges.
- UIDs typically start at `1000` on modern Debian/Ubuntu systems.

### Service Users
- Accounts automatically created when server software is installed (e.g., MySQL, Apache, Nginx).
- Each service runs under **its own isolated user account** to minimize blast radius in case of compromise.
- Service users typically **cannot log in interactively** — they exist solely to run background processes.
- UIDs typically fall in the range `100–999`.

| Service | Default Service User |
|---------|---------------------|
| MySQL / MariaDB | `mysql` |
| Apache Web Server | `www-data` |
| Nginx | `nginx` or `www-data` |
| SSH Daemon | `sshd` |
| Postfix Mail | `postfix` |

> **🔐 Security Best Practice:** Never run a service as root. Each service having its own dedicated user ensures that if one service is exploited, the attacker cannot automatically access other services or system resources.

---

## 2. Linux vs. Windows: Account Management Models

These two operating systems take fundamentally different approaches to user account management.

### Windows — Centralized Management

```
┌─────────────────────────────────────────────────────┐
│              WINDOWS (Active Directory)             │
│                                                     │
│   Admin Server (Domain Controller)                  │
│          │                                          │
│          │  Validates credentials                   │
│          ▼                                          │
│   ┌──────────┐   ┌──────────┐   ┌──────────┐        │
│   │  PC #1   │   │  PC #2   │   │  PC #3   │        │
│   │  User A  │   │  User B  │   │  User C  │        │
│   └──────────┘   └──────────┘   └──────────┘        │
│                                                     │
│   All machines check credentials against            │
│   a central admin server on every login.            │
└─────────────────────────────────────────────────────┘
```

- User accounts are managed **centrally** by an administrator via Active Directory or similar systems.
- Every login attempt is **validated against the central server** — if the machine is offline or off-network, login may be denied.
- An admin can add, disable, or modify a user account and it reflects across all machines in the network immediately.

### Linux — Local Management

```
┌─────────────────────────────────────────────────────┐
│                 LINUX (Local Accounts)              │
│                                                     │
│   Laptop A              Laptop B                    │
│   ┌──────────────┐      ┌──────────────┐            │
│   │ User: alice  │      │ User: bob    │            │
│   │ User: carol  │      │ User: dave   │            │
│   │              │      │              │            │
│   │ Own /etc/    │      │ Own /etc/    │            │
│   │ passwd file  │      │ passwd file  │            │
│   └──────────────┘      └──────────────┘            │
│                                                     │
│   No shared account database. Each machine          │
│   manages its own users independently.              │
└─────────────────────────────────────────────────────┘
```

- Linux accounts are managed **locally on each machine** — there is no central authentication server by default.
- If you have 2 laptops and 4 users — 2 assigned to each laptop — those users only exist on their respective machine and have no access to the other.
- **No automatic cross-machine synchronization** — a user added on one Linux box does not exist on another.

> **Note:** Enterprise Linux environments can implement centralized authentication using tools like **LDAP**, **FreeIPA**, or **Active Directory integration (SSSD)** — but these are additional configurations, not the default.

---

## 3. User & Group Permission Levels on Servers

In a server context where multiple users share a single machine, there are two strategies to assign permissions:

```
┌───────────────────────────────────────────────────────┐
│              PERMISSION ASSIGNMENT MODELS             │
│                                                       │
│  USER LEVEL                GROUP LEVEL                │
│  ─────────────             ─────────────              │
│  Root assigns              Create a group,            │
│  permissions to            assign permissions         │
│  each user                 to the group,              │
│  individually.             add users to it.           │
│                                                       │
│  alice ──► rw             ┌─────────────┐             │
│  bob   ──► r              │   devops    │─────► rwx   │
│  carol ──► rwx            │  alice      │             │
│                           │  bob        │             │
│  Tedious for              │  carol      │             │
│  large teams.             └─────────────┘             │
│                           Efficient & scalable.       │
└───────────────────────────────────────────────────────┘
```

### User Level
- The root user grants specific permissions **directly to individual users**.
- Suitable for small teams or unique per-user access requirements.
- Becomes difficult to manage at scale.

### Group Level
- A **group** is a collection of users that shares the same set of permissions.
- Assign permissions once to the group — all members inherit them automatically.
- Adding or removing a user from the group instantly changes their access.
- Common in school labs, development teams, and corporate environments.

> **Best Practice:** Always prefer **group-based access control** over individual user permissions for scalability and consistency.

---

## 4. System Files: Where Users & Groups Are Stored

Linux stores user and group metadata in plain-text system files. These should **never be edited manually** — always use dedicated commands.

### `/etc/passwd` — User Database

```bash
cat /etc/passwd
```

Each line represents one user in the format:

```
username:password:UID:GID:comment:home_directory:shell
```

**Example:**
```
alice:x:1001:1001:Alice Dev:/home/alice:/bin/bash
mysql:x:120:130:MySQL Server:/var/lib/mysql:/usr/sbin/nologin
root:x:0:0:root:/root:/bin/bash
```

| Field | Meaning |
|-------|---------|
| `username` | Login name |
| `x` | Password placeholder (actual hash stored in `/etc/shadow`) |
| `UID` | Unique User ID |
| `GID` | Primary Group ID |
| `comment` | Description / full name |
| `home_directory` | User's home path |
| `shell` | Default login shell (`/usr/sbin/nologin` for service users) |

### `/etc/shadow` — Password Hashes (Restricted)

```bash
sudo cat /etc/shadow
```

Stores **encrypted password hashes** and password aging policies. Only readable by root.

### `/etc/group` — Group Database

```bash
cat /etc/group
```

Format:
```
groupname:password:GID:member1,member2,...
```

**Example:**
```
devops:x:1002:alice,bob,carol
sudo:x:27:alice
```

> **⚠️ Important:** Never edit `/etc/passwd`, `/etc/shadow`, or `/etc/group` directly in a text editor. Corruption of these files can make your system unusable. Use the dedicated commands below.

---

## 5. Managing Users — Commands

### Adding a User

```bash
# Add a new user (interactive — prompts for password, name, etc.)
sudo adduser <userName>

# Add a new user silently (non-interactive, no home dir setup)
sudo useradd <userName>

# Add a new user and assign to an existing group at creation
sudo useradd -G devops nicole
```

> `adduser` is the friendlier, interactive Debian/Ubuntu wrapper. `useradd` is the lower-level POSIX command available on all distributions.

### Switching Users

```bash
# Switch to another user account
su - <username>

# Switch to root (requires root password)
su -

# Execute a single command as root without switching
sudo <command>
```

### Modifying a User

```bash
# Change a user's PRIMARY group
sudo usermod -g devops tim

# Add a user to MULTIPLE additional groups (overrides existing secondary groups)
sudo usermod -G group1,group2,group3 alice

# APPEND new groups to existing secondary groups (safe — does not remove existing ones)
sudo usermod -aG group2,group3 alice
```

> **Critical Distinction:**
> - `-G` (uppercase) **replaces** the user's secondary group list entirely.
> - `-aG` (append + groups) **adds** to the existing list without removing anything.
> - Always use `-aG` unless you intentionally want to reset all group memberships.

### Deleting a User

```bash
# Remove a user (keeps home directory)
sudo deluser <userName>

# Remove a user AND their home directory
sudo deluser --remove-home <userName>

# Low-level equivalent
sudo userdel -r <userName>
```

### Logging Out / Exiting a Session

```bash
exit        # Log out of the current user session
logout      # Alternative logout command
```

---

## 6. Managing Groups — Commands

### Creating & Deleting Groups

```bash
# Create a new group
sudo groupadd <groupName>

# Delete a group
sudo delgroup <groupName>

# Example: delete the 'tim' group
sudo delgroup tim
```

### Adding & Removing Users from Groups

```bash
# Remove a user from a specific group
sudo gpasswd -d <userName> <groupName>

# Example: remove alice from the devops group
sudo gpasswd -d alice devops
```

### Viewing Group Memberships

```bash
# Show all groups a user belongs to
groups <userName>

# Show current user's groups
groups

# Show detailed group info including members
getent group <groupName>
```

---

## 7. File Permissions (rwx Model)

Every file and directory in Linux has a permission set assigned to three entities:

```
-  r w x    r w x    r w x
│  ─────    ─────    ─────
│  Owner    Group    Others
│
└── File type: - = file, d = directory, l = symlink
```

### Permission Breakdown

| Symbol | Meaning | Numeric Value |
|--------|---------|--------------|
| `r` | Read | 4 |
| `w` | Write | 2 |
| `x` | Execute | 1 |
| `-` | No permission | 0 |

### Reading a Permission String

```bash
ls -l myfile.sh
# Output:
# -rwxr-xr--  1  alice  devops  1024  May 20 10:00  myfile.sh
#  ─────────         │      │
#  Permissions    Owner  Group
```

| Segment | Value | Meaning |
|---------|-------|---------|
| Owner | `rwx` | Alice can read, write, and execute |
| Group | `r-x` | devops members can read and execute, not write |
| Others | `r--` | Everyone else can only read |

### Numeric (Octal) Notation

Permissions are commonly expressed as a 3-digit number:

| Octal | Permission | Meaning |
|-------|-----------|---------|
| `7` | `rwx` | Full access |
| `6` | `rw-` | Read + Write |
| `5` | `r-x` | Read + Execute |
| `4` | `r--` | Read only |
| `0` | `---` | No access |

```bash
chmod 755 script.sh   # Owner: rwx, Group: r-x, Others: r-x
chmod 644 config.txt  # Owner: rw-, Group: r--, Others: r--
chmod 600 secret.key  # Owner: rw-, Group: ---, Others: ---
```

---

## 8. Changing Ownership & Permissions

### `chmod` — Change File Permissions

```bash
# Using octal notation
chmod 755 script.sh

# Using symbolic notation
chmod u+x script.sh       # Add execute for the owner (user)
chmod g-w config.txt      # Remove write from group
chmod o+r report.txt      # Add read for others
chmod a+x deploy.sh       # Add execute for ALL (owner, group, others)

# Apply recursively to a directory and its contents
chmod -R 755 /var/www/html
```

### `chown` — Change File Ownership

```bash
# Change owner
sudo chown alice file.txt

# Change owner and group
sudo chown alice:devops file.txt

# Change group only
sudo chown :devops file.txt

# Apply recursively
sudo chown -R alice:devops /home/alice/project/
```

---

## 9. Quick Reference Cheat Sheet

### User Management

| Command | Action |
|---------|--------|
| `sudo adduser <name>` | Add a new user (interactive) |
| `sudo useradd -G <group> <name>` | Add user to a group at creation |
| `su - <name>` | Switch to another user |
| `su -` | Switch to root |
| `sudo usermod -g <group> <user>` | Set primary group |
| `sudo usermod -aG <group> <user>` | Append to secondary groups |
| `sudo deluser --remove-home <name>` | Delete user and home directory |
| `exit` | Log out of current session |

### Group Management

| Command | Action |
|---------|--------|
| `sudo groupadd <group>` | Create a group |
| `sudo delgroup <group>` | Delete a group |
| `sudo gpasswd -d <user> <group>` | Remove user from group |
| `groups <user>` | List a user's groups |

### Permissions

| Command | Action |
|---------|--------|
| `ls -l` | View file permissions |
| `chmod 755 <file>` | Set permissions (octal) |
| `chmod u+x <file>` | Add execute for owner |
| `chown <user>:<group> <file>` | Change owner and group |
| `chmod -R 755 <dir>` | Apply permissions recursively |

### Key System Files

| File | Contents |
|------|----------|
| `/etc/passwd` | User accounts (username, UID, GID, shell) |
| `/etc/shadow` | Encrypted password hashes |
| `/etc/group` | Group definitions and memberships |

---

*End of Lesson 12*
