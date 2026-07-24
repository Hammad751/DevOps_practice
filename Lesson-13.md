# Lesson 13: Linux File Permissions

> Linux permissions are the gatekeeper of system security. Every file and directory has a defined **owner**, a **group**, and a set of **privileges** that control exactly who can read, write, or execute it. Mastering this model is non-negotiable for any Linux administrator or DevOps engineer.

---

## Table of Contents

1. [Viewing File Permissions](#1-viewing-file-permissions)
2. [Anatomy of a Permission String](#2-anatomy-of-a-permission-string)
3. [File Types](#3-file-types)
4. [The Two Core Concepts: Ownership & Permissions](#4-the-two-core-concepts-ownership--permissions)
5. [Changing Ownership — `chown` & `chgrp`](#5-changing-ownership--chown--chgrp)
6. [Changing Permissions — `chmod` (Symbolic Mode)](#6-changing-permissions--chmod-symbolic-mode)
7. [Changing Permissions — `chmod` (Numeric / Octal Mode)](#7-changing-permissions--chmod-numeric--octal-mode)
8. [Common Permission Patterns](#8-common-permission-patterns)
9. [Viewing Hidden Files](#9-viewing-hidden-files)
10. [Quick Reference Cheat Sheet](#10-quick-reference-cheat-sheet)

---

## 1. Viewing File Permissions

```bash
# List files with detailed permission information
ls -l

# List ALL files including hidden files (those starting with a dot)
ls -la
```

**Sample output:**

```
drwxrwxr-x  2  nana  nana  4096  Mai  7  11:00  api/
-rw-rw-r--  1  nana  nana   320  Mai  3  14:57  config.yaml
-rwxr-xr--  1  nana  nana   512  Mai  5  09:30  deploy.sh
```

---

## 2. Anatomy of a Permission String

Every line from `ls -l` follows this structure:

```
┌─ File Type
│  ┌─── Owner Permissions
│  │    ┌─── Group Permissions
│  │    │    ┌─── Others Permissions
│  │    │    │         ┌─ Hard Links
│  │    │    │         │  ┌─ Owner Name
│  │    │    │         │  │     ┌─ Group Name
│  │    │    │         │  │     │     ┌─ File Size (bytes)
│  │    │    │         │  │     │     │     ┌─ Last Modified
│  │    │    │         │  │     │     │     │               ┌─ Name
│  │    │    │         │  │     │     │     │               │
d  rwx  rwx  r-x       2  nana  nana  4096  Mai  7  11:00   api/
-  rw-  rw-  r--       1  nana  nana   320  Mai  3  14:57   config.yaml
```

### Breaking Down Each Permission Block

```
d  rwx  rwx  r-x
│  ─┬─  ─┬─  ─┬─
│   │    │    └── OTHERS  → people who are neither the owner nor in the group
│   │    └─────── GROUP   → users who belong to the file's assigned group
│   └──────────── OWNER   → the user who owns the file
└──────────────── TYPE    → d = directory, - = regular file, l = symlink
```

### What Each Character Means

| Character | Meaning | Applies To |
|-----------|---------|-----------|
| `r` | **Read** — view file contents or list directory | Owner / Group / Others |
| `w` | **Write** — modify file contents or add/remove files in directory | Owner / Group / Others |
| `x` | **Execute** — run as a program, or enter a directory (`cd`) | Owner / Group / Others |
| `-` | **No permission** for this action | Owner / Group / Others |

### Real Example Decoded

```
-rw-rw-r--   1   nana   nana   320   Mai 3   14:57   config.yaml
```

| Entity | Permissions | Can Do |
|--------|------------|--------|
| Owner (`nana`) | `rw-` | Read and write — cannot execute |
| Group (`nana`) | `rw-` | Read and write — cannot execute |
| Others | `r--` | Read only — cannot write or execute |

---

## 3. File Types

The very first character in the permission string identifies the **file type**:

| Character | Type | Description |
|-----------|------|-------------|
| `-` | Regular file | A standard file (text, binary, script, etc.) |
| `d` | Directory | A folder — in Linux, everything is treated as a file |
| `l` | Symbolic link | A shortcut/pointer to another file or directory |
| `c` | Character device | Hardware interface (e.g., keyboard, terminal) |
| `b` | Block device | Storage device (e.g., hard drive, USB) |

> **Linux Philosophy:** In Linux, *everything is a file* — directories, devices, sockets, and pipes are all represented as files in the filesystem.

---

## 4. The Two Core Concepts: Ownership & Permissions

Linux file security is built on two pillars:

```
┌──────────────────────────────────────────────┐
│            FILE SECURITY MODEL               │
│                                              │
│  OWNERSHIP              PERMISSIONS          │
│  ─────────              ───────────          │
│  Who owns it?           What can they do?    │
│                                              │
│  • User (owner)         • Read    (r)        │
│  • Group                • Write   (w)        │
│                         • Execute (x)        │
│                                              │
│  drwxrwxr-x  2  nana  nana  4096  api/       │
│                 ────  ────                   │
│                 User  Group                  │
└──────────────────────────────────────────────┘
```

In `drwxrwxr-x 2 nana nana 4096 Mai 7 11:00 api/`:
- **First `nana`** → the **user (owner)** of the file
- **Second `nana`** → the **group** assigned to the file

---

## 5. Changing Ownership — `chown` & `chgrp`

### `chown` — Change Owner and/or Group

```bash
# Change BOTH the owner and group in one command
sudo chown <userName>:<groupName> <fileName>

# Example: assign alice as owner and devops as group
sudo chown alice:devops config.yaml

# Change ONLY the owner (group stays the same)
sudo chown alice myfile.txt

# Change ONLY the group using chown
sudo chown :devops myfile.txt

# Apply recursively to a directory and all its contents
sudo chown -R alice:devops /var/www/project/
```

### `chgrp` — Change Group Only

```bash
# Change the group of a file
sudo chgrp <groupName> <fileName>

# Example
sudo chgrp devops config.yaml

# Apply recursively
sudo chgrp -R devops /var/www/project/
```

> **When to use which:**
> - Use `chown user:group file` when you need to change both at once.
> - Use `chgrp` when you only need to change the group and want to be explicit about it.

---

## 6. Changing Permissions — `chmod` (Symbolic Mode)

`chmod` uses **symbols** to describe who gets what permission and whether to add (`+`), remove (`-`), or set (`=`) it.

### Syntax

```
chmod  [who][operator][permissions]  <file>

Who:        u = user/owner    g = group    o = others    a = all (u+g+o)
Operator:   + = add           - = remove   = = set exactly
Permission: r = read          w = write    x = execute
```

### Adding & Removing Permissions

```bash
# Add execute permission for the owner
sudo chmod u+x script.sh

# Remove write permission from the group
sudo chmod g-w README.md

# Add write permission back to the group
sudo chmod g+w README.md

# Add execute permission for others
sudo chmod o+x script.sh

# Remove execute from EVERYONE (owner, group, and others)
sudo chmod a-x script.sh

# Add read for everyone
sudo chmod a+r report.txt
```

### Setting Permissions Exactly with `=`

The `=` operator **sets permissions to exactly what you specify** — anything not listed is removed.

```bash
# Give the group read, write, and execute (replaces whatever was there)
sudo chmod g=rwx config.yaml

# Give the group read and write only (execute is explicitly denied by omission)
sudo chmod g=rw config.yaml

# Give others read only
sudo chmod o=r config.yaml

# Set owner to full, group to read-execute, others to nothing
sudo chmod u=rwx,g=rx,o= script.sh
```

> **`=` vs `+`:**
> - `chmod g+x` adds execute while keeping existing permissions untouched.
> - `chmod g=x` sets group permission to execute-only, removing any existing `r` or `w`.

### Rename a File (bonus: `mv`)

```bash
# Rename a file using mv
mv test.txt script.sh
```

---

## 7. Changing Permissions — `chmod` (Numeric / Octal Mode)

Every permission combination maps to a number from 0–7. This is the fastest and most common way to set permissions in scripts and production environments.

### The Permission Number Table

| Number | Binary | Permission | Symbol |
|--------|--------|-----------|--------|
| `0` | 000 | No permission | `---` |
| `1` | 001 | Execute only | `--x` |
| `2` | 010 | Write only | `-w-` |
| `3` | 011 | Write + Execute | `-wx` |
| `4` | 100 | Read only | `r--` |
| `5` | 101 | Read + Execute | `r-x` |
| `6` | 110 | Read + Write | `rw-` |
| `7` | 111 | Read + Write + Execute | `rwx` |

### How to Build an Octal Permission Code

A 3-digit number covers **owner**, **group**, and **others** in that order:

```
chmod  7  5  0  README.md
       │  │  └── Others:  0 = ---  (no permission)
       │  └───── Group:   5 = r-x  (read + execute)
       └──────── Owner:   7 = rwx  (full access)
```

### Common Examples

```bash
# Give full permission to everyone
sudo chmod 777 README.md
# Owner: rwx | Group: rwx | Others: rwx

# Owner: full, Group: read+execute, Others: none
sudo chmod 750 README.md
# Owner: rwx | Group: r-x | Others: ---

# Owner: read+write, Group: read only, Others: none (private config)
sudo chmod 640 config.yaml
# Owner: rw- | Group: r-- | Others: ---

# Executable script — owner full, group read+execute, others read+execute
sudo chmod 755 deploy.sh
# Owner: rwx | Group: r-x | Others: r-x

# Owner read+write only — sensitive file (SSH keys, secrets)
sudo chmod 600 secret.key
# Owner: rw- | Group: --- | Others: ---
```

### Quick Mental Math for Octal

```
r = 4
w = 2
x = 1

Add them together:
r + w + x = 4 + 2 + 1 = 7   → rwx
r + w     = 4 + 2     = 6   → rw-
r + x     = 4 + 1     = 5   → r-x
r         = 4         = 4   → r--
none      = 0         = 0   → ---
```

---

## 8. Common Permission Patterns

| Octal | Symbol | Use Case |
|-------|--------|---------|
| `777` | `rwxrwxrwx` | Temporary testing only — never in production |
| `755` | `rwxr-xr-x` | Web server files, public scripts |
| `750` | `rwxr-x---` | Team scripts — others have no access |
| `644` | `rw-r--r--` | Standard documents and config files |
| `640` | `rw-r-----` | Sensitive configs readable by group only |
| `600` | `rw-------` | Private keys, secrets, credentials |
| `400` | `r--------` | Read-only archive — owner can't even write |

> **🔐 Security Rule of Thumb:** Always assign the **minimum permissions necessary**. Start restrictive and open up access only when required.

---

## 9. Viewing Hidden Files

In Linux, any file or directory whose name starts with a `.` (dot) is considered **hidden** — it won't appear in a standard `ls` listing.

```bash
# Show ALL files including hidden ones
ls -la

# Show hidden files only (using glob pattern)
ls -d .*
```

**Common hidden files and directories:**

| File / Directory | Purpose |
|-----------------|---------|
| `.bashrc` | User shell configuration |
| `.ssh/` | SSH keys and known hosts |
| `.gitignore` | Git ignore rules |
| `.env` | Environment variables (often contains secrets) |
| `.config/` | Application configuration directory |

> **⚠️ Note:** Hidden files are not secure — they are simply not displayed by default. Anyone with read access to the directory can still view them with `ls -la`.

---

## 10. Quick Reference Cheat Sheet

### Viewing Permissions

| Command | Action |
|---------|--------|
| `ls -l` | List files with permissions |
| `ls -la` | List all files including hidden |
| `stat <file>` | Detailed file metadata including permissions |

### Changing Ownership

| Command | Action |
|---------|--------|
| `sudo chown user:group <file>` | Change owner and group |
| `sudo chown user <file>` | Change owner only |
| `sudo chgrp group <file>` | Change group only |
| `sudo chown -R user:group <dir>` | Recursive ownership change |

### `chmod` Symbolic Mode

| Command | Action |
|---------|--------|
| `chmod u+x <file>` | Add execute for owner |
| `chmod g-w <file>` | Remove write from group |
| `chmod o+r <file>` | Add read for others |
| `chmod a-x <file>` | Remove execute from all |
| `chmod g=rwx <file>` | Set group to exact permissions |

### `chmod` Octal Mode

| Command | Result |
|---------|--------|
| `chmod 777 <file>` | Full access for everyone |
| `chmod 755 <file>` | Owner full, group/others read+execute |
| `chmod 750 <file>` | Owner full, group read+execute, others none |
| `chmod 644 <file>` | Owner read+write, group/others read only |
| `chmod 600 <file>` | Owner read+write only — private |

### Permission Number Reference

| Number | Permission |
|--------|-----------|
| `7` | `rwx` — full |
| `6` | `rw-` — read + write |
| `5` | `r-x` — read + execute |
| `4` | `r--` — read only |
| `0` | `---` — no access |

---

*End of Lesson 13*
