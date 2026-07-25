# Lesson 16: Shell Scripting II — Variables, Conditions, Loops & Functions

> This lesson expands on the shell scripting fundamentals from Lesson 15, diving deeper into variables, conditional logic with file/number/string operators, positional parameters, loops, and functions with return values.

---

## Table of Contents

1. [Variables in Shell Scripts](#1-variables-in-shell-scripts)
2. [Conditional Statements](#2-conditional-statements)
3. [File Test Operators](#3-file-test-operators)
4. [Number Comparison Operators](#4-number-comparison-operators)
5. [String Operators](#5-string-operators)
6. [User Input — Arguments & Prompts](#6-user-input--arguments--prompts)
7. [Positional Parameters](#7-positional-parameters)
8. [Loops](#8-loops)
9. [Functions & Return Values](#9-functions--return-values)
10. [Putting It All Together — Real Example](#10-putting-it-all-together--real-example)
11. [Quick Reference Cheat Sheet](#11-quick-reference-cheat-sheet)

---

## 1. Variables in Shell Scripts

### Declaring & Using Variables

```bash
#!/bin/bash

# Declare a variable — NO spaces around the = sign
file_name="config.yaml"
user_name="hammad"

# Use a variable with the $ prefix
echo "Config file is: $file_name"
echo "User is: $user_name"
```

> **Rule:** There must be **no spaces** around the `=` sign when assigning variables.
> - ✅ `file_name="config.yaml"`
> - ❌ `file_name = "config.yaml"` → Shell interprets this as a command, not assignment

### Capturing Command Output into a Variable

Use `$( )` to run a command and store its output directly into a variable:

```bash
# Store the output of 'ls config' into a variable
config_file=$(ls config)

# Store current date
today=$(date)

# Store current logged-in user
current_user=$(whoami)

echo "Files in config: $config_file"
echo "Today is: $today"
echo "Running as: $current_user"
```

---

## 2. Conditional Statements

### Basic `if / else / fi` Syntax

```bash
if [ condition ]; then
    # commands if condition is TRUE
else
    # commands if condition is FALSE
fi
```

> **`fi`** closes every `if` block — it is `if` spelled backwards.

### Real Example — Check if Directory Exists

```bash
#!/bin/bash

if [ -d "config" ]; then
    echo "Config directory found. Reading files..."
    config_file=$(ls config)
    echo "Contents: $config_file"
else
    echo "Directory not found. Creating it now..."
    mkdir config
    echo "New config directory created."
fi
```

### `if / elif / else` — Multiple Conditions

```bash
#!/bin/bash

score=85

if [ $score -ge 90 ]; then
    echo "Grade: A"
elif [ $score -ge 75 ]; then
    echo "Grade: B"
elif [ $score -ge 60 ]; then
    echo "Grade: C"
else
    echo "Grade: F"
fi
```

---

## 3. File Test Operators

File test operators check the **properties of a file or directory** inside an `if` condition.

| Operator | Meaning |
|----------|---------|
| `-d` | Is a **directory** |
| `-f` | Is a regular **file** |
| `-e` | **Exists** (file or directory) |
| `-r` | Is **readable** |
| `-w` | Is **writable** |
| `-x` | Is **executable** |
| `-s` | File exists and is **not empty** |
| `-z` | File exists and is **empty** |
| `!` | **Negation** — reverses the condition |

```bash
#!/bin/bash

# Check if a file exists
if [ -f "setup.sh" ]; then
    echo "setup.sh exists."
fi

# Check if a path is a directory
if [ -d "/etc/nginx" ]; then
    echo "Nginx config directory exists."
fi

# Check if a file is executable
if [ -x "deploy.sh" ]; then
    echo "deploy.sh is executable."
fi

# Negation — check if something does NOT exist
if [ ! -d "logs" ]; then
    echo "Logs directory missing. Creating..."
    mkdir logs
fi
```

> 📖 **Full list of file/test operators:** https://www.tutorialspoint.com/unix/unix-file-operators.htm

---

## 4. Number Comparison Operators

In shell scripts, **keywords** are used instead of symbols like `>` or `==` for comparing numbers. This avoids conflicts with redirect operators.

| Operator | Meaning | Example |
|----------|---------|---------|
| `-eq` | Equal to | `[ $a -eq 10 ]` |
| `-ne` | Not equal to | `[ $a -ne 10 ]` |
| `-lt` | Less than | `[ $a -lt 10 ]` |
| `-le` | Less than or equal to | `[ $a -le 10 ]` |
| `-gt` | Greater than | `[ $a -gt 10 ]` |
| `-ge` | Greater than or equal to | `[ $a -ge 10 ]` |

```bash
#!/bin/bash

num=10

if [ $num -eq 10 ]; then
    echo "Number is exactly 10."
fi

if [ $num -gt 5 ]; then
    echo "Number is greater than 5."
fi

if [ $num -ne 0 ]; then
    echo "Number is not zero."
fi
```

---

## 5. String Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` | Equal (POSIX — works in all shells) | `[ "$a" = "hello" ]` |
| `==` | Equal (Bash-specific) | `[ "$a" == "hello" ]` |
| `!=` | Not equal | `[ "$a" != "hello" ]` |
| `-z` | String is **empty** | `[ -z "$a" ]` |
| `-n` | String is **not empty** | `[ -n "$a" ]` |

```bash
#!/bin/bash

user_group="admin"

# Bash-specific (==)
if [ $user_group == "admin" ]; then
    echo "Welcome, admin!"
fi

# POSIX-compatible (=) — works in sh, bash, zsh, dash
if [ $user_group = "admin" ]; then
    echo "Access granted."
fi

# Check if a string is empty
name=""
if [ -z "$name" ]; then
    echo "Name was not provided."
fi
```

> **`=` vs `==`:**
> - `==` is **Bash-specific** — only works in Bash scripts
> - `=` is **POSIX standard** — cross-compatible with all shell programs (`sh`, `dash`, `zsh`)
> - If your script uses `#!/bin/bash`, either works. If you use `#!/bin/sh`, always use `=`

### Double Brackets `[[ ]]` — Bash Extended Syntax

```bash
# Double brackets allow more flexible comparisons in Bash
if [[ $user_group == "admin" || $user_group == "sudo" ]]; then
    echo "Privileged user."
fi
```

> **`[ ]` vs `[[ ]]`:**
> - `[ ]` → POSIX standard, portable across all shells
> - `[[ ]]` → Bash-only, supports `&&`, `||`, regex matching, and no quoting issues
> - Use `[[ ]]` for simpler, more readable Bash scripts; use `[ ]` when portability matters

---

## 6. User Input — Arguments & Prompts

### Method 1 — Command-Line Arguments

Pass values directly when running the script:

```bash
./setup.sh admin /etc/config
```

Inside the script, access them with positional parameters:

```bash
#!/bin/bash

user_group=$1      # First argument:  admin
config_path=$2     # Second argument: /etc/config

echo "Setting up user group: $user_group"
echo "Using config at: $config_path"
```

### Method 2 — Interactive Prompt with `read`

Prompt the user for input while the script is running:

```bash
#!/bin/bash

# Basic read
read -p "Enter username: " username
echo "Hello, $username!"

# Silent read — hides input (ideal for passwords)
read -sp "Enter password: " user_pwd
echo ""    # Print newline after hidden input
echo "Password stored securely."

# Read with a timeout (waits 10 seconds, then continues)
read -t 10 -p "Enter choice (you have 10s): " choice
```

| Flag | Meaning |
|------|---------|
| `-p` | Display a prompt message |
| `-s` | Silent mode — hides typed input (for passwords) |
| `-t` | Timeout in seconds |
| `-n` | Read only N characters |

---

## 7. Positional Parameters

Positional parameters let scripts accept and process multiple inputs passed as arguments.

| Parameter | Value |
|-----------|-------|
| `$0` | The script name itself |
| `$1` – `$9` | Individual arguments (1st through 9th) |
| `$*` | All arguments as a single string |
| `$@` | All arguments as a list (better for looping) |
| `$#` | Total **count** of arguments passed |
| `$?` | Exit code of the **last executed command** (`0` = success) |

```bash
#!/bin/bash

echo "Script name:        $0"
echo "First argument:     $1"
echo "Second argument:    $2"
echo "All arguments:      $*"
echo "Number of args:     $#"
```

**Run it:**
```bash
./setup.sh admin devops 8080
# Output:
# Script name:        ./setup.sh
# First argument:     admin
# Second argument:    devops
# All arguments:      admin devops 8080
# Number of args:     3
```

---

## 8. Loops

Shell scripting supports four types of loops:

### `for` Loop

Best for iterating over a known list or all arguments:

```bash
#!/bin/bash

# Loop over a fixed list
for package in git curl wget vim; do
    echo "Installing $package..."
    sudo apt install -y $package
done

# Loop over all script arguments using $*
for param in $*; do
    echo "Processing parameter: $param"
done
```

### `while` Loop

Runs as long as a condition is **true**:

```bash
#!/bin/bash

# Score accumulator — exits when user types 'q'
sum=0

while true; do
    read -p "Enter a score (or 'q' to quit): " score

    if [ "$score" == "q" ]; then
        echo "Quitting the loop."
        break
    fi

    sum=$(( sum + score ))
done

echo "Total score: $sum"
```

> **Arithmetic in shell scripts** uses double parentheses with `$`:
> ```bash
> result=$(( 5 + 3 ))     # result = 8
> sum=$(( sum + score ))  # increment sum
> ```

### `until` Loop

The opposite of `while` — runs as long as a condition is **false**:

```bash
#!/bin/bash

count=1

until [ $count -gt 5 ]; do
    echo "Count: $count"
    count=$(( count + 1 ))
done
```

### `select` Loop

Generates an interactive numbered menu for the user:

```bash
#!/bin/bash

echo "Choose an option:"
select option in "Install Nginx" "Install MySQL" "Quit"; do
    case $option in
        "Install Nginx")
            sudo apt install -y nginx
            ;;
        "Install MySQL")
            sudo apt install -y mysql-server
            ;;
        "Quit")
            echo "Exiting."
            break
            ;;
        *)
            echo "Invalid option."
            ;;
    esac
done
```

### Loop Control

| Command | Action |
|---------|--------|
| `break` | Exit the loop immediately |
| `continue` | Skip the current iteration and move to the next |

---

## 9. Functions & Return Values

### Defining and Calling Functions

```bash
#!/bin/bash

# Define a function
greet_user() {
    echo "Hello, $1! Welcome to the system."
}

# Call the function
greet_user "Hammad"
greet_user "Alice"
```

### Returning Values from Functions

Shell functions can only `return` an **integer exit code (0–255)**. To capture this value, use the special variable `$?` immediately after calling the function.

```bash
#!/bin/bash

function sum() {
    total=$(( $1 + $2 ))
    return $total
}

# Call the function with arguments
sum 1 32

# $? captures the return value of the last executed command
result=$?
echo "Result: $result"
```

> **Important limitation:** `return` in shell functions only supports integers `0–255`. For larger numbers or strings, use `echo` instead and capture with `$( )`:

```bash
#!/bin/bash

# Better pattern — use echo to "return" any value
calculate_sum() {
    echo $(( $1 + $2 ))
}

# Capture the echoed output
result=$(calculate_sum 50 200)
echo "Sum is: $result"    # Sum is: 250
```

| Method | Limitation | Best For |
|--------|-----------|---------|
| `return $value` + `$?` | Integer 0–255 only | Exit codes, status flags |
| `echo $value` + `$(func)` | Any value (strings, large numbers) | Actual computed results |

---

## 10. Putting It All Together — Real Example

A complete script that uses variables, conditions, arguments, loops, and functions:

```bash
#!/bin/bash
# =============================================================
# User Setup Script
# Usage: ./user_setup.sh <username> <group>
# =============================================================

set -e    # Exit immediately on error

# --- Validate Arguments ---
if [ $# -ne 2 ]; then
    echo "Usage: $0 <username> <group>"
    exit 1
fi

USERNAME=$1
GROUP=$2

# --- Functions ---
create_user() {
    if id "$1" &>/dev/null; then
        echo "User '$1' already exists. Skipping."
    else
        sudo adduser --disabled-password --gecos "" $1
        echo "User '$1' created."
    fi
}

assign_group() {
    sudo usermod -aG $2 $1
    echo "User '$1' added to group '$2'."
}

log_result() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> setup.log
}

# --- Main ---
echo "Starting user setup..."

create_user $USERNAME
assign_group $USERNAME $GROUP

# Loop through any extra groups passed as additional args
for extra_group in "${@:3}"; do
    assign_group $USERNAME $extra_group
done

log_result "Setup complete for user: $USERNAME"
echo "Done! Check setup.log for details."
```

**Run it:**
```bash
chmod u+x user_setup.sh
./user_setup.sh hammad devops sudo docker
```

---

## 11. Quick Reference Cheat Sheet

### Variables

```bash
name="value"              # Assign variable
echo $name                # Use variable
result=$(command)         # Capture command output
result=$(( 5 + 3 ))       # Arithmetic expression
```

### Conditionals

```bash
if [ condition ]; then    # Open
    # ...
elif [ condition ]; then  # Optional
    # ...
else                      # Optional
    # ...
fi                        # Close
```

### Test Operators

| Type | Operators |
|------|-----------|
| Files | `-f` `-d` `-e` `-r` `-w` `-x` `-s` |
| Numbers | `-eq` `-ne` `-lt` `-le` `-gt` `-ge` |
| Strings | `=` `==` `!=` `-z` `-n` |

### Loops

```bash
for item in $*; do        # For loop over all args
    echo $item
done

while [ condition ]; do   # While loop
    # ...
done

until [ condition ]; do   # Until loop (opposite of while)
    # ...
done
```

### Positional Parameters

| Variable | Meaning |
|----------|---------|
| `$1`–`$9` | Individual arguments |
| `$*` | All arguments |
| `$#` | Argument count |
| `$?` | Last command exit code |
| `$0` | Script name |

### Functions

```bash
my_function() {
    echo $(( $1 + $2 ))   # Return via echo (any value)
}
result=$(my_function 10 20)

# Or return integer exit code (0-255 only)
my_function() { return 42; }
my_function
echo $?
```

---

*End of Lesson 16*
