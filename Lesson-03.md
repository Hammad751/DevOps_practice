# Creating the README markdown content for Lesson 3
content = """# Lesson 3: Linux Administration & Core Networking

This comprehensive guide serves as the official documentation and reference manual for Lesson 3. It covers foundational operating system architectures, Linux systems administration, advanced shell scripting, permissions management, and secure remote enterprise networking.

---

## 🎯 Course Objectives

By the end of this module, you will master:
1. Architectural paradigms of Operating Systems and Virtualization.
2. The Linux File System Structure (FHS standard).
3. Linux Command-Line Interface (CLI) proficiency & VIM advanced mechanics.
4. Enterprise Package Management and Dependency Resolution.
5. Linux Multi-User Administration, Ownership, and Discretionary Access Control (DAC).
6. Enterprise Automation via Bash Shell Scripting.
7. System-Wide & Session-Specific Environment Variables.
8. Core Networking Infrastructure (TCP/IP stack, Subnetting, and Diagnosis).
9. Cryptographic Remote Server Access via SSH Architecture & Hardening.

---

## 💻 1. Operating Systems & Virtualization

### Operating System (OS) Fundamentals
An Operating System acts as the intermediary software layer managing hardware resources and presenting abstract execution spaces for software processes. Its critical subsystems include the Kernel (CPU/Memory management), File System, Process Scheduler, and I/O Device Drivers.

### Virtualization Paradigms
Virtualization enables the execution of multiple isolated OS instances simultaneously on a single physical host by abstracting the physical hardware.

* **Hypervisor Type-1 (Bare-Metal):** Runs directly on the host hardware (e.g., VMware ESXi, Proxmox VE, KVM). Standard for production DevOps infrastructures due to high performance and low overhead.
* **Hypervisor Type-2 (Hosted):** Runs as an application layer on top of an existing Host OS (e.g., VirtualBox, VMware Workstation). Ideal for local development environments.

---

## 🗂️ 2. Linux File System Hierarchy Standard (FHS)

Unlike Windows (which uses drive letters like `C:`), Linux organizes everything into a unified single-root tree structure starting at the `/` (root) directory.

| Directory | Purpose / Technical Function |
| :--- | :--- |
| `/` | The primary root directory of the entire system hierarchy. |
| `/bin` | Essential user command binaries required in single-user mode (e.g., `cat`, `ls`, `cp`). |
| `/sbin` | Essential system administration binaries restricted to the superuser (e.g., `iptables`, `fdisk`). |
| `/etc` | Host-specific system-wide configuration files (e.g., `/etc/passwd`, `/etc/ssh/sshd_config`). |
| `/var` | Variable data files that change dynamically during runtime (logs, spool directories, databases). |
| `/home` | User home directories storing personal configurations and documents. |
| `/root` | Home directory specifically reserved for the administrative `root` superuser. |
| `/opt` | Add-on application software packages installed outside system defaults. |
| `/proc` | Virtual pseudo-filesystem providing a window into the live Linux Kernel and running processes. |

---

## 🛠️ 3. Essential Linux CLI Commands

| Category | Command | Syntax / Example | Functional Description |
| :--- | :--- | :--- | :--- |
| **Navigation** | `pwd` | `pwd` | Prints the absolute path of the current working directory. |
| | `cd` | `cd /var/log` | Changes the current working directory space. |
| | `ls` | `ls -la` | Lists directory contents showing hidden files and detailed metadata. |
| **File Ops** | `mkdir` | `mkdir -p project/src` | Creates a new directory path recursively if parent paths do not exist. |
| | `touch` | `touch app.js` | Creates an empty file or updates the access timestamp of an existing file. |
| | `cp` | `cp -r src/ dist/` | Copies files or complete directories recursively. |
| | `mv` | `mv source.txt /tmp/` | Moves or renames files/directories within the file system. |
| | `rm` | `rm -rf /tmp/cache` | Forcefully deletes files or entire directories recursively (Use with caution). |
| **Inspection**| `cat` | `cat /etc/hostname` | Concatenates and displays the entire contents of a file to standard output. |
| | `less` | `less /var/log/syslog` | Opens a file for interactive page-by-page viewing without loading it all to memory. |
| | `grep` | `grep "ERROR" app.log` | Searches files for specific text patterns matching regular expressions. |

---

## 📦 4. Package Management & Dependency Resolution

Linux distributions use package managers to automate the installation, updating, configuration, and removal of software binaries and their underlying prerequisites.

### Debian/Ubuntu Advanced Package Tool (APT)
* Update native repository indices: `sudo apt update`
* Install software and dependencies: `sudo apt install curl openjdk-11-jdk`
* Remove packages along with configurations: `sudo apt purge nginx`

### RedHat/CentOS Yellowdog Updater Modified (YUM/DNF)
* Install software packages: `sudo dnf install httpd`

---

## 📝 5. VIM Editor Architecture

VIM (Vi Improved) is a modal terminal text editor standard across remote Linux instances. Efficient usage requires manipulating its modes:

1.  **Normal Mode (Default):** For navigation and structural file manipulation.
    * `i` -> Switch to **Insert Mode** to write text.
    * `:` -> Switch to **Command-Line Mode**.
    * `x` -> Delete a single character under the cursor.
    * `dd` -> Delete (cut) the entire current line.
2.  **Insert Mode:** For direct text input. Press `ESC` to return to Normal Mode.
3.  **Command-Line Mode:** For saving, exiting, and execution.
    * `:w` -> Save changes (write).
    * `:q!` -> Quit without saving modifications.
    * `:wq` -> Save changes and close the editor immediately.

---

## 🔒 6. Linux Multi-User Accounts & Access Control

### Discretionary Access Control (DAC) Metadata
Every file and directory in Linux has predefined permission bindings categorized into three scopes: **User (Owner)**, **Group**, and **Others**.

Permissions possess distinct symbolic mappings and absolute octal values:
* `r` (Read) = **4**
* `w` (Write) = **2**
* `x` (Execute) = **1**

### Numerical Octal Permission Math
Permissions are computed as the mathematical sum of the enabled modes:
* Full Permissions (`rwx`) = $4 + 2 + 1 = 7$
* Read & Write Only (`rw-`) = $4 + 2 + 0 = 6$
* Read & Execute Only (`r-x`) = $4 + 0 + 1 = 5$

Example: `chmod 755 script.sh` assigns `rwx` (7) to Owner, `r-x` (5) to Group, and `r-x` (5) to Others.

### Administration Commands
* **Change File Permissions:** `chmod 644 .env`
* **Change Ownership:** `sudo chown devops:webgroup server.log`
* **Create System User:** `sudo useradd -m -s /bin/bash hammad`

---

## 📜 7. Shell Scripting & Advanced Bash Automation

### Shell vs. Bash
* **Shell:** The generic abstract command-line interface framework that interprets inputs to execute system calls on the core Linux Kernel (e.g., Sh, Zsh, Fish).
* **Bash (Bourne Again Shell):** A specific, highly standardized implementation of a POSIX-compliant shell that serves as the default engine across enterprise Linux distributions.

### Bash Scripting Syntax & Concepts
A bash script must always start with a **Shebang** (`#!/bin/bash`) on the first line to instruct the kernel which interpreter engine to spawn.

#### Example Automation Script (`deploy.sh`):
