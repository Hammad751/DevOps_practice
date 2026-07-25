# Lesson 17: Environment Variables

> Environment Variables (ENVs) are dynamic named values that exist within a user's or system's environment. They influence the behavior of programs and processes, pass configuration to applications, and define how the system and software interact — without hardcoding values into code.

---

## Table of Contents

1. [What Are Environment Variables?](#1-what-are-environment-variables)
2. [Viewing Environment Variables](#2-viewing-environment-variables)
3. [Creating & Exporting Variables](#3-creating--exporting-variables)
4. [Deleting a Variable](#4-deleting-a-variable)
5. [Making Variables Persistent](#5-making-variables-persistent)
6. [User-Specific vs. Global Variables](#6-user-specific-vs-global-variables)
7. [The PATH Variable](#7-the-path-variable)
8. [Adding a Custom Program to PATH](#8-adding-a-custom-program-to-path)
9. [Environment Variables in Shell Scripts](#9-environment-variables-in-shell-scripts)
10. [Common Built-in Environment Variables](#10-common-built-in-environment-variables)
11. [Quick Reference Cheat Sheet](#11-quick-reference-cheat-sheet)

---

## 1. What Are Environment Variables?

```
┌──────────────────────────────────────────────────────────────┐
│                  LINUX ENVIRONMENT                           │
│                                                              │
│   User: alice                    User: bob                   │
│   ┌───────────────────┐          ┌───────────────────┐       │
│   │ HOME=/home/alice  │          │ HOME=/home/bob    │       │
│   │ USER=alice        │          │ USER=bob          │       │
│   │ DB_USER=alice_db  │          │ DB_USER=bob_db    │       │
│   │ PATH=...          │          │ PATH=...          │       │
│   └───────────────────┘          └───────────────────┘       │
│                                                              │
│   Same OS — different environments, different variables.     │
└──────────────────────────────────────────────────────────────┘
```

### Key Characteristics

- **Environment-specific:** Each user session has its own set of ENVs.
- **Dynamic:** Can be set, modified, or removed at any time.
- **Inherited:** Child processes inherit environment variables from their parent process.
- **Convention:** Environment variable names are written in **UPPERCASE** by convention (e.g., `DB_USER`, `HOME`, `PATH`).
- **Influential:** Programs read ENVs at runtime to configure their behavior — databases, web servers, and CLIs all rely on them.

### Why They Matter in DevOps

Environment variables are the standard way to:

| Use Case | Example |
|----------|---------|
| Store credentials safely | `DB_PASSWORD=secret` |
| Configure app behavior per environment | `APP_ENV=production` |
| Set runtime paths | `JAVA_HOME=/usr/lib/jvm/java-17` |
| Pass config to Docker containers | `docker run -e DB_HOST=localhost` |
| CI/CD pipeline secrets | GitHub Actions secrets → ENV |

---

## 2. Viewing Environment Variables

### List All Environment Variables

```bash
# Print all environment variables
printenv

# Paginate the output (easier to read)
printenv | less

# Alternative — also shows shell variables
env
```

### Check a Specific Variable

```bash
# Using printenv (variable name must be UPPERCASE)
printenv USER
printenv HOME
printenv PATH

# Using echo with the $ sign
echo $USER
echo $HOME
echo $PATH

# Using printenv for a custom variable
printenv DB_USER
```

> **Note:** `printenv VARIABLE` and `echo $VARIABLE` both work, but `printenv` is cleaner for scripts since it won't print anything (and returns exit code 1) if the variable doesn't exist, while `echo $UNDEFINED` silently prints a blank line.

---

## 3. Creating & Exporting Variables

### Step 1 — Assign a value

```bash
DB_USER=dbuser
```

This creates a **shell variable** — it exists only in the current shell session and is **not** available to child processes.

### Step 2 — Export it to the environment

```bash
export DB_USER=dbuser
```

`export` promotes the variable to an **environment variable**, making it available to the current shell AND all child processes spawned from it.

```bash
# Both steps in one line
export DB_USER=dbuser
export DB_PASSWORD=secretpass
export APP_PORT=8080
export APP_ENV=development
```

### Verify the variable was set

```bash
echo $DB_USER
# Output: dbuser

printenv DB_USER
# Output: dbuser
```

### Shell Variable vs Environment Variable

```
DB_USER=dbuser          →  Shell variable only
                           (child processes cannot see it)

export DB_USER=dbuser   →  Environment variable
                           (current shell + all child processes inherit it)
```

```bash
# Demonstration
MY_VAR="hello"
bash -c 'echo $MY_VAR'       # prints nothing — not exported

export MY_VAR="hello"
bash -c 'echo $MY_VAR'       # prints: hello
```

---

## 4. Deleting a Variable

```bash
# Remove a variable from the environment
unset DB_USER

# Verify it's gone
echo $DB_USER        # prints nothing
printenv DB_USER     # prints nothing, returns exit code 1
```

> **Note:** `unset` only removes the variable for the current session. If it was defined in `.bashrc` or `/etc/environment`, it will come back on the next login unless you remove it from those files too.

---

## 5. Making Variables Persistent

Variables set with `export` in the terminal are **session-only** — they disappear when you close the terminal or log out.

```
Terminal session starts  →  export DB_USER=dbuser  →  Terminal closes  →  DB_USER is gone
```

To make them **persist across sessions**, add them to a configuration file.

### For a Single User — `.bashrc`

`.bashrc` is executed every time a new interactive bash shell starts for that user.

```bash
# Open the file
vim ~/.bashrc
```

Add your variables at the bottom:

```bash
# Custom environment variables
export DB_USER=dbuser
export DB_PASSWORD=secretpass
export APP_ENV=production
export APP_PORT=8080
```

Save and exit (`:wq`), then apply the changes immediately without logging out:

```bash
source ~/.bashrc
```

> `source` (or `. ~/.bashrc`) re-executes the file in the current shell session, loading the new variables right away.

### Other User Config Files

| File | When It Runs |
|------|-------------|
| `~/.bashrc` | Every new interactive bash shell |
| `~/.bash_profile` | Once at login (SSH, terminal login) |
| `~/.profile` | Login shell — works with sh, bash, and others |
| `~/.zshrc` | Every new zsh shell (if using zsh) |

---

## 6. User-Specific vs. Global Variables

### User-Specific (`.bashrc`)

Variables added to `~/.bashrc` are only available to **that specific user**. Other users on the same system cannot see them.

```
alice's ~/.bashrc  →  DB_USER=alice_db   (only alice sees this)
bob's   ~/.bashrc  →  DB_USER=bob_db     (only bob sees this)
```

### Global (System-Wide) — `/etc/environment`

To make environment variables available to **all users** on the system, add them to `/etc/environment`:

```bash
# Open the global environment file
sudo vim /etc/environment
```

The file uses a simple `KEY=VALUE` format (no `export` keyword needed):

```
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
APP_ENV="production"
DB_HOST="localhost"
```

Save and either reboot or log out and back in for the changes to take effect for all users.

> **Security Warning:** Never store sensitive credentials like passwords or API keys in `/etc/environment` — it is readable by all users on the system. Use a secrets manager or user-specific files instead.

### Comparison

| Scope | File | Who Sees It |
|-------|------|-------------|
| Current session only | `export VAR=value` in terminal | Current user, current session |
| Single user, persistent | `~/.bashrc` | That user only, all future sessions |
| All users, persistent | `/etc/environment` | Every user on the system |

---

## 7. The PATH Variable

`PATH` is one of the most important environment variables in Linux. It is a colon-separated list of directories that the shell searches through when you type a command.

```bash
echo $PATH
# Output:
# /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games
```

### How PATH Works

```
User types: nginx

Shell searches:
  /usr/local/sbin/nginx  ← not found
  /usr/local/bin/nginx   ← not found
  /usr/sbin/nginx        ← FOUND! Execute it.
```

If a binary is not in any PATH directory, the shell returns:
```
command not found
```

### Viewing PATH Directories Clearly

```bash
# Replace : with newlines for easier reading
echo $PATH | tr ':' '\n'

# Output:
# /usr/local/sbin
# /usr/local/bin
# /usr/sbin
# /usr/bin
# /sbin
# /bin
```

---

## 8. Adding a Custom Program to PATH

When you install a custom tool or write your own script, you need to add its directory to PATH so you can run it from anywhere without typing the full path.

### Method 1 — Temporarily (current session only)

```bash
export PATH=$PATH:/path/to/your/custom/bin
```

The `$PATH` at the beginning **preserves** all existing directories — you are appending, not replacing.

```bash
# Example: add a custom scripts directory
export PATH=$PATH:/home/alice/scripts

# Now you can run any script in that folder from anywhere
my-deploy-script.sh
```

### Method 2 — Permanently for one user

Add to `~/.bashrc`:

```bash
vim ~/.bashrc
```

```bash
# Add custom bin directory to PATH
export PATH=$PATH:/home/alice/scripts
export PATH=$PATH:/opt/custom-tools/bin
```

```bash
source ~/.bashrc
```

### Method 3 — Permanently for all users

Add to `/etc/environment`:

```bash
sudo vim /etc/environment
```

```
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/custom-tools/bin"
```

> Always **append** to PATH (`$PATH:/new/dir`), never replace it — removing system directories from PATH will break basic commands like `ls` and `cd`.

---

## 9. Environment Variables in Shell Scripts

### Accessing ENVs in a Script

Scripts automatically inherit environment variables from the shell that runs them:

```bash
#!/bin/bash

echo "Running as user:    $USER"
echo "Home directory:     $HOME"
echo "Database user:      $DB_USER"
echo "App environment:    $APP_ENV"
```

### Setting Variables for a Single Script Run

Prefix the command with variable assignments — they apply only to that one execution:

```bash
APP_ENV=staging DB_USER=test_user ./deploy.sh
```

### Checking if a Variable is Set

```bash
#!/bin/bash

if [ -z "$DB_PASSWORD" ]; then
    echo "ERROR: DB_PASSWORD is not set. Exiting."
    exit 1
fi

echo "Connecting to database as $DB_USER..."
```

### Loading a `.env` File in a Script

In DevOps, it is common to store all project variables in a `.env` file and load them into a script:

```bash
# .env file contents:
# DB_USER=myapp
# DB_PASSWORD=secretpass
# APP_PORT=3000
```

```bash
#!/bin/bash

# Load the .env file
source .env

# Or using the dot shorthand
. .env

echo "App running on port $APP_PORT"
echo "DB user: $DB_USER"
```

> **Security:** Never commit a `.env` file containing real credentials to version control (Git). Add `.env` to your `.gitignore`.

---

## 10. Common Built-in Environment Variables

These are automatically set by the system and available in every session:

| Variable | Description | Example Value |
|----------|-------------|---------------|
| `USER` | Currently logged-in username | `alice` |
| `HOME` | Home directory of current user | `/home/alice` |
| `PATH` | Directories searched for commands | `/usr/bin:/bin:...` |
| `SHELL` | Path to the current shell | `/bin/bash` |
| `PWD` | Current working directory | `/home/alice/project` |
| `OLDPWD` | Previous working directory | `/home/alice` |
| `HOSTNAME` | Machine's hostname | `web-server-01` |
| `LANG` | System language and locale | `en_US.UTF-8` |
| `TERM` | Terminal type | `xterm-256color` |
| `EDITOR` | Default text editor | `vim` |
| `LOGNAME` | Login name of the user | `alice` |
| `UID` | User ID of current user | `1001` |

```bash
# Quick check of common variables
echo "User:     $USER"
echo "Home:     $HOME"
echo "Shell:    $SHELL"
echo "Location: $PWD"
echo "Host:     $HOSTNAME"
```

---

## 11. Quick Reference Cheat Sheet

### Viewing Variables

```bash
printenv              # List all environment variables
printenv USER         # Check a specific variable
echo $USER            # Alternative way to check
env                   # List all (includes shell variables)
```

### Setting Variables

```bash
MY_VAR=value          # Shell variable (current shell only)
export MY_VAR=value   # Environment variable (inherited by child processes)
```

### Deleting Variables

```bash
unset MY_VAR          # Remove a variable from the environment
```

### Making Variables Persistent

```bash
vim ~/.bashrc                    # User-specific (single user)
source ~/.bashrc                 # Apply changes immediately

sudo vim /etc/environment        # System-wide (all users)
```

### Managing PATH

```bash
echo $PATH                       # View current PATH
export PATH=$PATH:/new/dir       # Append a directory (session only)
echo $PATH | tr ':' '\n'         # View PATH directories one per line
```

### In Scripts

```bash
source .env                      # Load a .env file into the script
if [ -z "$VAR" ]; then ...       # Check if a variable is unset/empty
APP_ENV=staging ./deploy.sh      # Set variable for a single run only
```

---

*End of Lesson 17*
