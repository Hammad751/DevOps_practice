# Lesson 15: Introduction to Shell Scripting

> Shell scripting is the practice of writing a sequence of commands into a reusable file. Instead of typing the same commands repeatedly across multiple systems, you write them once in a script and execute it anywhere — making it a cornerstone of automation, DevOps, and system administration.

---

## Table of Contents

1. [What is a Shell Script?](#1-what-is-a-shell-script)
2. [What is a Shell?](#2-what-is-a-shell)
3. [Common Shell Implementations in Linux](#3-common-shell-implementations-in-linux)
4. [The Shebang Line](#4-the-shebang-line)
5. [Creating Your First Shell Script](#5-creating-your-first-shell-script)
6. [Running a Shell Script](#6-running-a-shell-script)
7. [Shell Script Fundamentals — Variables, Conditions & Loops](#7-shell-script-fundamentals--variables-conditions--loops)
8. [User Input & Arguments](#8-user-input--arguments)
9. [Functions in Shell Scripts](#9-functions-in-shell-scripts)
10. [Real-World Example: Server Setup Script](#10-real-world-example-server-setup-script)
11. [Best Practices](#11-best-practices)
12. [Quick Reference Cheat Sheet](#12-quick-reference-cheat-sheet)

---

## 1. What is a Shell Script?

A **shell script** is a plain text file containing a sequence of Linux commands that the shell reads and executes line by line — exactly as if you typed each command manually in the terminal.

```
WITHOUT a script:                    WITH a script:
─────────────────                    ──────────────
$ sudo apt update                    $ ./setup.sh
$ sudo apt install -y nginx               │
$ sudo systemctl start nginx              ▼
$ sudo systemctl enable nginx      Runs ALL commands
$ sudo ufw allow 'Nginx Full'      automatically, in
$ echo "Done"                      order, every time.
                                   On any machine.
```

### Why Use Shell Scripts?

| Benefit | Description |
|---------|-------------|
| **Automation** | Eliminate repetitive manual commands |
| **Consistency** | Same commands run the same way every time |
| **Reusability** | Write once, run on any compatible system |
| **Speed** | Provision a server in seconds instead of minutes |
| **Auditability** | Scripts are readable — you can see exactly what ran |

> **Key Requirement:** Both systems must have the same or compatible configuration for a script to work correctly across machines.

---

## 2. What is a Shell?

The **shell** is the command-line interpreter that sits between you and the Linux kernel:

```
┌──────────────────────────────────────────────────────┐
│                     USER                             │
│              (Types commands)                        │
└───────────────────────┬──────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────┐
│                    SHELL                             │
│         (Interprets & translates commands)           │
│              e.g., Bash, sh, zsh                     │
└───────────────────────┬──────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────┐
│                 LINUX KERNEL                         │
│        (Executes system-level operations)            │
└──────────────────────────────────────────────────────┘
```

- The shell **reads** commands from the terminal or a script file.
- It **translates** those commands into instructions the OS kernel can understand.
- It **returns** the output back to the user.

---

## 3. Common Shell Implementations in Linux

Linux comes with multiple shell programs. They share the same core syntax but differ in features:

| Shell | Path | Description |
|-------|------|-------------|
| **sh** (Bourne Shell) | `/bin/sh` | The original Unix shell. Minimal, portable, available on every Unix/Linux system. |
| **Bash** (Bourne Again Shell) | `/bin/bash` | The most widely used shell. Feature-rich improvement of `sh`. Default on most Linux distros. |
| **zsh** (Z Shell) | `/bin/zsh` | Extended Bash with better autocomplete, theming (Oh My Zsh). Default on macOS. |
| **fish** | `/usr/bin/fish` | Beginner-friendly with syntax highlighting and suggestions built in. |
| **dash** | `/bin/dash` | Lightweight, fast — often used as `/bin/sh` on Ubuntu for boot scripts. |

### Bash vs. sh

```
sh  ──────────────────────────► Bash
(1979, original Bourne Shell)   (1989, Bourne Again Shell)
                                  ↑
                          Superset of sh:
                          • Arrays
                          • String manipulation
                          • Extended conditionals
                          • Command history
                          • Tab completion
```

> **Bash and Shell are often used interchangeably** in practice — but technically, Bash is *one specific* shell implementation. Bash is simultaneously:
> - A **shell program** (command interpreter)
> - A **programming language** (with variables, loops, conditionals, functions)

---

## 4. The Shebang Line

### What is it?

When a Linux system encounters a script file, it needs to know **which interpreter** to use to run it — since multiple shells (`sh`, `bash`, `zsh`, etc.) may be installed. The **shebang** solves this.

```bash
#!/bin/bash
```

### Rules

- Must be the **very first line** of the script — line 1, character 1, no exceptions.
- Starts with `#!` (hash + exclamation mark — pronounced "shebang" or "hashbang").
- Followed by the **absolute path** to the interpreter.
- The OS reads this line before executing anything else.

### Common Shebang Lines

```bash
#!/bin/bash        # Use Bash — most common for Linux scripts
#!/bin/sh          # Use sh — more portable across Unix/Linux systems
#!/usr/bin/env bash  # Find bash wherever it is — most portable
#!/usr/bin/python3   # Python script
#!/usr/bin/env node  # Node.js script
```

> **Best Practice for Portability:** Use `#!/usr/bin/env bash` instead of `#!/bin/bash`. The `env` command searches your `$PATH` for `bash`, making the script work even if bash is installed in a non-standard location (common on macOS or some Linux distros).

---

## 5. Creating Your First Shell Script

### Step 1 — Create the file

```bash
vim setup.sh
```

### Step 2 — Write the script

```bash
#!/bin/bash
# My first shell script
# This script updates the system and installs nginx

echo "Starting server setup..."

# Update package index
sudo apt update

# Install nginx
sudo apt install -y nginx

# Start and enable nginx
sudo systemctl start nginx
sudo systemctl enable nginx

echo "Setup complete! Nginx is running."
```

> Lines starting with `#` are **comments** — they are ignored by the interpreter but explain the code to humans. Good commenting is a professional habit.

### Step 3 — Save and exit Vim

```
Press ESC → type :wq → press Enter
```

### Step 4 — Check current permissions

```bash
ls -l setup.sh
# Output:
# -rw-r--r--  1  alice  alice  245  Jul 20  10:00  setup.sh
#  ───
#  No x (execute) permission — can't run yet!
```

### Step 5 — Add execute permission

```bash
# Grant execute permission to the owner
sudo chmod u+x setup.sh

# Verify the permission was added
ls -l setup.sh
# Output:
# -rwxr--r--  1  alice  alice  245  Jul 20  10:00  setup.sh
#   ─
#   x is now present for the owner
```

### Step 6 — Execute the script

```bash
./setup.sh
```

---

## 6. Running a Shell Script

There are two methods to execute a shell script:

### Method 1 — Direct Execution (Universal)

```bash
./setup.sh
```

- The `./` tells the shell to look in the **current directory** for the file.
- Requires **execute permission** (`chmod u+x`) on the file.
- Uses whatever interpreter is specified in the **shebang line**.
- Works for **any script type** — bash, python, node, etc.

### Method 2 — Explicit Interpreter (Bash-specific)

```bash
bash setup.sh
```

- Passes the script directly to the `bash` interpreter.
- Does **not require** execute permission on the file.
- Ignores the shebang line — always uses bash regardless.
- Only works for bash scripts (not universal).

### Comparison

| | `./setup.sh` | `bash setup.sh` |
|--|-------------|----------------|
| Execute permission needed? | ✅ Yes | ❌ No |
| Reads shebang line? | ✅ Yes | ❌ No (forces bash) |
| Works for any script type? | ✅ Yes | ❌ Bash only |
| Recommended for? | Production use | Quick testing |

### Running with Elevated Privileges

```bash
# Run as root (for system-level operations)
sudo ./setup.sh

# Run as a different user
sudo -u alice ./setup.sh
```

### Running in Debug Mode

```bash
# Print each command before executing it (great for troubleshooting)
bash -x setup.sh

# Or add to the shebang line
#!/bin/bash -x
```

---

## 7. Shell Script Fundamentals — Variables, Conditions & Loops

### Variables

```bash
#!/bin/bash

# Declare a variable (no spaces around =)
APP_NAME="nginx"
VERSION="1.24"
PORT=80

# Use a variable with $
echo "Installing $APP_NAME version $VERSION on port $PORT"

# Capture command output into a variable
CURRENT_USER=$(whoami)
echo "Running as: $CURRENT_USER"
```

> **Convention:** Use `UPPERCASE` for constants and environment variables, `lowercase` for local variables.

### Conditional Statements

```bash
#!/bin/bash

# Check if a package is installed
if [ $(which nginx) ]; then
    echo "Nginx is already installed."
else
    echo "Installing Nginx..."
    sudo apt install -y nginx
fi

# Check if a file exists
if [ -f "/etc/nginx/nginx.conf" ]; then
    echo "Config file exists."
fi

# Check if a directory exists
if [ -d "/var/www/html" ]; then
    echo "Web root exists."
fi
```

### Common Comparison Operators

| Type | Operator | Meaning |
|------|----------|---------|
| Numbers | `-eq` | Equal |
| | `-ne` | Not equal |
| | `-lt` | Less than |
| | `-gt` | Greater than |
| Strings | `=` | Equal |
| | `!=` | Not equal |
| | `-z` | Is empty |
| Files | `-f` | File exists |
| | `-d` | Directory exists |
| | `-x` | File is executable |

### Loops

```bash
#!/bin/bash

# For loop — iterate over a list
for PACKAGE in git curl wget vim; do
    echo "Installing $PACKAGE..."
    sudo apt install -y $PACKAGE
done

# While loop — run until condition is false
COUNT=1
while [ $COUNT -le 5 ]; do
    echo "Attempt $COUNT"
    COUNT=$((COUNT + 1))
done
```

---

## 8. User Input & Arguments

### Reading User Input

```bash
#!/bin/bash

echo "Enter your username:"
read USERNAME

echo "Hello, $USERNAME!"
```

### Passing Arguments to a Script

```bash
# Run the script with arguments
./setup.sh alice devops 8080
```

Inside the script, arguments are accessed with `$1`, `$2`, `$3`, etc.:

```bash
#!/bin/bash

USERNAME=$1      # First argument:  alice
GROUP=$2         # Second argument: devops
PORT=$3          # Third argument:  8080

echo "Creating user $USERNAME in group $GROUP on port $PORT"

# $0 = the script name itself
# $# = total number of arguments passed
# $@ = all arguments as a list
```

---

## 9. Functions in Shell Scripts

Functions allow you to group commands into reusable blocks:

```bash
#!/bin/bash

# Define a function
install_package() {
    echo "Installing $1..."
    sudo apt install -y $1
    echo "$1 installed successfully."
}

check_status() {
    if systemctl is-active --quiet $1; then
        echo "$1 is running."
    else
        echo "$1 is NOT running."
    fi
}

# Call the functions
install_package nginx
install_package curl
check_status nginx
```

---

## 10. Real-World Example: Server Setup Script

Here is a complete, professional-grade setup script combining everything from this lesson:

```bash
#!/bin/bash
# =============================================================
# Server Setup Script
# Description: Installs and configures a basic web server
# Usage: sudo ./setup.sh <environment>
# =============================================================

# --- Variables ---
ENVIRONMENT=$1
APP_USER="webadmin"
PACKAGES="nginx curl git ufw"
LOG_FILE="/var/log/setup.log"

# --- Functions ---
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a $LOG_FILE
}

install_packages() {
    log "Updating package index..."
    sudo apt update

    for PKG in $PACKAGES; do
        log "Installing $PKG..."
        sudo apt install -y $PKG
    done
}

configure_firewall() {
    log "Configuring firewall..."
    sudo ufw allow OpenSSH
    sudo ufw allow 'Nginx Full'
    sudo ufw --force enable
}

# --- Main Execution ---
log "=== Starting setup for environment: $ENVIRONMENT ==="

install_packages
configure_firewall

sudo systemctl start nginx
sudo systemctl enable nginx

log "=== Setup complete! ==="
```

**Run it:**

```bash
chmod u+x setup.sh
sudo ./setup.sh production
```

---

## 11. Best Practices

| Practice | Why It Matters |
|----------|---------------|
| Always include a shebang line | Ensures the correct interpreter is used |
| Add comments to explain intent | Scripts are read by humans too |
| Use `set -e` at the top | Exit immediately if any command fails — prevents silent errors |
| Use `set -u` at the top | Treat unset variables as errors — catches typos |
| Quote your variables: `"$VAR"` | Prevents word-splitting bugs with spaces in values |
| Use meaningful variable names | `APP_PORT` is clearer than `p` |
| Log important steps | Makes debugging and auditing easier |
| Test with `bash -x` before deploying | Prints each command as it runs |

```bash
#!/bin/bash
set -e    # Exit on error
set -u    # Error on undefined variables
set -o pipefail   # Catch errors in pipes too
```

---

## 12. Quick Reference Cheat Sheet

### File Operations

```bash
vim setup.sh          # Create/edit script
ls -l setup.sh        # Check permissions
chmod u+x setup.sh    # Add execute permission
./setup.sh            # Run script (universal)
bash setup.sh         # Run with bash explicitly
bash -x setup.sh      # Run in debug mode
```

### Script Structure

```bash
#!/bin/bash            # Shebang — always first line
# Comment             # Ignored by interpreter

VARIABLE="value"       # Variable assignment (no spaces)
echo $VARIABLE         # Use variable

if [ condition ]; then
    # commands
fi

for ITEM in list; do
    # commands
done

function_name() {
    # commands
}
```

### Special Variables

| Variable | Meaning |
|----------|---------|
| `$0` | Script name |
| `$1`, `$2`... | Positional arguments |
| `$#` | Number of arguments |
| `$@` | All arguments |
| `$?` | Exit code of last command (`0` = success) |
| `$$` | PID of current script |
| `$USER` | Current logged-in user |
| `$HOME` | Home directory path |
| `$PWD` | Current working directory |

---

*End of Lesson 15*
