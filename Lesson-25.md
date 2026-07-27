# Lesson 25: WSL2 Migration & Solana/Anchor Development Environment Setup

> This lesson documents the complete journey of establishing a **Solana smart contract development environment** using WSL2 on Windows. It covers every issue encountered — GLIBC compatibility failures, SSL errors, toolchain mismatches, and Node.js version conflicts — along with the reasoning behind each fix and the final working architecture. Use this as both a learning reference and a rebuild guide.

---

## Table of Contents

1. [Environment Overview](#1-environment-overview)
2. [The Problem — Why Ubuntu-20.04 Broke Everything](#2-the-problem--why-ubuntu-2004-broke-everything)
3. [Phase-by-Phase Troubleshooting Journey](#3-phase-by-phase-troubleshooting-journey)
4. [Detailed Issues & Root Causes](#4-detailed-issues--root-causes)
5. [WSL2 Migration — Ubuntu-20.04 to Ubuntu-24.04](#5-wsl2-migration--ubuntu-2004-to-ubuntu-2404)
6. [Installing the Full Toolchain from Scratch](#6-installing-the-full-toolchain-from-scratch)
7. [Visual Studio Code Integration](#7-visual-studio-code-integration)
8. [Target Environment Specification](#8-target-environment-specification)
9. [Validation Checklist](#9-validation-checklist)
10. [Lessons Learned](#10-lessons-learned)
11. [Quick Reference Cheat Sheet](#11-quick-reference-cheat-sheet)

---

## 1. Environment Overview

### What We Are Building

A full **Solana smart contract development environment** running on WSL2 (Windows Subsystem for Linux 2), using:

```
┌──────────────────────────────────────────────────────────────┐
│                     WINDOWS HOST                             │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐   │
│  │              WSL2 — Ubuntu-24.04                      │   │
│  │                                                       │   │
│  │  Node.js v22 (nvm)                                    │   │
│  │  Rust + Cargo (rustup)                                │   │
│  │  Solana CLI (Mucho / official installer)              │   │
│  │  Anchor CLI (AVM — Anchor Version Manager)            │   │
│  │  pnpm (package manager)                               │   │
│  │                                                       │   │
│  │  Project files: /home/user/projects/                  │   │
│  └───────────────────────────────────────────────────────┘   │
│                                                              │
│  VS Code + Remote-WSL extension (Windows-side editor)        │
└──────────────────────────────────────────────────────────────┘
```

### Key Tools Explained

| Tool | Purpose |
|------|---------|
| **WSL2** | Runs a real Linux kernel on Windows — required for Solana/Rust tooling |
| **Ubuntu-24.04** | The Linux distro — chosen for GLIBC 2.38+ compatibility |
| **Rust + Cargo** | Solana programs are written in Rust; Cargo is its build system |
| **Solana CLI** | Deploy, test, and interact with the Solana blockchain |
| **Anchor** | Framework that simplifies writing Solana smart contracts |
| **AVM** | Anchor Version Manager — switch between Anchor versions easily |
| **nvm** | Node Version Manager — manage multiple Node.js versions |
| **pnpm** | Fast, disk-efficient alternative to npm |
| **Mucho** | Simplified all-in-one installer for the Solana ecosystem |

---

## 2. The Problem — Why Ubuntu-20.04 Broke Everything

Ubuntu-20.04 ships with **GLIBC 2.31**. Modern Solana and Anchor tooling requires **GLIBC 2.38+**. This single version mismatch caused a cascade of failures across every layer of the stack.

```
Ubuntu-20.04 ships GLIBC 2.31
         │
         ├──► Anchor 0.31.1 needs GLIBC 2.38 → FAILS
         │
         ├──► Modern OpenSSL behaves differently → TLS errors
         │
         ├──► Rust toolchain mismatch → build-bpf deprecated
         │
         └──► Node.js 18 too old for npm 11 → version conflicts

The root cause of ALL problems was the outdated base OS.
```

### GLIBC Version by Ubuntu Release

| Ubuntu Version | GLIBC Version | Status for Solana Dev |
|----------------|--------------|----------------------|
| Ubuntu 20.04 LTS | 2.31 | Too old — causes failures |
| Ubuntu 22.04 LTS | 2.35 | Partial — some tools work |
| Ubuntu 24.04 LTS | 2.39 | Recommended — fully compatible |

> **Key Lesson:** Always start with the **latest LTS Ubuntu** when setting up a blockchain development environment. The tooling moves fast and consistently targets newer system libraries.

---

## 3. Phase-by-Phase Troubleshooting Journey

### Phase 1 — Initial WSL2 Usage

Started in an existing **Ubuntu-20.04** WSL2 environment. Performed basic Linux administration — deleting directories, understanding file permissions.

**Issue encountered:**

```bash
rm -r sourcify
# Error: rm: remove write-protected regular file '.git/objects/pack/...'?
```

Git object pack files are marked **write-protected** to prevent accidental corruption. The standard `rm -r` prompts for each protected file.

**Fix:**

```bash
rm -rf sourcify
```

The `-f` flag forces deletion without prompting. Always confirm you have the right target directory before using `-rf`.

---

### Phase 2 — Anchor CLI Failure (GLIBC Error)

After installing Anchor CLI 0.31.1:

```bash
anchor init hello-world
# Error:
# anchor: /lib/x86_64-linux-gnu/libc.so.6: version 'GLIBC_2.38' not found
# anchor: /lib/x86_64-linux-gnu/libc.so.6: version 'GLIBC_2.39' not found
```

**Root Cause:** Anchor 0.31.1 was compiled against GLIBC 2.38/2.39. Ubuntu-20.04 only provides GLIBC 2.31.

**Temporary Fix — Downgrade Anchor:**

```bash
# Install Anchor Version Manager
cargo install --git https://github.com/coral-xyz/anchor avm --locked --force

# Install an older compatible version
avm install 0.27.0
avm use 0.27.0

# Verify
anchor --version
```

---

### Phase 3 — AVM Version Switching Problem

After switching to Anchor 0.27.0, the shell kept launching 0.31.1.

**Root Cause:** PATH was still pointing to the old binary:

```bash
which anchor
# /home/user/.avm/bin/anchor-0.31.1   ← wrong binary still active

echo $PATH
# ~/.avm/bin was present but the symlink still pointed to 0.31.1
```

**Fix:**

```bash
# Force AVM to relink
avm use 0.27.0

# Verify the active version
anchor --version
# anchor-cli 0.27.0

# If still wrong, check and fix PATH manually
ls -la ~/.avm/bin/anchor
# Should be a symlink pointing to anchor-0.27.0
```

---

### Phase 4 — Solana Build Pipeline Mismatch

```bash
anchor build
# Error: no such command: build-bpf
```

**Root Cause:** Solana deprecated `build-bpf` in favour of `build-sbf` (SBF = Solana Bytecode Format). Older Anchor versions still called `build-bpf`.

```
Old:  solana build-bpf   (deprecated)
New:  solana build-sbf   (current)

Anchor 0.27 → calls build-bpf → fails on modern Solana CLI
Anchor 0.29+ → calls build-sbf → works with modern Solana CLI
```

**Fix:** Align all three versions — Anchor, Solana CLI, and Rust toolchain must be compatible with each other.

---

### Phase 5 — TLS/SSL Errors During Solana Installation

```bash
curl -sSfL https://release.solana.com/v1.14.17/install | sh
# Error: error:0A000126:SSL routines::unexpected eof while reading

wget https://release.solana.com/v1.14.17/install
# Error: SSL_ERROR_SYSCALL
```

**Diagnosis:**

```bash
# Test general internet connectivity
curl https://google.com
# Returns HTTP 301 — internet works fine

# The issue is specific to release.solana.com
```

**Root Cause:** Ubuntu-20.04's OpenSSL version had TLS handshake compatibility issues with specific server configurations. The network was fine — the SSL stack interaction with `release.solana.com` was failing.

**Fix:** Migrate to Ubuntu-24.04 which ships a modern OpenSSL that handles these handshakes correctly.

---

### Phase 6 — The Decision: Modernise Instead of Patch

At this point, three independent issues all traced back to the same root cause — Ubuntu-20.04 being too old:

```
GLIBC too old      →  Anchor CLI fails
OpenSSL mismatch   →  Solana download fails
build-bpf removed  →  Anchor build fails
```

**Decision:** Stop patching Ubuntu-20.04. Install Ubuntu-24.04 as a fresh WSL2 distribution.

```powershell
# Run in PowerShell on Windows
wsl --install -d Ubuntu-24.04
```

---

### Phase 7 — VS Code Integration

After Ubuntu-24.04 was running:

```bash
# Navigate to your project inside WSL
cd ~/projects/hello-world

# Launch VS Code from WSL — Remote-WSL extension connects automatically
code .
```

VS Code opens on Windows but connects to the WSL2 filesystem — giving you a native Linux terminal with a full GUI editor.

---

### Phase 8 — Node.js Version Problem

```bash
npm install -g npm@11
# Error: npm@11 requires Node.js >=20.17.0 or >=22.9.0
# Current: Node.js v18.20.6
```

**Fix — Install nvm and upgrade Node.js:**

```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Reload shell config
source ~/.bashrc

# Install Node.js 22
nvm install 22
nvm use 22
nvm alias default 22

# Verify
node -v   # v22.x.x
npm -v    # 10.x.x or later
```

---

### Phase 9 — Mucho Installation (Partial)

```bash
npx mucho install
# Partially succeeds, then:
# Error: cargo: not found
# AVM installation halted
```

**Root Cause:** Mucho tried to install AVM (which requires Cargo/Rust) but Rust was not yet installed.

**Fix:** Install Rust first, then rerun Mucho.

---

### Phase 10 — Rust Installation and Recovery

```bash
# Install Rust via rustup (the official way)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Follow prompts — choose option 1 (default install)
# Then load Rust into the current shell
source ~/.cargo/env

# Verify
rustc --version
cargo --version
rustup --version
```

After Rust is available, rerun Mucho to complete the Solana/Anchor installation.

---

## 4. Detailed Issues & Root Causes

### Complete Issue Reference

| # | Problem | Root Cause | Fix |
|---|---------|-----------|-----|
| 1 | `rm -r` prompts on Git pack files | Git objects are write-protected | Use `rm -rf` |
| 2 | Anchor 0.31.1 GLIBC error | Ubuntu-20.04 has GLIBC 2.31; needs 2.38+ | Downgrade Anchor or migrate OS |
| 3 | AVM switches version but wrong binary runs | PATH precedence / stale symlink | Re-run `avm use <version>` |
| 4 | `anchor build` fails — no `build-bpf` | Solana deprecated `build-bpf` for `build-sbf` | Align Anchor + Solana CLI versions |
| 5 | SSL error downloading Solana | Ubuntu-20.04 OpenSSL TLS mismatch | Migrate to Ubuntu-24.04 |
| 6 | `wsl --list` not found inside Linux | `wsl` is a Windows command | Run WSL commands from PowerShell |
| 7 | npm@11 fails — Node.js too old | Node.js 18 < required 20.17/22.9 | Install Node.js 22 via nvm |
| 8 | Mucho halts — `cargo: not found` | Rust not installed | Install Rust via rustup first |
| 9 | `anchor`, `solana` commands missing | Incomplete install sequence | Complete Rust → Mucho → verify PATH |
| 10 | `pnpm install` fails | pnpm not installed | `npm install -g pnpm` |

### Understanding the WSL Command Context

A very common confusion for beginners:

```
WRONG — running inside WSL terminal:
$ wsl --list --verbose
bash: wsl: command not found

CORRECT — running in PowerShell (Windows):
PS C:\> wsl --list --verbose

Reason: wsl.exe is a Windows binary. It does not exist inside Linux.
        WSL management commands always run from Windows (PowerShell / CMD).
```

---

## 5. WSL2 Migration — Ubuntu-20.04 to Ubuntu-24.04

### Step-by-Step Migration

```powershell
# Step 1 — Check current state (PowerShell)
wsl --list --verbose
# NAME            STATE    VERSION
# Ubuntu-20.04    Running  2

# Step 2 — Shut down existing WSL
wsl --shutdown

# Step 3 — Install Ubuntu-24.04 alongside existing distro
wsl --install -d Ubuntu-24.04

# Step 4 — Verify both are installed
wsl --list --verbose
# NAME            STATE    VERSION
# Ubuntu-20.04    Stopped  2
# Ubuntu-24.04    Stopped  2

# Step 5 — Set Ubuntu-24.04 as default
wsl --set-default Ubuntu-24.04

# Step 6 — Start Ubuntu-24.04
wsl -d Ubuntu-24.04
```

### Inside Ubuntu-24.04 — First-Time Setup

```bash
# Update all packages first
sudo apt update && sudo apt upgrade -y

# Install essential build tools
sudo apt install -y build-essential curl wget git pkg-config libssl-dev

# Verify GLIBC version (should be 2.38+)
ldd --version
# ldd (Ubuntu GLIBC 2.39-0ubuntu8.3) 2.39
```

### Migrating Projects from Ubuntu-20.04

```powershell
# Option A — Copy files via Windows filesystem
# In Ubuntu-20.04:
cp -r ~/projects /mnt/c/Users/YourName/temp-migration/

# In Ubuntu-24.04:
cp -r /mnt/c/Users/YourName/temp-migration/projects ~/

# Option B — Export and import (full distro backup)
wsl --export Ubuntu-20.04 D:\backups\ubuntu20-backup.tar

# Option C — Use Git (recommended for code)
# Push to GitHub from Ubuntu-20.04, clone in Ubuntu-24.04
```

### Keeping Ubuntu-20.04 Until Validated

Do **not** delete Ubuntu-20.04 until you have confirmed everything works in 24.04:

```powershell
# Only after full validation — unregister old distro
wsl --unregister Ubuntu-20.04
```

---

## 6. Installing the Full Toolchain from Scratch

This is the complete, ordered installation sequence for a fresh Ubuntu-24.04 WSL2 environment.

### Step 1 — System Updates & Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y \
    build-essential \
    curl \
    wget \
    git \
    pkg-config \
    libssl-dev \
    libudev-dev \
    clang \
    cmake
```

### Step 2 — Install Node.js via nvm

```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Load nvm (or open a new terminal)
source ~/.bashrc

# Install and set Node.js 22 as default
nvm install 22
nvm use 22
nvm alias default 22

# Verify
node -v     # v22.x.x
npm -v      # 10.x.x
```

### Step 3 — Install pnpm

```bash
npm install -g pnpm

# Verify
pnpm --version
```

### Step 4 — Install Rust via rustup

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
# Choose option 1 (default install)

# Load into current shell
source ~/.cargo/env

# Add to shell config permanently
echo 'source ~/.cargo/env' >> ~/.bashrc

# Verify
rustc --version
cargo --version
rustup --version

# Install the Solana-required Rust toolchain
rustup component add rustfmt clippy
```

### Step 5 — Install Solana CLI

**Method A — Via Mucho (recommended for Solana ecosystem)**

```bash
# Mucho installs Solana, Anchor, and AVM in one command
npx mucho install

# If cargo error appears — Rust wasn't loaded yet
source ~/.cargo/env
npx mucho install
```

**Method B — Official Solana installer**

```bash
sh -c "$(curl -sSfL https://release.solana.com/stable/install)"

# Add to PATH
export PATH="$HOME/.local/share/solana/install/active_release/bin:$PATH"
echo 'export PATH="$HOME/.local/share/solana/install/active_release/bin:$PATH"' >> ~/.bashrc

# Verify
solana --version
```

### Step 6 — Install Anchor via AVM

```bash
# Install AVM
cargo install --git https://github.com/coral-xyz/anchor avm --locked --force

# Add AVM to PATH
export PATH="$HOME/.avm/bin:$PATH"
echo 'export PATH="$HOME/.avm/bin:$PATH"' >> ~/.bashrc

# Install latest stable Anchor
avm install latest
avm use latest

# Verify
anchor --version
```

### Step 7 — Configure Solana for Development

```bash
# Set to local development cluster (for testing)
solana config set --url localhost

# Or use devnet (free test SOL)
solana config set --url devnet

# Generate a new keypair (your wallet for development)
solana-keygen new --outfile ~/.config/solana/id.json

# Check configuration
solana config get

# Get test SOL on devnet
solana airdrop 2
```

### Full PATH Setup (add to `~/.bashrc`)

```bash
# Cargo / Rust
source ~/.cargo/env

# Solana
export PATH="$HOME/.local/share/solana/install/active_release/bin:$PATH"

# AVM / Anchor
export PATH="$HOME/.avm/bin:$PATH"

# nvm
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```

Apply all changes:

```bash
source ~/.bashrc
```

---

## 7. Visual Studio Code Integration

VS Code on Windows connects to WSL2 via the **Remote - WSL** extension, giving you a native Linux development experience with a full GUI editor.

### Setup

1. Install **VS Code** on Windows
2. Install the **Remote - WSL** extension in VS Code
3. Open Ubuntu-24.04 in your terminal
4. Navigate to your project directory
5. Run:

```bash
cd ~/projects/hello-world
code .
```

6. VS Code opens on Windows and automatically connects to WSL2
7. Verify the connection — bottom-left status bar shows:

```
WSL: Ubuntu-24.04
```

### Recommended Extensions for Solana/Rust Development

| Extension | Purpose |
|-----------|---------|
| **Remote - WSL** | Connect VS Code to WSL2 |
| **Rust Analyzer** | Rust language server — autocomplete, errors, go-to-definition |
| **Even Better TOML** | Syntax highlighting for `Cargo.toml` |
| **ESLint** | JavaScript/TypeScript linting |
| **Prettier** | Code formatting |
| **GitLens** | Enhanced Git history and blame |
| **Solana Snippets** | Code snippets for Solana programs (optional) |

### Why Store Projects Inside WSL (Not on `/mnt/c/`)

```
/home/user/projects/     ← FAST  (native Linux filesystem ext4)
/mnt/c/Users/.../        ← SLOW  (cross-filesystem access overhead)

Rule: Always create and work on projects inside the WSL filesystem.
      Only access /mnt/c/ when you need to exchange files with Windows apps.
```

---

## 8. Target Environment Specification

The recommended final architecture after completing this setup:

| Component | Recommended Version / Config |
|-----------|------------------------------|
| **OS** | Ubuntu-24.04 on WSL2 |
| **GLIBC** | 2.39 (ships with Ubuntu-24.04) |
| **Editor** | VS Code + Remote-WSL extension |
| **Node.js** | v22 (managed via nvm) |
| **npm** | Latest compatible with Node.js 22 |
| **Package Manager** | pnpm |
| **Rust** | Latest stable (via rustup) |
| **Cargo** | Ships with Rust/rustup |
| **Solana CLI** | Latest stable (via Mucho or official installer) |
| **Anchor** | Latest stable (via AVM) |
| **Project Storage** | `/home/<user>/projects/` |

---

## 9. Validation Checklist

Run these commands inside your WSL2 Ubuntu-24.04 terminal. Every command should return a version number without errors.

```bash
# System
ldd --version            # GLIBC 2.39

# Node.js ecosystem
node -v                  # v22.x.x
npm -v                   # 10.x.x
pnpm --version           # 9.x.x or later

# Rust
rustc --version          # rustc 1.xx.x
cargo --version          # cargo 1.xx.x
rustup --version         # rustup 1.xx.x

# Solana
solana --version         # solana-cli 1.18.x or later
solana config get        # Shows RPC URL and keypair path

# Anchor
anchor --version         # anchor-cli 0.30.x or later

# VS Code (from inside WSL)
code .                   # Opens VS Code connected to WSL

# Create a test Anchor project
anchor init hello-world  # Should complete without errors
cd hello-world
anchor build             # Should compile successfully
```

### Expected `anchor build` Output

```
BPF SDK: /home/user/.local/share/solana/install/active_release/bin/sdk/sbf
cargo build-sbf --arch sbf ...
   Compiling hello-world v0.1.0
    Finished release [optimized] target(s)
```

If you see `build-sbf` in the output (not `build-bpf`), the toolchain alignment is correct.

---

## 10. Lessons Learned

### 1. Start with the Right OS Version

The entire troubleshooting journey could have been avoided by starting with Ubuntu-24.04. Always check the minimum GLIBC requirement of your target toolchain before starting.

### 2. Blockchain Tooling Moves Fast

Modern blockchain development tooling (Solana, Anchor, Rust) moves quickly and consistently targets newer Linux distributions. Long-term support does not mean compatible with bleeding-edge dev tools.

### 3. Migrate, Don't Patch

When a base OS becomes a compatibility bottleneck, migrating to a modern OS is faster and more sustainable than continuously applying workarounds. Each patch introduces new risks.

### 4. Keep the Old Environment Until Validated

Never delete the old distro until every tool has been verified working in the new environment. Run both side by side during the transition period.

### 5. PATH Order Matters

When multiple versions of a tool are installed (e.g., `anchor-0.27.0` and `anchor-0.31.1`), the shell executes whichever appears first in `$PATH`. Always verify with `which <command>` after switching versions.

```bash
which anchor
# Shows the actual binary being invoked

echo $PATH
# Shows the full search order
```

### 6. WSL Commands Belong in PowerShell

`wsl` is a Windows executable. It cannot be run from inside a Linux terminal. All WSL management (list, shutdown, import, export) runs in PowerShell or CMD on the Windows side.

---

## 11. Quick Reference Cheat Sheet

### WSL Management (PowerShell)

```powershell
wsl --list --verbose                              # List distros
wsl --install -d Ubuntu-24.04                     # Install new distro
wsl --set-default Ubuntu-24.04                    # Set default
wsl --shutdown                                    # Stop all distros
wsl --export Ubuntu-24.04 D:\backup.tar          # Backup
wsl --unregister Ubuntu-20.04                     # Delete old distro
```

### Toolchain Installation Order

```
1. sudo apt update && sudo apt upgrade
2. Install system dependencies (build-essential, libssl-dev, etc.)
3. Install nvm → Node.js 22
4. Install pnpm
5. Install Rust via rustup
6. source ~/.cargo/env
7. Install Solana (Mucho or official installer)
8. Install AVM → Anchor
9. Update ~/.bashrc with all PATH entries
10. source ~/.bashrc
11. Run validation checklist
```

### Version Check Commands

```bash
ldd --version        # GLIBC
node -v              # Node.js
npm -v               # npm
pnpm --version       # pnpm
rustc --version      # Rust compiler
cargo --version      # Cargo build tool
solana --version     # Solana CLI
anchor --version     # Anchor CLI
```

### AVM — Anchor Version Manager

```bash
avm install latest        # Install latest Anchor
avm install 0.29.0        # Install specific version
avm use 0.29.0            # Switch to version
avm list                  # List installed versions
which anchor              # Verify active binary path
```

### nvm — Node Version Manager

```bash
nvm install 22            # Install Node.js 22
nvm use 22                # Use Node.js 22 in current shell
nvm alias default 22      # Set as default permanently
nvm list                  # List installed versions
nvm current               # Show active version
```

### Solana Config

```bash
solana config get                        # Show current config
solana config set --url devnet           # Use devnet
solana config set --url localhost        # Use local validator
solana-keygen new                        # Generate dev wallet
solana airdrop 2                         # Get test SOL (devnet)
solana balance                           # Check wallet balance
```

### Common Fixes

```bash
# Rust not found after install
source ~/.cargo/env

# Wrong Anchor version active
avm use <version>
which anchor           # verify

# Mucho fails — cargo not found
source ~/.cargo/env
npx mucho install

# pnpm missing
npm install -g pnpm

# PATH not persisting
echo 'source ~/.cargo/env' >> ~/.bashrc
source ~/.bashrc
```

---

*End of Lesson 25*
