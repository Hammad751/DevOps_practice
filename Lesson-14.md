# Lesson 14: Pipes & Redirects in Linux

> Two of Linux's most powerful command-line features are **Pipes** and **Redirects**. Together, they allow you to chain commands, filter output, and control exactly where data flows — turning simple tools into powerful, composable workflows without writing a single script.

---

## Table of Contents

1. [The Linux Philosophy: Do One Thing Well](#1-the-linux-philosophy-do-one-thing-well)
2. [Pipes — Connecting Commands](#2-pipes--connecting-commands)
3. [Grep — Filtering Output](#3-grep--filtering-output)
4. [Chaining Multiple Pipes](#4-chaining-multiple-pipes)
5. [Redirects — Writing Output to Files](#5-redirects--writing-output-to-files)
6. [Standard Streams (stdin, stdout, stderr)](#6-standard-streams-stdin-stdout-stderr)
7. [Redirect Operators Reference](#7-redirect-operators-reference)
8. [Practical Real-World Examples](#8-practical-real-world-examples)
9. [Essential Commands Reference](#9-essential-commands-reference)
10. [Quick Reference Cheat Sheet](#10-quick-reference-cheat-sheet)

---

## 1. The Linux Philosophy: Do One Thing Well

Linux commands are designed to be **small, focused, and composable**. Each command does one thing well:

- `cat` → reads files
- `grep` → filters text
- `sort` → sorts lines
- `uniq` → removes duplicates
- `less` → paginates output
- `wc` → counts lines/words/characters

**Pipes and Redirects** are the glue that connects these tools into powerful pipelines:

```
┌─────────┐      ┌─────────┐      ┌─────────┐      ┌──────────┐
│  cat    │─────►│  sort   │─────►│  uniq   │─────►│  Output  │
│ file.txt│  |   │   -r    │  |   │         │      │(terminal │
│         │      │         │      │         │      │ or file) │
└─────────┘      └─────────┘      └─────────┘      └──────────┘
     Command 1         Command 2        Command 3
```

---

## 2. Pipes — Connecting Commands

### What is a Pipe?

A **pipe** (`|`) is a temporary connection between two commands. It takes the **output (stdout)** of the command on the left and feeds it as **input (stdin)** to the command on the right — all in a single line, in memory, with no intermediate files.

```
command1 | command2 | command3
    │            │           │
    │   stdout───►stdin      │
    │            │  stdout───►stdin
    │            │           │
   runs        runs         runs
   first       second       third
```

### Basic Pipe Example

```bash
# Read a large log file and paginate it (scroll instead of flood the terminal)
cat /var/log/syslog | less
```

- `cat /var/log/syslog` → outputs the entire syslog file to stdout
- `| less` → receives that output and displays it one page at a time
- Press `Space` to scroll down, `q` to quit `less`

### Pipes vs. Redirects at a Glance

| Feature | Pipe `\|` | Redirect `>` / `>>` |
|---------|-----------|---------------------|
| Connects | Command → Command | Command → File |
| Output goes to | Next command's input | A file on disk |
| Intermediate file? | No — in memory | Yes — writes to disk |
| Direction | Left to right | Command to file |

---

## 3. Grep — Filtering Output

### What is `grep`?

`grep` stands for **"Globally search for a Regular Expression and Print"**. It scans input line by line and returns only the lines that match a given pattern. It is the most commonly used filter in Linux pipelines.

```bash
# Basic syntax
grep "pattern" <file>

# Used in a pipe
command | grep "pattern"
```

### Searching Command History

```bash
# Find all commands in your history that contain "sudo"
history | grep sudo

# Search for a multi-word string (must use quotes)
history | grep "sudo chmod"
```

> **Single word** → quotes optional: `history | grep sudo`
> **Multiple words / spaces** → quotes required: `history | grep "sudo chmod"`

### Common `grep` Flags

```bash
# Case-insensitive search
grep -i "error" /var/log/syslog

# Show line numbers alongside matches
grep -n "failed" /var/log/auth.log

# Invert match — show lines that do NOT contain the pattern
grep -v "debug" /var/log/syslog

# Count the number of matching lines
grep -c "error" /var/log/syslog

# Search recursively inside a directory
grep -r "TODO" /home/alice/project/

# Show N lines of context before and after each match
grep -A 2 -B 2 "ERROR" app.log
```

---

## 4. Chaining Multiple Pipes

You can chain as many pipes as needed. Each command processes the output of the previous one:

```bash
# View history entries containing "sudo chmod", paginated
history | grep "sudo chmod" | less
```

**Flow:**
```
history          →  all shell history
  | grep "sudo chmod"  →  filter to matching lines only
    | less               →  display one page at a time
```

### Multi-Pipe Example: Sort & Deduplicate

```bash
cat words.txt | sort -r | uniq
```

**Step-by-step breakdown:**

```
┌─────────────────┐
│  cat words.txt  │  → Outputs all lines of the file to stdout
└────────┬────────┘
         │ pipe
         ▼
┌─────────────────┐
│   sort -r       │  → Sorts all lines in REVERSE alphabetical order
└────────┬────────┘
         │ pipe
         ▼
┌─────────────────┐
│     uniq        │  → Removes consecutive duplicate lines
└────────┬────────┘
         │
         ▼
    Terminal output
    (sorted, deduplicated)
```

> **Note:** `uniq` only removes **consecutive** duplicates. Always `sort` first before piping to `uniq`, so that duplicates are adjacent and can be caught.

### More Multi-Pipe Examples

```bash
# Count how many times "error" appears in a log file
cat /var/log/syslog | grep -i "error" | wc -l

# Find all running processes related to nginx
ps aux | grep nginx

# List the 10 most recently modified files
ls -lt | head -10

# Show unique IP addresses from an access log
cat access.log | grep "GET" | awk '{print $1}' | sort | uniq
```

---

## 5. Redirects — Writing Output to Files

### What is Redirection?

**Redirection** reroutes the output of a command to a **file** instead of the terminal. Unlike pipes (which connect command to command), redirects connect a **command to a file**.

### Overwrite Redirect `>`

```bash
# Redirect output to a file — CREATES or OVERWRITES the file
command > filename.txt
```

```bash
# Save command history to a file (overwrites existing content)
history > history_backup.txt

# Save directory listing to a file
ls -la > file_list.txt

# Save grep results to a file
grep "error" /var/log/syslog > errors.txt
```

> **⚠️ Warning:** The `>` operator **overwrites** the entire file if it already exists. The previous content is permanently lost.

### Append Redirect `>>`

```bash
# Redirect output and APPEND to the end of the file
command >> filename.txt
```

```bash
# Append new rm commands to an existing log file (does not erase old content)
history | grep rm >> rm_commands.txt

# Keep adding error logs across multiple runs
grep "error" /var/log/syslog >> error_log.txt
```

> **`>` vs `>>`:**
> - `>` → overwrites from the top (destructive)
> - `>>` → appends to the bottom (safe, non-destructive)

### Input Redirect `<`

```bash
# Feed a file's contents as input to a command
command < filename.txt
```

```bash
# Sort the contents of a file
sort < words.txt

# Feed a list of emails into a mail command
mail -s "Report" admin@example.com < report.txt
```

---

## 6. Standard Streams (stdin, stdout, stderr)

Every Linux process has three default data streams:

```
┌─────────────────────────────────────────────────┐
│                  LINUX PROCESS                  │
│                                                 │
│  stdin  (0) ──►  [  command  ]  ──►  stdout (1) │
│                       │                         │
│                       └──────────►  stderr (2)  │
│                                                 │
└─────────────────────────────────────────────────┘
```

| Stream | Number | Default Destination | Description |
|--------|--------|-------------------|-------------|
| `stdin` | `0` | Keyboard | Input fed to the command |
| `stdout` | `1` | Terminal screen | Normal output |
| `stderr` | `2` | Terminal screen | Error messages |

### Redirecting stderr

```bash
# Redirect errors to a separate file
command 2> errors.txt

# Redirect both stdout and stderr to the same file
command > output.txt 2>&1

# Discard errors entirely (send to /dev/null — the "black hole")
command 2> /dev/null

# Discard ALL output (stdout + stderr)
command > /dev/null 2>&1
```

> **`/dev/null`** is a special system file that discards everything written to it. Use it to silence noisy commands in scripts.

### Practical Example

```bash
# Run a find command but suppress "Permission denied" errors
find / -name "config.yaml" 2>/dev/null
```

---

## 7. Redirect Operators Reference

| Operator | Name | Action |
|----------|------|--------|
| `\|` | Pipe | Connect command output → next command input |
| `>` | Overwrite redirect | Send stdout → file (overwrites) |
| `>>` | Append redirect | Send stdout → file (appends) |
| `<` | Input redirect | Feed file → command as stdin |
| `2>` | stderr redirect | Send error output → file |
| `2>>` | Append stderr | Append error output → file |
| `2>&1` | Merge streams | Redirect stderr into stdout stream |
| `&>` | Both streams | Redirect stdout AND stderr → file |
| `/dev/null` | Null device | Discard output entirely |

---

## 8. Practical Real-World Examples

### DevOps & System Administration

```bash
# Monitor live logs and filter for errors only
tail -f /var/log/syslog | grep "ERROR"

# Count total number of lines in a log file
wc -l < /var/log/syslog

# Save all active network connections to a file
netstat -tuln > open_ports.txt

# Find all files larger than 100MB and log them
find / -size +100M 2>/dev/null > large_files.txt

# Save process list snapshot
ps aux > process_snapshot.txt
```

### Text Processing

```bash
# Sort a file and remove duplicates, save result
sort words.txt | uniq > cleaned_words.txt

# Count unique words in a file
cat essay.txt | tr ' ' '\n' | sort | uniq | wc -l

# Extract specific columns from a CSV
cat data.csv | cut -d',' -f1,3 > extracted.csv

# Find and count all TODO comments in a codebase
grep -r "TODO" ./src | wc -l
```

### Combining Pipes & Redirects

```bash
# Find all sudo commands used, deduplicate, and save to a file
history | grep sudo | sort | uniq > sudo_history.txt

# Filter errors from log, sort them, and append to a report
grep "ERROR" /var/log/app.log | sort | uniq >> weekly_report.txt
```

---

## 9. Essential Commands Reference

These are the most commonly used commands inside pipes and redirect workflows:

| Command | Purpose | Common Usage |
|---------|---------|-------------|
| `cat` | Output file contents | `cat file.txt \| ...` |
| `less` | Paginate output | `... \| less` |
| `grep` | Filter by pattern | `... \| grep "pattern"` |
| `sort` | Sort lines | `... \| sort` / `sort -r` (reverse) |
| `uniq` | Remove duplicate lines | `... \| uniq` (sort first!) |
| `wc -l` | Count lines | `... \| wc -l` |
| `head` | Show first N lines | `... \| head -10` |
| `tail` | Show last N lines | `... \| tail -20` |
| `tail -f` | Follow a live file | `tail -f /var/log/syslog` |
| `cut` | Extract columns | `... \| cut -d',' -f1` |
| `tr` | Translate/replace chars | `... \| tr 'a-z' 'A-Z'` |
| `awk` | Field-based text processing | `... \| awk '{print $2}'` |
| `tee` | Write to file AND terminal | `... \| tee output.txt` |

### The `tee` Command — Output to Both File and Screen

```bash
# See output in terminal AND save to file simultaneously
cat /var/log/syslog | grep "error" | tee errors.txt

# Append version
cat /var/log/syslog | grep "error" | tee -a errors.txt
```

---

## 10. Quick Reference Cheat Sheet

### Pipe

```bash
command1 | command2          # Output of 1 becomes input of 2
command1 | command2 | command3   # Chain as many as needed
```

### Redirect

```bash
command > file.txt           # Overwrite file with output
command >> file.txt          # Append output to file
command < file.txt           # Use file as input
command 2> errors.txt        # Redirect errors to file
command > out.txt 2>&1       # Redirect both stdout and stderr
command > /dev/null 2>&1     # Discard all output
```

### Grep Flags

```bash
grep "pattern" file          # Basic search
grep -i "pattern" file       # Case-insensitive
grep -n "pattern" file       # Show line numbers
grep -v "pattern" file       # Invert (exclude matches)
grep -c "pattern" file       # Count matches
grep -r "pattern" dir/       # Recursive search
```

### Common Pipeline Patterns

```bash
cat file | sort | uniq               # Sort and deduplicate
command | grep "x" | wc -l          # Count matching lines
command | grep "x" | less           # Filter and paginate
command | tee file.txt              # Output to screen AND file
find / -name "x" 2>/dev/null        # Suppress permission errors
```

---

*End of Lesson 14*
