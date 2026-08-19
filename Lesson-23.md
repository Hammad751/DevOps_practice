# Lesson 23: Creating & Managing Linux Users — DevOps Best Practices

> One of the most fundamental security practices in DevOps is **never working as root**. Instead, you create dedicated users — one per application, one per team member — and grant only the minimum permissions needed. This lesson walks through the complete workflow: creating users, assigning sudo privileges, setting up SSH key authentication, and understanding why this matters in production environments.

---

## Table of Contents

1. [The Core Security Principles](#1-the-core-security-principles)
2. [Understanding User Types & Prompts](#2-understanding-user-types--prompts)
3. [Creating a New Linux User](#3-creating-a-new-linux-user)
4. [Granting sudo Privileges](#4-granting-sudo-privileges)
5. [Switching Between Users](#5-switching-between-users)
6. [Setting Up SSH Key Authentication for the New User](#6-setting-up-ssh-key-authentication-for-the-new-user)
7. [Logging In Directly as the New User](#7-logging-in-directly-as-the-new-user)
8. [One User Per Application — The DevOps Pattern](#8-one-user-per-application--the-devops-pattern)
9. [Complete Workflow — From Root to Secure User](#9-complete-workflow--from-root-to-secure-user)
10. [Managing Users — Additional Commands](#10-managing-users--additional-commands)
11. [Quick Reference Cheat Sheet](#11-quick-reference-cheat-sheet)

---

## 1. The Core Security Principles

Before writing a single command, understand **why** this matters:

```
┌──────────────────────────────────────────────────────────────┐
│              THREE RULES OF LINUX USER SECURITY              │
│                                                              │
│  1. CREATE a separate user for every application             │
│     → If one app is compromised, others stay safe            │
│                                                              │
│  2. GRANT only the permissions needed for that application   │
│     → Principle of Least Privilege                           │
│                                                              │
│  3. NEVER work as root — even for admin tasks                │
│     → Use sudo for specific elevated commands instead        │
└──────────────────────────────────────────────────────────────┘
```

### Why Not Root?

```
Working as root:
  rm -rf /var/lib/jenkins    →  Deletes Jenkins data (intended)
  rm -rf /var/lib/          →  Typo destroys ALL application data (disaster)

  One typo = irreversible system-wide damage.
  No safety net. No "are you sure?". Instant execution.

Working as a dedicated user:
  rm -rf /var/lib/jenkins    →  Permission denied (user can't touch other apps)
  sudo rm -rf /var/lib/jenkins → Requires deliberate escalation + logs the action
```

### The Principle of Least Privilege

Every user and every application should have access to **only what it needs** — nothing more:

| User | What They Can Do | What They Cannot Do |
|------|-----------------|---------------------|
| `jenkins` | Read/write Jenkins files, run builds | Access MySQL data, modify nginx config |
| `nexus` | Read/write Nexus repository files | Access Jenkins data, modify system files |
| `hammad` | Run sudo commands when needed | Accidentally break other apps |
| `root` | Everything | — (but should never be used directly) |

---

## 2. Understanding User Types & Prompts

Linux gives you a visual indicator of which user you are working as — the **prompt symbol**:

```
root@server:~#          ← The # means you are ROOT
                          Full system access — be very careful

hammad@server:~$        ← The $ means you are a standard user
                          Limited access — much safer for daily work
```

| Symbol | User Type | Risk Level |
|--------|----------|-----------|
| `#` | Root user | High — every command runs with full system power |
| `$` | Standard user | Low — mistakes are contained to user's permissions |

> **Habit to build:** The moment you see `#`, treat every command with extra caution. Prefer working at the `$` prompt and escalating with `sudo` only when necessary.

---

## 3. Creating a New Linux User

### The `adduser` Command

```bash
# Create a new user — interactive, prompts for details
sudo adduser <username>

# Example: creating a user named 'hammad'
sudo adduser hammad
```

**The full interactive flow:**

```
Adding user 'hammad' ...
Adding new group 'hammad' (1001) ...
Adding new user 'hammad' (1001) with group 'hammad' ...
Creating home directory '/home/hammad' ...
Copying files from '/etc/skel' ...

New password: ••••••••          ← Set a strong password
Retype new password: ••••••••   ← Confirm it

passwd: password updated successfully

Changing the user information for hammad
Enter the new value, or press ENTER for the default

  Full Name []: Hammad           ← Optional: press Enter to skip
  Room Number []:                ← Press Enter
  Work Phone []:                 ← Press Enter
  Home Phone []:                 ← Press Enter
  Other []:                      ← Press Enter

Is the information correct? [Y/n] Y
```

### What `adduser` Creates Automatically

```
/home/hammad/          ← Home directory (user's personal space)
/home/hammad/.bashrc   ← Shell configuration
/home/hammad/.profile  ← Login profile
Group: hammad          ← A private group with the same name
```

### `adduser` vs `useradd`

| Command | Type | Behaviour |
|---------|------|-----------|
| `adduser` | High-level (Debian/Ubuntu) | Interactive, creates home dir, sets up defaults automatically |
| `useradd` | Low-level (all distros) | Non-interactive, minimal setup — requires manual flags |

```bash
# adduser — beginner friendly, recommended
sudo adduser hammad

# useradd — requires manual flags for same result
sudo useradd -m -s /bin/bash -G sudo hammad
sudo passwd hammad
```

> Use `adduser` for interactive user creation on Ubuntu/Debian. Use `useradd` in automated scripts where you don't want interactive prompts.

---

## 4. Granting sudo Privileges

A newly created user has **no root privileges** by default. They cannot install software, modify system files, or run administrative commands.

To grant sudo access, add the user to the **sudo group**:

```bash
# Add user to the sudo group
sudo usermod -aG sudo <username>

# Example
sudo usermod -aG sudo hammad
```

**Flag breakdown:**

| Flag | Meaning |
|------|---------|
| `-a` | **Append** — add to the group without removing existing groups |
| `-G` | **Groups** — specify which group(s) to add to |
| `sudo` | The group name that has sudo privileges on Ubuntu/Debian |

> **Critical:** Always use `-aG` together. Using `-G` alone (without `-a`) **removes** the user from all other groups and only keeps the specified one — a common destructive mistake.

### Verify the User is in the sudo Group

```bash
# Check all groups for a user
groups hammad
# Output: hammad : hammad sudo

# More detailed view
id hammad
# Output: uid=1001(hammad) gid=1001(hammad) groups=1001(hammad),27(sudo)
```

### Test sudo Works

```bash
# Switch to the new user
su - hammad

# Test sudo access
sudo whoami
# [sudo] password for hammad: ••••••••
# root    ← if this prints, sudo is working correctly
```

---

## 5. Switching Between Users

### Switch to a Specific User

```bash
# Switch to user 'hammad' (loads their full environment)
su - hammad

# The dash (-) is important:
# su hammad   → switches user but keeps current environment (partial switch)
# su - hammad → full login switch: loads user's .bashrc, .profile, home dir
```

### Return to Previous User

```bash
exit        # return to the previous user
# or
logout
```

### Switch to Root Temporarily

```bash
# Become root (requires root password)
su -

# Better approach — run a single command as root and return
sudo <command>

# Open a root shell using sudo (uses your user password, not root password)
sudo -i
```

### Visual Confirmation — Watch the Prompt Change

```
root@server:~#          ← Start as root
$ su - hammad
hammad@server:~$        ← Now as hammad ($ prompt)
$ exit
root@server:~#          ← Back to root (# prompt)
```

---

## 6. Setting Up SSH Key Authentication for the New User

By default, the new user has no SSH keys configured — they can only log in with a password. Setting up SSH key authentication is more secure and is the DevOps standard.

### Why the First SSH Attempt Fails

When you try to SSH directly as the new user without keys configured:

```bash
ssh hammad@172.26.133.181
# hammad@172.26.133.181's password:
# Permission denied, please try again.
# Permission denied, please try again.
# Permission denied, please try again.
```

This fails because:
1. The server may have `PasswordAuthentication no` set in `sshd_config`
2. Even if passwords are allowed, SSH key auth is preferred and more secure
3. The user's `~/.ssh/authorized_keys` file doesn't exist yet

### Step 1 — Switch to the New User

```bash
# From root, switch to hammad
su - hammad

# Confirm you are now hammad
whoami
# hammad
```

### Step 2 — Create the `.ssh` Directory

```bash
# Create the SSH config directory in the user's home
mkdir -p ~/.ssh

# Set correct permissions — SSH REQUIRES these exact permissions
# Wrong permissions = SSH silently ignores the key
chmod 700 ~/.ssh
```

### Step 3 — Add the Public Key to `authorized_keys`

```bash
# Create the authorized_keys file and open it
sudo vim ~/.ssh/authorized_keys

# OR without sudo (you own this directory)
vim ~/.ssh/authorized_keys
```

Paste the **public key** content (from your `id_ed25519.pub` file on your local machine):

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx your_email@example.com
```

Save and exit (`:wq`), then set correct permissions:

```bash
chmod 600 ~/.ssh/authorized_keys
```

### Step 4 — Verify Permissions Are Correct

```bash
ls -la ~/.ssh/
# drwx------  2 hammad hammad 4096  Aug 15  .      ← 700
# -rw-------  1 hammad hammad  571  Aug 15  authorized_keys ← 600
```

### SSH Permissions — Why They Must Be Exact

SSH is deliberately strict about permissions. If permissions are wrong, it **silently ignores your keys** and falls back to password authentication (or rejects entirely):

| Path | Required | If Wrong |
|------|---------|---------|
| `~/.ssh/` | `700` | SSH ignores the entire directory |
| `~/.ssh/authorized_keys` | `600` | SSH ignores all keys in the file |
| `~/.ssh/id_ed25519` (private key) | `600` | SSH refuses to use the key |

### Alternative — Use `ssh-copy-id` (Easiest Method)

From your **local machine**, automatically copy your public key to the server:

```bash
# Copy your public key to the new user on the server
ssh-copy-id -i ~/.ssh/id_ed25519.pub hammad@server-ip

# This automatically:
# - Creates ~/.ssh/ if it doesn't exist
# - Appends the key to authorized_keys
# - Sets correct permissions
```

---

## 7. Logging In Directly as the New User

After setting up SSH keys, you can log in directly as the new user — bypassing root entirely:

```bash
# From your local machine (PowerShell / terminal)
ssh hammad@localhost

# Or with explicit key
ssh -i ~/.ssh/id_ed25519 hammad@server-ip
```

**What happens:**

```
meta@DESKTOP-JSORRSM:~$ ssh hammad@localhost

Enter passphrase for key '/home/meta/.ssh/id_ed25519': ••••••••
← Enter your SSH key passphrase (not the user's password)

Welcome to Ubuntu 24.04.2 LTS (GNU/Linux 6.6.87.2-microsoft-standard-WSL2 x86_64)

  System information as of Sat Aug 15 08:24:04 PKT 2026
  Usage of /: 1.3% of 1006.85GB
  Users logged in: 1

hammad@DESKTOP-JSORRSM:~$    ← Logged in as hammad with $ prompt
```

> **Key distinction:** The SSH key passphrase belongs to **your SSH key** (set when you ran `ssh-keygen`). The user's Linux password is separate. SSH key auth uses the key — not the user's password.

### Passwordless Login (No Passphrase)

If you generated your SSH key **without** a passphrase, connection is instant:

```bash
meta@DESKTOP-JSORRSM:~$ ssh hammad@localhost
hammad@DESKTOP-JSORRSM:~$    ← Immediate login, no prompt
```

---

## 8. One User Per Application — The DevOps Pattern

This is the pattern you repeat for **every application** you deploy on a server:

```
┌──────────────────────────────────────────────────────────────┐
│              ONE USER PER APPLICATION PATTERN                │
│                                                              │
│  Server                                                      │
│  ├── User: jenkins    → owns /var/lib/jenkins/               │
│  │         runs Jenkins CI/CD service                        │
│  │         can only access Jenkins files                     │
│  │                                                           │
│  ├── User: nexus      → owns /opt/sonatype/nexus/            │
│  │         runs Nexus artifact repository                    │
│  │         cannot access Jenkins data                        │
│  │                                                           │
│  ├── User: postgres   → owns /var/lib/postgresql/            │
│  │         runs PostgreSQL database service                  │
│  │         cannot access application files                   │
│  │                                                           │
│  └── User: hammad     → your admin user (has sudo)           │
│            manages the server, deploys apps                  │
│            works with $ prompt, uses sudo when needed        │
└──────────────────────────────────────────────────────────────┘
```

### Creating Application Users

For application service users, you typically **don't** want them to be able to log in interactively — they just need to run a process:

```bash
# Create a system/service user (no home dir, no login shell)
# Used for applications like Jenkins, Nexus, nginx
sudo useradd --system --no-create-home --shell /usr/sbin/nologin jenkins

# Check the user was created
id jenkins
# uid=998(jenkins) gid=998(jenkins) groups=998(jenkins)

cat /etc/passwd | grep jenkins
# jenkins:x:998:998::/home/jenkins:/usr/sbin/nologin
# nologin = this user cannot open a shell session
```

### For Human Operators (like your admin user)

```bash
# Full interactive user with home dir and sudo
sudo adduser hammad
sudo usermod -aG sudo hammad
# Set up SSH keys (as covered in section 6)
```

### Real-World Application User Setup Examples

```bash
# Jenkins user
sudo adduser jenkins
# Jenkins typically manages its own user during apt install

# Nexus repository manager user
sudo adduser nexus
sudo mkdir -p /opt/nexus
sudo chown -R nexus:nexus /opt/nexus

# Custom app user with restricted access
sudo adduser myapp
sudo chown -R myapp:myapp /var/www/myapp
su - myapp   # switch to run the app as this user
```

---

## 9. Complete Workflow — From Root to Secure User

Here is the full sequence you follow every time you provision a new server or add a new user:

```bash
# ─────────────────────────────────────────────
# STEP 1: Connect as root (first time only)
# ─────────────────────────────────────────────
ssh root@server-ip

# ─────────────────────────────────────────────
# STEP 2: Update the system
# ─────────────────────────────────────────────
apt update && apt upgrade -y

# ─────────────────────────────────────────────
# STEP 3: Create a new non-root admin user
# ─────────────────────────────────────────────
adduser hammad
# Set password, press Enter for remaining prompts

# ─────────────────────────────────────────────
# STEP 4: Grant sudo privileges
# ─────────────────────────────────────────────
usermod -aG sudo hammad

# ─────────────────────────────────────────────
# STEP 5: Switch to the new user
# ─────────────────────────────────────────────
su - hammad
# Prompt changes from # to $

# ─────────────────────────────────────────────
# STEP 6: Set up SSH directory
# ─────────────────────────────────────────────
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# ─────────────────────────────────────────────
# STEP 7: Add your public SSH key
# ─────────────────────────────────────────────
vim ~/.ssh/authorized_keys
# Paste your public key → :wq
chmod 600 ~/.ssh/authorized_keys

# ─────────────────────────────────────────────
# STEP 8: Open a new terminal — test login
# ─────────────────────────────────────────────
# From your LOCAL machine (new terminal window):
ssh hammad@server-ip
# Should connect using SSH key

# ─────────────────────────────────────────────
# STEP 9: Verify sudo works
# ─────────────────────────────────────────────
sudo whoami
# root  ← sudo is working

# ─────────────────────────────────────────────
# STEP 10: Disable root SSH login (harden server)
# ─────────────────────────────────────────────
sudo vim /etc/ssh/sshd_config
# Set: PermitRootLogin no
# Set: PasswordAuthentication no
sudo systemctl restart sshd

# ─────────────────────────────────────────────
# DONE — Close root session. Work only as hammad.
# ─────────────────────────────────────────────
```

---

## 10. Managing Users — Additional Commands

### Viewing Users on the System

```bash
# List all users (one per line)
cat /etc/passwd | cut -d: -f1

# Show detailed info about a specific user
id hammad
# uid=1001(hammad) gid=1001(hammad) groups=1001(hammad),27(sudo)

# Show which groups a user belongs to
groups hammad

# Show all logged-in users right now
who
w          # more detailed — shows what each user is doing
```

### Modifying Users

```bash
# Change a user's password
sudo passwd hammad

# Lock a user account (disable login)
sudo usermod -L hammad

# Unlock a user account
sudo usermod -U hammad

# Change the user's shell
sudo usermod -s /bin/bash hammad

# Change the user's home directory
sudo usermod -d /new/home hammad

# Rename a user
sudo usermod -l newname hammad

# Add user to multiple groups
sudo usermod -aG sudo,docker,developers hammad
```

### Deleting Users

```bash
# Remove user — keeps home directory
sudo deluser hammad

# Remove user AND their home directory and files
sudo deluser --remove-home hammad

# Low-level alternative — also removes home dir
sudo userdel -r hammad
```

### Managing Groups

```bash
# Create a group
sudo groupadd developers

# Add user to a group
sudo usermod -aG developers hammad

# Remove user from a group
sudo gpasswd -d hammad developers

# Delete a group
sudo groupdel developers

# List all groups on the system
cat /etc/group
```

### The `/etc/passwd` File — Understanding User Records

```bash
cat /etc/passwd | grep hammad
# hammad:x:1001:1001:Hammad,,,:/home/hammad:/bin/bash
# ──────────────────────────────────────────────────
# Field 1: username       → hammad
# Field 2: password       → x (actual hash in /etc/shadow)
# Field 3: UID            → 1001
# Field 4: GID            → 1001
# Field 5: comment/name   → Hammad,,,
# Field 6: home directory → /home/hammad
# Field 7: shell          → /bin/bash
```

---

## 11. Quick Reference Cheat Sheet

### Create & Configure a User

```bash
sudo adduser <username>               # Create user (interactive)
sudo usermod -aG sudo <username>      # Grant sudo
su - <username>                       # Switch to user
sudo whoami                           # Verify sudo → prints "root"
groups <username>                     # Check group membership
id <username>                         # Show UID, GID, all groups
```

### SSH Key Setup (as the new user)

```bash
mkdir -p ~/.ssh                       # Create .ssh directory
chmod 700 ~/.ssh                      # Set directory permissions
vim ~/.ssh/authorized_keys            # Paste public key here
chmod 600 ~/.ssh/authorized_keys      # Set file permissions
```

### From Local Machine

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@host   # Copy key automatically
ssh user@host                                     # Connect with key
ssh user@localhost                                # Connect to local WSL2
```

### Prompt Reference

```
root@server:~#    → Root user  — # symbol — maximum caution
user@server:~$    → Standard user — $ symbol — safe daily work
```

### Manage Users

```bash
sudo passwd <user>            # Change password
sudo usermod -L <user>        # Lock account
sudo usermod -U <user>        # Unlock account
sudo deluser --remove-home <user>  # Delete user + home dir
cat /etc/passwd               # View all users
who                           # View logged-in users
```

### Security Rules

```
✅ Create one user per application
✅ Use sudo instead of logging in as root
✅ Set up SSH key authentication for every user
✅ Set chmod 700 on ~/.ssh/
✅ Set chmod 600 on ~/.ssh/authorized_keys
✅ Disable root SSH login after admin user is set up
✅ Grant only the minimum permissions needed (least privilege)
✅ Test SSH key login in a new terminal BEFORE disabling password auth
```

---

*End of Lesson 23*