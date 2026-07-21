# Lesson 11: Vi & Vim — Terminal Text Editing

> **Vi** and **Vim** (Vi IMproved) are powerful terminal-based text editors available on virtually every Unix/Linux system. Unlike GUI editors, Vim operates entirely through the keyboard, making it an essential skill for server-side development, remote SSH sessions, and DevOps workflows where no graphical interface is available.

---

## Table of Contents

1. [Why Learn Vim?](#1-why-learn-vim)
2. [Opening & Creating Files](#2-opening--creating-files)
3. [Understanding Vim Modes](#3-understanding-vim-modes)
4. [Saving & Exiting](#4-saving--exiting)
5. [Editing: Insert Mode Essentials](#5-editing-insert-mode-essentials)
6. [Navigation: Command Mode](#6-navigation-command-mode)
7. [Deleting & Undoing](#7-deleting--undoing)
8. [Search & Replace](#8-search--replace)
9. [Quick Reference Cheat Sheet](#9-quick-reference-cheat-sheet)

---

## 1. Why Learn Vim?

In real-world enterprise and cloud environments, you will often need to edit configuration files, scripts, or source code directly on a remote server via SSH — with no GUI available. Vim is:

- **Pre-installed** on virtually all Linux/Unix distributions
- **Lightweight** — runs in any terminal with zero overhead
- **Powerful** — supports macros, plugins, syntax highlighting, and scripting
- **Industry-standard** — knowing Vim is a baseline expectation for DevOps, SRE, and backend engineering roles

---

## 2. Opening & Creating Files

```bash
# Open an existing file in Vim
vim main.java

# Open a new file (creates it on save)
vim newFileName.txt

# Open a file and jump directly to a specific line number
vim +12 main.java

# Open a file in read-only mode (safe for inspecting configs)
vim -R /etc/nginx/nginx.conf
```

> **Note:** If the file does not exist, Vim will create it when you save for the first time.

---

## 3. Understanding Vim Modes

Vim operates on a **modal editing** philosophy — different modes serve distinct purposes. This is the most important concept to grasp as a beginner.

```
┌─────────────────────────────────────────────────────┐
│                    VIM MODES                        │
│                                                     │
│   ┌─────────────────┐       ┌──────────────────┐    │
│   │  COMMAND MODE   │──i──► │   INSERT MODE    │    │
│   │   (Default)     │◄─ESC──│  (Type freely)   │    │
│   └────────┬────────┘       └──────────────────┘    │
│            │                                        │
│            │ :                                      │
│            ▼                                        │
│   ┌─────────────────┐                               │
│   │  EX / COLON     │  (Save, quit, replace)        │
│   │     MODE        │                               │
│   └─────────────────┘                               │
└─────────────────────────────────────────────────────┘
```

| Mode | How to Enter | What It Does |
|------|-------------|--------------|
| **Command Mode** | Default on open / press `ESC` | Navigate, delete, copy, undo |
| **Insert Mode** | Press `i` from Command Mode | Type and edit text freely |
| **Ex Mode** | Press `:` from Command Mode | Run file-level commands (save, quit, replace) |

> **Golden Rule:** When in doubt, press `ESC` to return to Command Mode. Vim always starts in Command Mode.

---

## 4. Saving & Exiting

All save/exit commands are entered from **Command Mode** by pressing `:` first.

```vim
:w          " Save (write) the file without exiting
:q          " Quit (only works if there are no unsaved changes)
:wq         " Save and quit
:x          " Save and quit (same as :wq, but only writes if changes exist)
:q!         " Force quit WITHOUT saving — discards all changes
:w!         " Force save (overrides read-only warnings)
```

**Step-by-step save workflow:**

```
1. Finish editing in Insert Mode
2. Press ESC  →  returns to Command Mode
3. Type :wq   →  saves and exits
```

> **⚠️ Warning:** `:q!` permanently discards all unsaved changes. There is no recycle bin or undo after exiting.

---

## 5. Editing: Insert Mode Essentials

Once in **Insert Mode** (press `i`), Vim behaves like a standard text editor. However, there are multiple ways to enter Insert Mode, each placing your cursor differently:

| Key | Enters Insert Mode... |
|-----|-----------------------|
| `i` | Before the current cursor position |
| `a` | After the current cursor position |
| `o` | On a new line below the current line |
| `O` | On a new line above the current line |
| `Shift + A` | At the **end** of the current line |
| `Shift + I` | At the **beginning** of the current line |

---

## 6. Navigation: Command Mode

All navigation below is performed in **Command Mode** (no `:` needed — just type the key directly).

### Cursor Movement

| Key | Action |
|-----|--------|
| `h` | Move left one character |
| `l` | Move right one character |
| `j` | Move down one line |
| `k` | Move up one line |
| `0` (zero) | Jump to the **beginning** of the current line |
| `$` | Jump to the **end** of the current line |
| `Shift + A` | Jump to end of line and enter Insert Mode |
| `gg` | Jump to the **first line** of the file |
| `G` | Jump to the **last line** of the file |
| `<number>G` | Jump to a **specific line number** (e.g., `12G` → line 12) |

### Example: Line Navigation

```
12G   →  Jumps to line 12
1G    →  Jumps to the very first line (same as gg)
```

---

## 7. Deleting & Undoing

All deletion commands below work in **Command Mode**.

### Deletion Commands

| Command | Action |
|---------|--------|
| `dd` | Delete the **entire current line** |
| `d<number>` | Delete the next N lines (e.g., `d5` deletes 5 lines) |
| `x` | Delete a single character under the cursor |
| `dw` | Delete from cursor to the end of the current word |
| `D` | Delete from cursor to the end of the line |

### Undo & Redo

| Command | Action |
|---------|--------|
| `u` | **Undo** the last change |
| `U` | Undo all changes on the current line |
| `Ctrl + r` | **Redo** (reverse an undo) |

> **Tip:** Vim supports multiple levels of undo. Keep pressing `u` to step back through your edit history.

---

## 8. Search & Replace

### Searching for a Pattern

```vim
/pattern        " Search FORWARD for 'pattern'
?pattern        " Search BACKWARD for 'pattern'
```

Once a search is active:

| Key | Action |
|-----|--------|
| `n` | Jump to the **next** match (forward) |
| `N` | Jump to the **previous** match (backward) |

**Example:**

```vim
/NullPointerException    " Highlights all matches in the file
n                        " Jumps to the next occurrence
N                        " Jumps back to the previous occurrence
```

### Search & Replace (Global Substitution)

```vim
:%s/old/new           " Replace FIRST occurrence on each line
:%s/old/new/g         " Replace ALL occurrences in the entire file
:%s/old/new/gc        " Replace ALL, but ASK for confirmation each time
:5,20s/old/new/g      " Replace ALL occurrences between lines 5 and 20
```

**Breakdown of `:%s/old/new/g`:**

```
:       →  Enter Ex Mode
%       →  Apply to the entire file (% = all lines)
s       →  Substitute command
/old    →  Pattern to find
/new    →  Replacement string
/g      →  Global flag (replace ALL occurrences per line, not just the first)
```

> **Practical Example:** Rename a variable across an entire file:
> ```vim
> :%s/userName/userId/g
> ```

---

## 9. Quick Reference Cheat Sheet

### File Operations

| Command | Action |
|---------|--------|
| `vim <file>` | Open a file |
| `vim <newFile>` | Create a new file |
| `:w` | Save |
| `:wq` | Save and quit |
| `:q!` | Quit without saving |

### Mode Switching

| Key | Action |
|-----|--------|
| `i` | Enter Insert Mode (before cursor) |
| `Shift + A` | Enter Insert Mode at end of line |
| `o` | New line below and enter Insert Mode |
| `ESC` | Return to Command Mode |

### Navigation

| Key | Action |
|-----|--------|
| `0` | Beginning of line |
| `$` | End of line |
| `<n>G` | Go to line n |
| `gg` | First line |
| `G` | Last line |

### Editing

| Key | Action |
|-----|--------|
| `dd` | Delete current line |
| `d<n>` | Delete n lines |
| `x` | Delete character |
| `u` | Undo |
| `Ctrl + r` | Redo |

### Search & Replace

| Command | Action |
|---------|--------|
| `/pattern` | Search forward |
| `n` / `N` | Next / previous match |
| `:%s/old/new/g` | Replace all occurrences |

---

## Vim vs. Nano — When to Use Which?

| Criteria | Vim | Nano |
|----------|-----|------|
| **Learning Curve** | Steep — modal editing takes practice | Gentle — behaves like a standard editor |
| **Power & Flexibility** | Extremely powerful; scriptable and extensible | Basic editing only |
| **Speed (once learned)** | Much faster — keyboard-only workflows | Slower for large files |
| **Availability** | Available on all Unix/Linux systems | Usually available but not always pre-installed |
| **Best For** | Production DevOps, large codebases, frequent edits | Quick one-off config edits for beginners |

> **Recommendation:** Invest time in learning Vim. The initial learning curve pays off significantly in long-term productivity during remote server management and CI/CD pipeline work.

---

*End of Lesson 11*
