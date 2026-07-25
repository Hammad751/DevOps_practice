# Lesson 16: Shell Scripting II — Variables, Conditionals, Loops & Functions

> This lesson builds on the Shell Scripting fundamentals from Lesson 15, diving deeper into variables, conditional logic, user input, positional parameters, loops, and functions — the core building blocks of any real-world automation script.

---

## Table of Contents

1. [Variables in Shell Scripts](#1-variables-in-shell-scripts)
2. [Conditional Statements — if/else](#2-conditional-statements--ifelse)
3. [File Test Operators](#3-file-test-operators)
4. [Number Comparison Operators](#4-number-comparison-operators)
5. [String Operators](#5-string-operators)
6. [User Input — Arguments & Prompts](#6-user-input--arguments--prompts)
7. [Positional Parameters](#7-positional-parameters)
8. [Loops in Shell Scripts](#8-loops-in-shell-scripts)
9. [Functions & Return Values](#9-functions--return-values)
10. [Double Brackets & Portability](#10-double-brackets--portability)
11. [Complete Example Script](#11-complete-example-script)
12. [Quick Reference Cheat Sheet](#12-quick-reference-cheat-sheet)

---

## 1. Variables in Shell Scripts

### Declaring Variables

```bash
# No spaces around the = sign — this is mandatory
file_name="config.yaml"
user_name="alice"
port=8080

# WRONG — will throw an error
file_name = "config.yaml"   # spaces not allowed
```

### Using Variables

```bash
# Reference a variable with the $ sign
echo "Config file is $file_name"
echo "User: $user_name running on port $port"
```

### Capturing Command Output into a Variable

Use `$()` to run a command and store its output directly in a variable:

```bash
# Store the output of the ls command into a variable
config_file=$(ls config)

# Store current date
today=$(date)

# Store current user
current_user=$(whoami)

echo "Files in config: $config_file"
echo "Script run by: $current_user on $today"
```

> **Rule:** No spaces around `=`. Always wrap variable usage in double quotes (`"$var"`) to handle values that may contain spaces.

---

## 2. Conditional Statements — if/else

### Basic Syntax

```bash
if [ condition ]; then
    # commands if condition is TRUE
else
    # commands if condition is FALSE
fi
```

> `fi` closes the `if` block — it is `if` spelled backwards.

### Real-World Example: Check if a Directory Exists

```bash
#!/bin/bash

if [ -d "config" ]; then
    echo "Config directory found. Reading files..."
    config_file=$(ls config)
    echo "Files: $config_file"
else
    echo "Config directory not found. Creating it..."
    mkdir config
    echo "Directory created successfully."
fi
```

### if / elif / else

```bash
#!/bin/bash

score=$1

if [ $score -ge 90 ]; then
    echo "Grade: A"
elif [ $score -ge 75 ]; then
    echo "Grade: B"
elif [ $score -ge 60 ]; then
    echo "Grade: C"
else
    echo "Grade: F — Please retry."
fi
```

---

## 3. File Test Operators

File test operators check properties of files and directories inside `[ ]` conditions.

| Operator | Meaning |
|----------|---------|
| `-d file` | True if file is a directory |
| `-f file` | True if file exists and is a regular file |
| `-e file` | True if file exists (any type) |
| `-r file` | True if file is readable |
| `-w file` | True if file is writable |
| `-x file` | True if file is executable |
| `-s file` | True if file exists and is not empty |
| `-z file` | True if file has zero size (is empty) |
| `! -d file` | True if file does NOT exist as a directory |

```bash
# Examples
if [ -f "config.yaml" ]; then
    echo "Config file exists."
fi

if [ -x "deploy.sh" ]; then
    echo "Script is executable."
fi

if [ ! -d "logs" ]; then
    mkdir logs
    echo "Logs directory created."
fi
```

> For a full list of test operators, refer to:
> [Unix File & Test Operators — TutorialsPoint](https://www.tutorialspoint.com/unix/unix-file-operators.htm)

---

## 4. Number Comparison Operators

In shell scripting, use keyword operators instead of `<`, `>`, `==` for comparing numbers:

| Operator | Meaning | Example |
|----------|---------|---------|
| `-eq` | Equal to | `[ $a -eq 10 ]` |
| `-ne` | Not equal to | `[ $a -ne 0 ]` |
| `-lt` | Less than | `[ $a -lt 100 ]` |
| `-le` | Less than or equal | `[ $a -le 50 ]` |
| `-gt` | Greater than | `[ $a -gt 0 ]` |
| `-ge` | Greater than or equal | `[ $a -ge 18 ]` |

```bash
#!/bin/bash

num=10

if [ $num -eq 10 ]; then
    echo "Number is exactly 10"
fi

if [ $num -gt 5 ]; then
    echo "Number is greater than 5"
fi

if [ $num -ne 0 ]; then
    echo "Number is non-zero"
fi
```

---

## 5. String Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` | Equal — POSIX, portable across all shells | `[ "$a" = "hello" ]` |
| `==` | Equal — Bash-specific | `[ "$a" == "hello" ]` |
| `!=` | Not equal | `[ "$a" != "admin" ]` |
| `-z "$a"` | String is empty | `[ -z "$user" ]` |
| `-n "$a"` | String is NOT empty | `[ -n "$user" ]` |

```bash
#!/bin/bash

user_group=$1

if [ "$user_group" == "admin" ]; then
    echo "Welcome, Administrator."
elif [ "$user_group" = "devops" ]; then
    echo "Welcome, DevOps Engineer."
else
    echo "Access denied. Unknown group: $user_group"
fi
```

### `=` vs `==` — Which to Use?

```
=    →  POSIX standard. Works in sh, bash, zsh, dash — ALL shells.
==   →  Bash-specific. Only guaranteed to work in bash scripts.

Rule: Use = for portability.
      Use == only when your script will always run in bash.
```

---

## 6. User Input — Arguments & Prompts

### Method 1 — Command-Line Arguments

Pass values directly when running the script:

```bash
./setup.sh admin config.yaml
```

Inside the script:

```bash
#!/bin/bash

user_group=$1       # First argument:  admin
config_file=$2      # Second argument: config.yaml

echo "Group:  $user_group"
echo "Config: $config_file"
```

### Method 2 — Interactive Prompt with `read`

Ask the user to enter a value while the script is running:

```bash
#!/bin/bash

# -p displays the prompt message inline
read -p "Enter your username: " username
read -p "Enter your group: "    user_group

# -s flag makes input silent — hides what the user types (for passwords)
read -sp "Enter password: " user_pwd
echo ""   # newline after hidden input

echo "Hello $username, group: $user_group"
```

> Always use `-s` for password input so the characters are not echoed to the screen.

---

## 7. Positional Parameters

Positional parameters are the arguments passed to a script, accessed by their position number:

| Variable | Value |
|----------|-------|
| `$0` | The script name itself (`./setup.sh`) |
| `$1` | First argument |
| `$2` | Second argument |
| `$3` ... `$9` | Third through ninth arguments |
| `$*` | All arguments as a single string |
| `$@` | All arguments as separate items (better for loops) |
| `$#` | Count of total arguments passed |

```bash
#!/bin/bash

echo "Script name:      $0"
echo "First argument:   $1"
echo "Second argument:  $2"
echo "Total arguments:  $#"
echo "All arguments:    $*"
```

**Run:**
```bash
./setup.sh admin devops 8080
```

**Output:**
```
Script name:      ./setup.sh
First argument:   admin
Second argument:  devops
Total arguments:  3
All arguments:    admin devops 8080
```

---

## 8. Loops in Shell Scripts

### Types of Loops

| Loop | Use Case |
|------|---------|
| `for` | Iterate over a known list or range |
| `while` | Repeat while a condition is true |
| `until` | Repeat until a condition becomes true |
| `select` | Generate a numbered menu for user selection |

---

### `for` Loop

```bash
# Loop over a static list
for user in alice bob carol dave; do
    echo "Creating user: $user"
    sudo adduser $user
done

# Loop over all script arguments using $*
for param in $*; do
    echo "Parameter: $param"
done

# Loop over a number range
for i in {1..5}; do
    echo "Attempt $i"
done
```

---

### `while` Loop

Repeats as long as the condition is true:

```bash
#!/bin/bash

# Interactive score accumulator — keeps running until user types 'q'
sum=0

while true; do
    read -p "Enter a score (or 'q' to quit): " score

    if [ "$score" = "q" ]; then
        echo "Quitting the loop."
        break
    fi

    sum=$(( sum + score ))
    echo "Running total: $sum"
done

echo "Total score: $sum"
```

**Key syntax:**

| Keyword | Purpose |
|---------|---------|
| `while true` | Runs forever until explicitly stopped |
| `break` | Immediately exits the loop |
| `continue` | Skips current iteration, goes to next |
| `$(( expr ))` | Arithmetic evaluation |

---

### `until` Loop

The opposite of `while` — repeats until the condition becomes true:

```bash
#!/bin/bash

count=1

until [ $count -gt 5 ]; do
    echo "Count: $count"
    count=$(( count + 1 ))
done
```

---

### `select` Loop — Interactive Menu

Automatically generates a numbered menu from a list:

```bash
#!/bin/bash

echo "Select an action:"
select action in "Install Nginx" "Install MySQL" "Check Status" "Quit"; do
    case $action in
        "Install Nginx")
            sudo apt install -y nginx ;;
        "Install MySQL")
            sudo apt install -y mysql-server ;;
        "Check Status")
            systemctl status nginx ;;
        "Quit")
            echo "Exiting."
            break ;;
        *)
            echo "Invalid option. Try again." ;;
    esac
done
```

---

### Arithmetic in Shell Scripts

Use double parentheses `$(( ))` for all arithmetic:

```bash
a=10
b=3

echo $(( a + b ))   # 13 — addition
echo $(( a - b ))   # 7  — subtraction
echo $(( a * b ))   # 30 — multiplication
echo $(( a / b ))   # 3  — integer division
echo $(( a % b ))   # 1  — remainder (modulo)

# Increment a variable
count=$(( count + 1 ))
```

---

## 9. Functions & Return Values

### Defining and Calling Functions

```bash
#!/bin/bash

# Define a function
greet_user() {
    echo "Hello, $1! Welcome to the $2 team."
}

# Call the function with arguments
greet_user "Alice" "DevOps"
greet_user "Bob"   "Backend"
```

### Returning Values — Using `return` and `$?`

Shell functions can only `return` an integer (0–255). The special variable `$?` captures the return value of the last executed command:

```bash
#!/bin/bash

sum() {
    total=$(( $1 + $2 ))
    return $total
}

sum 1 32
result=$?
echo "Result: $result"   # 33
```

### Better Approach — Return via `echo`

For values larger than 255 or strings, use `echo` inside the function and capture with `$()`:

```bash
#!/bin/bash

sum() {
    echo $(( $1 + $2 ))
}

result=$(sum 100 250)
echo "Result: $result"   # 350
```

### Exit Codes

Every command exits with a numeric code:

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | General error |
| `2` | Misuse of shell command |
| `127` | Command not found |

```bash
# Check if the last command succeeded
if [ $? -eq 0 ]; then
    echo "Command succeeded."
else
    echo "Command failed."
fi
```

---

## 10. Double Brackets & Portability

Bash offers `[[ ]]` as an enhanced alternative to `[ ]`:

| Feature | `[ ]` | `[[ ]]` |
|---------|-------|---------|
| Works in all shells (portable) | Yes | No — Bash only |
| Pattern matching with `*` | No | Yes |
| Regex matching with `=~` | No | Yes |
| Safe without quoting variables | No | Yes |
| AND/OR with `&&` and `||` inside | No | Yes |

```bash
# Pattern matching
if [[ "$filename" == *.sh ]]; then
    echo "This is a shell script."
fi

# Regex matching
if [[ "$email" =~ ^[a-z]+@[a-z]+\.[a-z]+$ ]]; then
    echo "Valid email format."
fi
```

### Portability Rule

```
[ ]   →  Use for portable scripts compatible with all shells (sh, bash, dash)
[[ ]] →  Use when writing bash-only scripts needing pattern or regex matching

For complex scripting needs, consider Python or Ansible —
they offer cleaner syntax, better error handling, and far more power.
```

---

## 11. Complete Example Script

A script combining all concepts from this lesson:

```bash
#!/bin/bash
# =============================================================
# User Provisioning Script
# Usage: ./setup.sh <group> <username1> <username2> ...
# Example: ./setup.sh devops alice bob carol
# =============================================================
set -e

GROUP=$1

# --- Validate input ---
if [ $# -lt 2 ]; then
    echo "Usage: $0 <group> <username1> [username2] ..."
    exit 1
fi

# --- Functions ---
log() {
    echo "[$(date '+%H:%M:%S')] $1"
}

create_user() {
    local username=$1

    if id "$username" &>/dev/null; then
        log "User $username already exists. Skipping."
    else
        sudo adduser --disabled-password --gecos "" "$username"
        sudo usermod -aG "$GROUP" "$username"
        log "User $username created and added to group $GROUP."
    fi
}

# --- Check if group exists ---
if ! getent group "$GROUP" > /dev/null; then
    log "Group $GROUP not found. Creating it..."
    sudo groupadd "$GROUP"
fi

# --- Loop through all usernames (skip first arg which is the group) ---
shift
for username in $@; do
    create_user "$username"
done

log "Total users processed: $#"
log "All done!"
```

**Run:**
```bash
chmod u+x setup.sh
./setup.sh devops alice bob carol
```

---

## 12. Quick Reference Cheat Sheet

### Variables

```bash
name="alice"              # Declare (no spaces around =)
echo "$name"              # Use with $ sign
result=$(command)         # Capture command output
sum=$(( 5 + 3 ))          # Arithmetic with $(( ))
```

### Conditionals

```bash
if [ condition ]; then
    ...
elif [ condition ]; then
    ...
else
    ...
fi
```

### File Test Operators

| Op | Check |
|----|-------|
| `-f` | Is a regular file |
| `-d` | Is a directory |
| `-e` | Exists |
| `-x` | Is executable |
| `-z` | Is empty |
| `!` | Negate any condition |

### Number Comparison

| Op | Meaning |
|----|---------|
| `-eq` | Equal |
| `-ne` | Not equal |
| `-lt` | Less than |
| `-gt` | Greater than |
| `-ge` | Greater or equal |
| `-le` | Less or equal |

### Loops

```bash
for item in $@; do
    echo "$item"
done

while [ condition ]; do
    ...
done

until [ condition ]; do
    ...
done
```

### Functions

```bash
my_func() {
    echo $(( $1 + $2 ))    # return via echo
}
result=$(my_func 5 10)     # capture output
echo $?                    # get exit/return code
```

### Special Variables

| Variable | Meaning |
|----------|---------|
| `$0` | Script name |
| `$1`–`$9` | Positional arguments |
| `$#` | Argument count |
| `$*` | All arguments as string |
| `$@` | All arguments as list |
| `$?` | Last command exit code |

---

*End of Lesson 16*
