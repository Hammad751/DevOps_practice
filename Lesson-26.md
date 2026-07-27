# WSL2 Lab — Windows Subsystem for Linux 2

> A practical reference for developers using WSL2, Docker Desktop, Hyper-V, SSH, and Windows. This guide covers architecture, setup, migration, troubleshooting, and best practices for working with WSL2 in real development environments.

---

## Table of Contents

1. [What is WSL2?](#1-what-is-wsl2)
2. [Core Components](#2-core-components)
3. [Architecture Flow](#3-architecture-flow)
4. [Typical Use Cases](#4-typical-use-cases)
5. [Installing WSL2](#5-installing-wsl2)
6. [Essential WSL2 Commands](#6-essential-wsl2-commands)
7. [Migrating a Distribution](#7-migrating-a-distribution)
8. [WSL Networking](#8-wsl-networking)
9. [Docker Desktop with WSL2](#9-docker-desktop-with-wsl2)
10. [SSH on WSL2](#10-ssh-on-wsl2)
11. [Performance Tips](#11-performance-tips)
12. [Configuring WSL2 — `.wslconfig`](#12-configuring-wsl2--wslconfig)
13. [Common Errors & Fixes](#13-common-errors--fixes)
14. [Recovery Without Rebooting](#14-recovery-without-rebooting)
15. [Troubleshooting Matrix](#15-troubleshooting-matrix)
16. [Best Practices](#16-best-practices)
17. [Quick Reference Cheat Sheet](#17-quick-reference-cheat-sheet)

---

## 1. What is WSL2?

**WSL2 (Windows Subsystem for Linux 2)** runs a real Linux kernel inside a lightweight **Hyper-V virtual machine**. It provides near-native Linux compatibility while integrating tightly with Windows — letting developers run Linux tools, scripts, Docker, and servers directly from Windows without dual-booting or a full VM.

```
┌──────────────────────────────────────────────────────┐
│                   WINDOWS HOST                       │
│                                                      │
│  ┌───────────────────────────────────────────────┐   │
│  │           Hyper-V Utility VM                  │   │
│  │                                               │   │
│  │   ┌─────────────────────────────────────┐     │   │
│  │   │         Real Linux Kernel           │     │   │
│  │   │                                     │     │   │
│  │   │  ┌──────────┐   ┌───────────────┐   │     │   │
│  │   │  │  Ubuntu  │   │  Your App /   │   │     │   │
│  │   │  │  Distro  │   │  Docker/Node  │   │     │   │
│  │   │  └──────────┘   └───────────────┘   │     │   │
│  │   └─────────────────────────────────────┘     │   │
│  └───────────────────────────────────────────────┘   │
│                                                      │
│   Windows Apps    File Explorer    VS Code           │
└──────────────────────────────────────────────────────┘
```

### WSL1 vs WSL2

| Feature | WSL1 | WSL2 |
|---------|------|------|
| Linux kernel | Translation layer (no real kernel) | Real Linux kernel |
| Filesystem performance | Faster on Windows files | Much faster on Linux files |
| System call compatibility | Partial | Full (runs any Linux binary) |
| Docker support | Limited | Full native support |
| Memory usage | Lower | Higher (VM overhead) |
| Network | Shared with Windows | Virtual network via Hyper-V |

---

## 2. Core Components

Understanding WSL2's internals helps significantly when troubleshooting.

| Component | Role |
|-----------|------|
| **WslService** | Windows service that manages the WSL lifecycle — starting, stopping, and registering distributions |
| **vmcompute** | Hyper-V Host Compute Service — creates and manages the utility VM that WSL2 runs inside |
| **vmmem / vmmemWSL** | The running VM process visible in Task Manager — consumes RAM and CPU on behalf of WSL2 |
| **ext4.vhdx** | Virtual disk file storing the entire Linux filesystem. Located at `%LOCALAPPDATA%\Packages\...\LocalState\ext4.vhdx` |
| **Hyper-V** | Windows virtualization platform — the foundation WSL2 is built on |
| **Docker Desktop** | Uses WSL2 as its backend engine when WSL integration is enabled |
| **wsl.exe** | The CLI entry point — the command you type to interact with WSL |

### Dependency Chain

```
wsl.exe
  └── WslService       (manages WSL lifecycle)
        └── vmcompute  (Hyper-V host compute)
              └── vmmem (the running VM)
                    └── Linux Kernel
                          └── Ubuntu / Distro
                                └── Your Applications
```

---

## 3. Architecture Flow

```
You type a command in PowerShell or Terminal
        │
        ▼
    wsl.exe
        │
        ▼
    WslService      ← Windows service managing WSL
        │
        ▼
    vmcompute       ← Hyper-V backend (creates the VM)
        │
        ▼
    vmmem           ← The live VM process (uses RAM/CPU)
        │
        ▼
  Linux Kernel      ← Real kernel running inside the VM
        │
        ▼
  Ubuntu / Distro   ← Your Linux distribution
        │
        ▼
  Your Applications ← Node, Python, Docker, etc.
```

> **Key insight:** If WSL freezes, the problem is almost always in the `vmcompute` ↔ `vmmem` layer — not in Linux itself. Restarting `vmcompute` fixes most stuck-WSL scenarios without a full Windows reboot.

---

## 4. Typical Use Cases

WSL2 is widely used for:

| Category | Examples |
|----------|---------|
| **Containerization** | Docker, Kubernetes, Podman |
| **Web Development** | Node.js, Python, Go, Rust, PHP |
| **DevOps & Cloud** | Ansible, Terraform, AWS CLI, kubectl |
| **Databases** | MySQL, PostgreSQL, Redis, MongoDB |
| **Blockchain** | Ethereum tools, Hardhat, Foundry |
| **Linux Utilities** | grep, awk, sed, curl, vim, bash scripts |
| **Version Control** | Git with native Linux performance |

---

## 5. Installing WSL2

### Prerequisites

- Windows 10 version 1903+ (Build 18362+) or Windows 11
- Virtualization enabled in BIOS/UEFI
- Hyper-V feature available

### Installation

Open **PowerShell as Administrator** and run:

```powershell
# Install WSL2 with the default Ubuntu distribution
wsl --install

# Install a specific distribution
wsl --install -d Ubuntu-22.04

# List available distributions
wsl --list --online
```

Restart your machine after installation.

### Verify Installation

```powershell
# Check WSL version
wsl --version

# List installed distributions and their WSL version
wsl --list --verbose
# or
wsl -l -v
```

**Expected output:**
```
  NAME            STATE           VERSION
* Ubuntu-22.04    Running         2
```

### Set WSL2 as Default

```powershell
# Make WSL2 the default for all new distributions
wsl --set-default-version 2

# Convert an existing WSL1 distro to WSL2
wsl --set-version Ubuntu 2
```

---

## 6. Essential WSL2 Commands

All commands below run in **PowerShell** or **Command Prompt** on Windows.

### Starting & Stopping

```powershell
# Start the default Linux distribution
wsl

# Start a specific distribution
wsl -d Ubuntu-22.04

# Stop ALL running distributions (full shutdown)
wsl --shutdown

# Stop a specific distribution
wsl -t Ubuntu-22.04
```

### Managing Distributions

```powershell
# List installed distributions
wsl --list --verbose

# Set a default distribution
wsl --set-default Ubuntu-22.04

# Unregister (delete) a distribution — DESTRUCTIVE, data is lost
wsl --unregister Ubuntu-22.04
```

### Exporting & Importing

```powershell
# Export a distribution to a .tar file (backup)
wsl --export Ubuntu-22.04 C:\Backups\ubuntu-backup.tar

# Import a distribution from a .tar file
wsl --import Ubuntu-22.04 D:\WSL\Ubuntu C:\Backups\ubuntu-backup.tar

# Import with WSL version specified
wsl --import Ubuntu-22.04 D:\WSL\Ubuntu C:\Backups\ubuntu-backup.tar --version 2
```

### Service Management (PowerShell as Admin)

```powershell
# Stop WSL manager service
net stop WslService

# Start WSL manager service
net start WslService

# Stop Hyper-V backend
net stop vmcompute

# Start Hyper-V backend
net start vmcompute
```

---

## 7. Migrating a Distribution

Migrating moves your WSL2 distro to another drive (useful when your C: drive is running low on space).

### Step-by-Step Migration

```powershell
# Step 1 — Shut down WSL cleanly
wsl --shutdown

# Step 2 — Export the distribution to a tar file
wsl --export Ubuntu-22.04 D:\Backups\ubuntu-export.tar

# Step 3 — Unregister the existing installation
wsl --unregister Ubuntu-22.04

# Step 4 — Import into the new location (e.g., another SSD)
wsl --import Ubuntu-22.04 D:\WSL\Ubuntu D:\Backups\ubuntu-export.tar --version 2

# Step 5 — Set the default user (otherwise it defaults to root after import)
ubuntu2204 config --default-user your_username

# Step 6 — Verify it starts correctly
wsl -d Ubuntu-22.04

# Step 7 — Reconfigure Docker Desktop integration if needed
# Open Docker Desktop → Settings → Resources → WSL Integration → enable your distro
```

> **Important:** After importing, WSL sets the default user to root. Always run the `config --default-user` step to restore your normal user.

---

## 8. WSL Networking

WSL2 uses a **virtual network** provided by Hyper-V — it has its own IP address separate from Windows.

### Key Networking Facts

```
Windows Host IP:   192.168.x.x  (your regular network)
WSL2 IP:           172.x.x.x    (virtual network, changes on restart)
localhost:         Works from WSL → Windows and Windows → WSL (most cases)
```

### Accessing Windows from WSL

```bash
# Get the Windows host IP from inside WSL
cat /etc/resolv.conf | grep nameserver

# Or
ip route show | grep -i default | awk '{ print $3}'
```

### Accessing WSL from Windows

```
# In Windows Explorer — browse Linux files
\\wsl.localhost\Ubuntu-22.04\home\username

# Or use the path
\\wsl$\Ubuntu-22.04
```

### Fixing `\\wsl.localhost` Inaccessible

```powershell
# Restart WSL
wsl --shutdown
wsl

# If still broken, restart the Hyper-V backend
net stop vmcompute
net start vmcompute
```

### Port Forwarding (Accessing WSL2 services from other machines)

WSL2's virtual network is not directly accessible from other devices on your LAN. To expose a service:

```powershell
# Forward port 3000 from Windows to WSL2
netsh interface portproxy add v4tov4 listenport=3000 listenaddress=0.0.0.0 connectport=3000 connectaddress=$(wsl hostname -I)

# View existing port proxies
netsh interface portproxy show all

# Remove a port proxy
netsh interface portproxy delete v4tov4 listenport=3000 listenaddress=0.0.0.0
```

---

## 9. Docker Desktop with WSL2

Docker Desktop uses WSL2 as its backend engine — containers run inside the WSL2 VM.

### Enabling WSL2 Backend

1. Open **Docker Desktop**
2. Go to **Settings → General**
3. Ensure **"Use the WSL 2 based engine"** is checked
4. Go to **Settings → Resources → WSL Integration**
5. Enable integration for your distro (e.g., Ubuntu-22.04)
6. Click **Apply & Restart**

### Verifying Docker Works in WSL

```bash
# Inside WSL terminal
docker --version
docker run hello-world
docker ps
```

### Docker Fails After Migration

If Docker stops working after migrating your distro:

```
1. Start WSL normally first:    wsl -d Ubuntu-22.04
2. Verify Linux is running:     wsl -l -v  (should show Running)
3. Open Docker Desktop
4. Go to Settings → Resources → WSL Integration
5. Re-enable the migrated distro
6. Apply & Restart Docker Desktop
```

> Docker Desktop needs to re-detect the distro after migration — it does not automatically follow the new location.

---

## 10. SSH on WSL2

WSL2 includes a full SSH client and can generate and use SSH keys exactly like any Linux system.

### Generating SSH Keys

```bash
# Generate a modern Ed25519 key (recommended)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Or RSA 4096 (widely compatible)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

This creates:
- `~/.ssh/id_ed25519` → **private key** (never share or expose)
- `~/.ssh/id_ed25519.pub` → **public key** (place this on servers)

### Copying Your Public Key to a Server

```bash
# View your public key
cat ~/.ssh/id_ed25519.pub

# Copy it to a remote server
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server-ip

# Or manually add to authorized_keys on the server
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
```

### Setting Correct Permissions

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

### Connecting to a Server

```bash
# Basic connection
ssh user@server-ip

# Specify a key explicitly
ssh -i ~/.ssh/id_ed25519 user@server-ip

# Connect on a non-standard port
ssh -p 2222 user@server-ip
```

### Using SSH Config for Convenience

Create `~/.ssh/config` to save connection shortcuts:

```
Host my-server
    HostName 192.168.1.100
    User alice
    IdentityFile ~/.ssh/id_ed25519
    Port 22
```

Now connect with just:

```bash
ssh my-server
```

### Adding SSH Keys to GitHub (from WSL)

```bash
# Copy your public key
cat ~/.ssh/id_ed25519.pub
```

1. Go to **GitHub → Settings → SSH and GPG keys**
2. Click **New SSH key**
3. Paste the public key content
4. Test: `ssh -T git@github.com`

---

## 11. Performance Tips

WSL2 runs a full VM, so some tuning helps keep it fast and resource-efficient.

### File System Performance

```
Fast:   Working on files inside WSL  (/home/user/project)
Slow:   Working on Windows files from WSL  (/mnt/c/Users/...)
```

> Always store your projects **inside the WSL filesystem** (`~/ or /home/user/`), not on `/mnt/c/`. Cross-filesystem access is significantly slower.

### Disk Space

```bash
# Check disk usage inside WSL
df -h

# Find large directories
du -sh /* 2>/dev/null | sort -rh | head -20
```

### Compacting the Virtual Disk

Over time, `ext4.vhdx` grows even after deleting files. Compact it periodically:

```powershell
# Shut down WSL first
wsl --shutdown

# Open diskpart
diskpart

# In diskpart prompt:
select vdisk file="C:\Users\YourName\AppData\Local\Packages\...\LocalState\ext4.vhdx"
attach vdisk readonly
compact vdisk
detach vdisk
exit
```

### Storing Distros on Another Drive

When importing, specify a different drive location:

```powershell
wsl --import Ubuntu-22.04 D:\WSL\Ubuntu C:\backup\ubuntu.tar
```

---

## 12. Configuring WSL2 — `.wslconfig`

Create a `.wslconfig` file in your Windows home directory (`C:\Users\YourName\.wslconfig`) to control VM-level settings:

```ini
[wsl2]
# Limit RAM usage (default: 50% of total RAM)
memory=4GB

# Limit CPU cores (default: all cores)
processors=4

# Enable swap
swap=2GB

# Disable swap
# swap=0

# Custom kernel (optional)
# kernel=C:\\path\\to\\kernel

# Localhost forwarding
localhostForwarding=true

# Enable experimental features (Windows 11)
[experimental]
autoMemoryReclaim=gradual
networkingMode=mirrored
dnsTunneling=true
```

Apply changes:

```powershell
wsl --shutdown
wsl
```

> **`memory` is the most impactful setting.** Without it, WSL2 can consume up to 50% of your system RAM, causing Windows to slow down noticeably.

---

## 13. Common Errors & Fixes

### `Restart-Service LxssManager` — Service Not Found

```
Error: Cannot find any service with service name 'LxssManager'
```

**Fix:** The service was renamed. Use `WslService` instead:

```powershell
Restart-Service WslService
```

### `net stop WslService` — Could Not Be Stopped

```
The WSL Service service could not be stopped.
```

**Fix:** Stop the Hyper-V backend instead:

```powershell
net stop vmcompute
net start vmcompute
```

### Docker Integration Unavailable After Migration

**Fix:** Re-enable WSL integration in Docker Desktop settings for the migrated distro (see Section 9).

### `\\wsl.localhost` Inaccessible

**Fix:**

```powershell
wsl --shutdown
net stop vmcompute
net start vmcompute
wsl
```

### High CPU / RAM from `vmmem`

**Cause:** WSL2 is caching memory aggressively, or a runaway process inside Linux.

**Fix:**

```powershell
# Immediate fix — shut down WSL
wsl --shutdown

# Long-term fix — limit memory in .wslconfig
# memory=4GB
```

### WSL Stuck — Cannot Start or Stop

```powershell
# Force kill all WSL and VM processes
Get-Process -Name "vmmem*" -ErrorAction SilentlyContinue | Stop-Process -Force
Get-Process -Name "wsl*" -ErrorAction SilentlyContinue | Stop-Process -Force

# Then restart
net start vmcompute
wsl
```

---

## 14. Recovery Without Rebooting

When WSL becomes unresponsive, follow this sequence before resorting to a full Windows reboot:

### Step 1 — Shut down WSL cleanly

```powershell
wsl --shutdown
```

### Step 2 — Stop and restart the Hyper-V backend

```powershell
net stop vmcompute
net start vmcompute
```

### Step 3 — Start WSL again

```powershell
wsl
```

### Step 4 — If still stuck, force kill all processes

```powershell
Get-Process -Name "vmmem*" -ErrorAction SilentlyContinue | Stop-Process -Force
Get-Process -Name "wsl*" -ErrorAction SilentlyContinue | Stop-Process -Force
```

Then repeat Steps 2 and 3.

### Why This Works

```
Your WSL utility VM became stuck.
WslService was still active but couldn't manage the VM.

Restarting vmcompute:
  → Tears down the stuck Hyper-V VM
  → Recreates a clean backend
  → WSL can start fresh

Result: Full recovery without rebooting Windows.
```

> Restarting `vmcompute` is the most powerful WSL recovery tool. It fixes ~90% of WSL freeze and unresponsive scenarios.

---

## 15. Troubleshooting Matrix

| Problem | Symptom | Fix |
|---------|---------|-----|
| WSL won't start | `wsl` hangs or errors | `wsl --shutdown` → restart `vmcompute` |
| WslService won't stop | `net stop WslService` fails | Stop `vmcompute` instead |
| LxssManager not found | PowerShell error | Use `WslService` — it was renamed |
| vmmem high memory | Task Manager shows high RAM | `wsl --shutdown` + set `memory=` in `.wslconfig` |
| vmmem high CPU | Fan running constantly | Find runaway process inside WSL: `top` or `htop` |
| Docker unavailable | Docker Desktop errors | Verify WSL backend + re-enable distro in Docker settings |
| `\\wsl.localhost` broken | Can't browse Linux files | Restart WSL + `vmcompute` |
| Disk space low | C: drive filling up | Migrate distro to another drive or compact `ext4.vhdx` |
| Slow file access | Cross-filesystem lag | Move project files inside WSL filesystem |
| WSL completely frozen | Nothing works | Force kill `vmmem*` and `wsl*` processes |

---

## 16. Best Practices

| Practice | Why It Matters |
|----------|---------------|
| **Export backups before major changes** | Migration, upgrades, or experiments can go wrong — a `.tar` export is your safety net |
| **Store projects inside WSL filesystem** | `/home/user/` is dramatically faster than `/mnt/c/Users/...` |
| **Limit memory in `.wslconfig`** | Prevents WSL2 from consuming half your RAM while idle |
| **Use separate distros for experiments** | Keep a clean production distro; use a separate one for testing |
| **Shut down WSL when not in use** | Frees RAM and CPU — `wsl --shutdown` |
| **Compact `ext4.vhdx` periodically** | Reclaims disk space that Linux freed but Windows still holds |
| **Avoid force-killing unless necessary** | Ungraceful kills can corrupt the filesystem |
| **Keep Windows and WSL updated** | Many WSL bugs are fixed in Windows updates |
| **Use SSH keys, not passwords** | More secure for remote server access from WSL |
| **Understand WSL internals** | Knowing the `vmcompute → vmmem → kernel` chain saves hours of debugging |

---

## 17. Quick Reference Cheat Sheet

### WSL Control (PowerShell)

```powershell
wsl                          # Start default distro
wsl -d Ubuntu-22.04          # Start specific distro
wsl --shutdown               # Stop ALL distros
wsl -t Ubuntu-22.04          # Stop specific distro
wsl -l -v                    # List distros and status
wsl --set-default Ubuntu-22.04   # Set default distro
wsl --set-default-version 2      # Set WSL2 as default
```

### Service Recovery (PowerShell as Admin)

```powershell
net stop vmcompute           # Stop Hyper-V backend
net start vmcompute          # Start Hyper-V backend
net stop WslService          # Stop WSL manager
net start WslService         # Start WSL manager

# Force kill stuck processes
Get-Process -Name "vmmem*" -ErrorAction SilentlyContinue | Stop-Process -Force
Get-Process -Name "wsl*" -ErrorAction SilentlyContinue | Stop-Process -Force
```

### Export & Import

```powershell
wsl --export Ubuntu-22.04 D:\backup.tar          # Backup
wsl --unregister Ubuntu-22.04                     # Delete (destructive)
wsl --import Ubuntu-22.04 D:\WSL D:\backup.tar   # Restore/Migrate
```

### SSH (inside WSL terminal)

```bash
ssh-keygen -t ed25519 -C "email@example.com"     # Generate keys
cat ~/.ssh/id_ed25519.pub                         # View public key
ssh-copy-id user@server-ip                        # Copy key to server
ssh user@server-ip                                # Connect
chmod 700 ~/.ssh                                  # Fix permissions
chmod 600 ~/.ssh/authorized_keys
```

### Performance

```powershell
# In .wslconfig (C:\Users\Name\.wslconfig)
[wsl2]
memory=4GB
processors=4
```

```powershell
wsl --shutdown    # Apply .wslconfig changes
wsl               # Restart with new settings
```

---

*WSL_Lab — Based on The Complete WSL2 Handbook*
