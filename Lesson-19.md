# Lesson 19: IaaS — Infrastructure as a Service & Secure Server Setup

> IaaS (Infrastructure as a Service) is the cloud computing model that powers modern DevOps. Instead of buying and maintaining physical servers, you rent virtualized infrastructure from providers like AWS and Azure — and spin up production-ready servers in minutes. But provisioning a server is only the first step. **Securing it properly** is where the real DevOps work begins. This lesson covers both: what IaaS is, and the complete professional workflow for hardening a freshly provisioned cloud server.

---

## Table of Contents

1. [What is IaaS?](#1-what-is-iaas)
2. [On-Premise vs IaaS](#2-on-premise-vs-iaas)
3. [Cloud Service Models — IaaS vs PaaS vs SaaS](#3-cloud-service-models--iaas-vs-paas-vs-saas)
4. [Major IaaS Providers](#4-major-iaas-providers)
5. [IaaS in the DevOps Workflow](#5-iaas-in-the-devops-workflow)
6. [Provisioning a New Server — The First Login](#6-provisioning-a-new-server--the-first-login)
7. [Checking Active Network Connections](#7-checking-active-network-connections)
8. [Security Rule #1 — Never Work as Root](#8-security-rule-1--never-work-as-root)
9. [Creating a Secure Non-Root User](#9-creating-a-secure-non-root-user)
10. [Setting Up SSH Key Authentication](#10-setting-up-ssh-key-authentication)
11. [Hardening SSH Configuration](#11-hardening-ssh-configuration)
12. [Configuring the Firewall](#12-configuring-the-firewall)
13. [Complete Server Hardening Checklist](#13-complete-server-hardening-checklist)
14. [Quick Reference Cheat Sheet](#14-quick-reference-cheat-sheet)

---

## 1. What is IaaS?

**Infrastructure as a Service (IaaS)** is a cloud computing model where a third-party provider delivers virtualized computing resources — servers, storage, and networking — over the internet on a **pay-as-you-go** basis.

```
┌──────────────────────────────────────────────────────────────┐
│                    THE IAAS MODEL                            │
│                                                              │
│   Your Company                   IaaS Provider               │
│   ─────────────                  ─────────────               │
│   • Builds applications          • Owns the hardware         │
│   • Manages OS & software        • Manages data centers      │
│   • Configures servers           • Manages networking        │
│   • Controls security policies   • Manages virtualization    │
│   • Pays per usage               • Guarantees uptime SLA     │
│                                                              │
│        ←──── Rented over internet ────→                      │
│              (Virtual Machines,                              │
│               Storage, Networks)                             │
└──────────────────────────────────────────────────────────────┘
```

### What IaaS Provides

| Resource | Description | AWS Example | Azure Example |
|----------|-------------|------------|---------------|
| **Compute** | Virtual machines, CPUs, RAM | EC2 Instances | Virtual Machines |
| **Storage** | Block, object, file storage | S3, EBS | Blob Storage, Disks |
| **Networking** | VPCs, load balancers, firewalls | VPC, ELB, Security Groups | VNet, Load Balancer, NSG |
| **DNS** | Domain management | Route 53 | Azure DNS |
| **CDN** | Content delivery | CloudFront | Azure CDN |

### What You Are Still Responsible For

```
┌──────────────────────────┐
│     Your Application     │  ← You manage
├──────────────────────────┤
│  Runtime / Middleware    │  ← You manage
├──────────────────────────┤
│   Operating System       │  ← You manage
├──────────────────────────┤
│   Virtualization Layer   │  ← Provider manages
├──────────────────────────┤
│   Physical Servers       │  ← Provider manages
├──────────────────────────┤
│   Data Center / Power    │  ← Provider manages
└──────────────────────────┘
```

> **Key insight:** With IaaS, you get the flexibility of your own servers without the burden of managing physical hardware. You focus on what your application needs — the provider handles everything underneath.

---

## 2. On-Premise vs IaaS

| Factor | On-Premise (Physical Servers) | IaaS (Cloud) |
|--------|-------------------------------|--------------|
| **Upfront cost** | High — buy servers, racks, cables | Zero — no hardware purchase |
| **Ongoing cost** | Power, cooling, maintenance staff | Pay per hour/GB used |
| **Setup time** | Weeks to months | Minutes |
| **Scalability** | Limited — must buy more hardware | Instant — spin up in seconds |
| **Maintenance** | Your team handles hardware failures | Provider replaces failed hardware |
| **Staff required** | Network engineers, sysadmins, security | DevOps / cloud engineers |
| **Flexibility** | Low — fixed capacity | Very high — scale up/down anytime |
| **Disaster recovery** | Complex and expensive | Built-in snapshots, multi-region |
| **Global reach** | Requires building in each location | Data centers worldwide |

### The Business Case

```
Startup scenario:
  On-premise: $50,000 upfront for servers → months to set up
              → hire IT staff → manage hardware failures
              → underutilized 70% of the time

  IaaS:       $50/month to start → running in 10 minutes
              → scale up during launch → scale down during quiet periods
              → only pay for what you use
```

---

## 3. Cloud Service Models — IaaS vs PaaS vs SaaS

IaaS is one of three fundamental cloud service models. Understanding where they differ helps you choose the right tool:

```
                    MORE CONTROL ←────────────────→ LESS CONTROL
                    MORE RESPONSIBILITY              LESS RESPONSIBILITY

IaaS ────────────────────────────────────────────────────────
     AWS EC2, Azure VMs, DigitalOcean Droplets
     You manage: OS, runtime, app, data
     Provider manages: hardware, network, virtualization

PaaS ────────────────────────────────────────────────────────
     Heroku, AWS Elastic Beanstalk, Google App Engine
     You manage: app code and data only
     Provider manages: OS, runtime, scaling, patching

SaaS ────────────────────────────────────────────────────────
     Gmail, Slack, GitHub, Jira
     You manage: nothing (just use the software)
     Provider manages: everything
```

| Model | You Manage | Best For |
|-------|-----------|---------|
| **IaaS** | OS, runtime, app, data | Full control, custom infrastructure |
| **PaaS** | App and data only | Fast deployment, no infra management |
| **SaaS** | Just use it | End-user software |

---

## 4. Major IaaS Providers

| Provider | Key Services | Market Position |
|----------|-------------|-----------------|
| **AWS (Amazon Web Services)** | EC2, S3, RDS, VPC, Lambda | Largest — 31% market share |
| **Microsoft Azure** | VMs, Blob Storage, AKS | 2nd — strong enterprise/Windows |
| **Google Cloud Platform (GCP)** | Compute Engine, GKE, BigQuery | 3rd — strong in data/ML |
| **DigitalOcean** | Droplets, Spaces, Managed DBs | Developer-friendly, simpler pricing |
| **Linode / Akamai** | Linode instances | Cost-effective alternative |
| **Hetzner** | Cloud Servers, Dedicated | Very affordable, European-based |
| **Vultr** | Cloud Compute | Global locations, simple pricing |

> **For DevOps beginners:** Start with **DigitalOcean** (simplest UI, predictable pricing) or **AWS** (industry standard, most job market demand).

---

## 5. IaaS in the DevOps Workflow

IaaS is the foundation on which every DevOps practice runs:

```
┌─────────────────────────────────────────────────────────────┐
│                  DEVOPS + IAAS WORKFLOW                     │
│                                                             │
│  Developer pushes code                                      │
│         │                                                   │
│         ▼                                                   │
│  CI/CD Pipeline (GitHub Actions / Jenkins)                  │
│         │  runs on IaaS (EC2 / VM)                          │
│         ▼                                                   │
│  Build & Test on IaaS servers                               │
│         │                                                   │
│         ▼                                                   │
│  Deploy to IaaS (staging environment)                       │
│         │  auto-provision new servers via Terraform         │
│         ▼                                                   │
│  Deploy to IaaS (production environment)                    │
│         │  load balanced across multiple VMs                │
│         ▼                                                   │
│  Monitor with tools running on IaaS                         │
│    (Prometheus, Grafana, ELK Stack)                         │
└─────────────────────────────────────────────────────────────┘
```

### Why DevOps Engineers Must Know IaaS

- **Provision servers** for development, staging, and production environments
- **Configure networking** — VPCs, subnets, security groups, load balancers
- **Automate infrastructure** using Terraform or Ansible on IaaS resources
- **Troubleshoot deployments** — network issues, port conflicts, failed connections
- **Manage costs** — right-size instances, use auto-scaling, monitor usage
- **Ensure security** — harden servers, manage SSH keys, configure firewalls

---

## 6. Provisioning a New Server — The First Login

When you create a new cloud VM (e.g., AWS EC2, DigitalOcean Droplet, Azure VM), the provider gives you:

- A **public IP address**
- A **root user** (or default admin user like `ubuntu` on AWS)
- **SSH access** via a key pair you selected during provisioning

### First Connection

```bash
# Connect as root (or default user) using your key
ssh root@your-server-ip

# AWS uses a default user 'ubuntu', not root
ssh -i ~/.ssh/your-key.pem ubuntu@your-server-ip
```

### Immediately Update the System

```bash
# Always update packages first — patches security vulnerabilities
sudo apt update && sudo apt upgrade -y
```

### Standard First-Login Hardening Sequence

```
1. Update system packages
       │
       ▼
2. Create a non-root user
       │
       ▼
3. Grant sudo privileges
       │
       ▼
4. Set up SSH key authentication for new user
       │
       ▼
5. Disable root SSH login
       │
       ▼
6. Disable password authentication
       │
       ▼
7. Configure firewall (UFW)
       │
       ▼
8. Deploy your application
```

> **Never skip this sequence on a production server.** An unprotected root account with password authentication is typically compromised within minutes of being exposed to the internet through automated brute-force attacks.

---

## 7. Checking Active Network Connections

Before and after configuring services, verify what is running and which ports are open.

### Install net-tools

```bash
# Install the package that provides netstat
sudo apt install net-tools
```

### Check Active Connections

```bash
# Show all active TCP connections with listening ports
netstat -lpnt
```

**Flag breakdown:**

| Flag | Meaning |
|------|---------|
| `-l` | Show only **listening** sockets (services waiting for connections) |
| `-p` | Show the **PID and program name** for each socket |
| `-n` | Show **numeric** IPs/ports (no DNS resolution — faster) |
| `-t` | Show **TCP** connections only |

**Sample output:**

```
Proto  Recv-Q  Send-Q  Local Address     Foreign Address  State   PID/Program
tcp    0       0       0.0.0.0:22        0.0.0.0:*        LISTEN  1024/sshd
tcp    0       0       0.0.0.0:80        0.0.0.0:*        LISTEN  2048/nginx
tcp    0       0       127.0.0.1:3306    0.0.0.0:*        LISTEN  3012/mysqld
```

**Reading this:**
- Port `22` → SSH is listening (remote access is active)
- Port `80` → Nginx web server is running
- Port `3306` on `127.0.0.1` → MySQL is running but only accessible locally (good — not exposed to internet)

### Modern Alternative — `ss` (pre-installed)

```bash
# Same as netstat -lpnt — no package install needed
ss -lpnt

# Show all connections (TCP + UDP)
ss -tupn

# Check a specific port
ss -lpnt | grep :22
ss -lpnt | grep :80
```

---

## 8. Security Rule #1 — Never Work as Root

> **The most important security principle on any Linux server: never use the root account for day-to-day operations.**

Root has **unrestricted access** to everything. A single typo can destroy the entire server:

```bash
# As root — INSTANT, IRREVERSIBLE destruction
rm -rf /          # deletes the entire OS
chmod -R 777 /    # opens every file to everyone
> /etc/passwd     # wipes all user accounts
```

### Why Root Is Dangerous

| Risk | Explanation |
|------|------------|
| **No safety net** | Root can delete, overwrite, or corrupt anything instantly |
| **No audit trail** | Hard to track what root did vs. other users |
| **Attack surface** | Brute-force attacks specifically target the root account |
| **No privilege boundary** | A compromised application running as root has full system access |

### The Secure Alternative — sudo

Instead of staying as root, create a **standard user** and grant them `sudo` — which allows executing specific commands with root privileges only when explicitly needed:

```bash
# The same dangerous command — now requires deliberate action
sudo rm -rf /          # must type 'sudo' + enter password
```

`sudo` also **logs every privileged command** to `/var/log/auth.log` — giving you a complete audit trail:

```bash
# View sudo audit log
sudo cat /var/log/auth.log | grep sudo
```

---

## 9. Creating a Secure Non-Root User

### Step 1 — Create the User

```bash
# Interactive — prompts for password and user info
sudo adduser devops_alice

# You'll be asked to:
# - Set a password
# - Confirm password
# - Enter optional user info (press Enter to skip each)
```

### Step 2 — Add to the sudo Group

```bash
# -aG: append to group (doesn't remove existing groups)
sudo usermod -aG sudo devops_alice
```

### Step 3 — Verify sudo Access

```bash
# Switch to the new user
su - devops_alice

# Test sudo
sudo whoami
# Output: root  ← confirms sudo is working

# Or check group membership
groups devops_alice
# Output: devops_alice : devops_alice sudo
```

### Step 4 — Open a Second Terminal to Test

> **Critical:** Before proceeding, open a **new terminal window** and test that you can SSH in as the new user. Do NOT close your root session until you've confirmed access works. A mistake here could lock you out permanently.

```bash
# From your local machine — test new user SSH access
ssh devops_alice@your-server-ip
```

---

## 10. Setting Up SSH Key Authentication

Password authentication is vulnerable to **brute-force attacks**. SSH key authentication uses cryptographic key pairs — exponentially more secure.

### How It Works

```
Your Local Machine                    Cloud Server
┌─────────────────────┐              ┌────────────────────────────┐
│                     │              │                            │
│  Private Key        │──── SSH ────►│  ~/.ssh/authorized_keys    │
│  (stays here,       │   connect    │  (contains your public key)│
│  never sent)        │◄─ if match ──│                            │
│                     │   → access   │                            │
└─────────────────────┘              └────────────────────────────┘

The server verifies: "does this private key match the public key I have?"
The private key is NEVER sent over the network.
```

### Step 1 — Generate SSH Key Pair (on your local machine)

```bash
# Ed25519 — modern, recommended (shorter key, stronger security)
ssh-keygen -t ed25519 -C "your_email@example.com"

# RSA 4096 — older, widely compatible
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# You'll be asked for a file location (press Enter for default)
# and an optional passphrase (recommended for extra security)
```

This creates two files:
```
~/.ssh/id_ed25519      ← PRIVATE KEY — never share, never expose
~/.ssh/id_ed25519.pub  ← PUBLIC KEY — this goes on the server
```

### Step 2 — View Your Public Key

```bash
cat ~/.ssh/id_ed25519.pub
# Output: ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... your_email@example.com
```

### Step 3 — Create the SSH Directory on the Server

Switch to your new user on the server and set up the SSH directory:

```bash
# On the server, as the new user (devops_alice)
su - devops_alice

# Create the .ssh directory
mkdir -p ~/.ssh

# Set correct permissions — SSH REQUIRES these exact permissions
# If permissions are wrong, SSH silently ignores the keys
chmod 700 ~/.ssh
```

### Step 4 — Add Your Public Key to authorized_keys

```bash
# Create the authorized_keys file and paste your public key inside
vim ~/.ssh/authorized_keys

# Paste the FULL content of your id_ed25519.pub file
# One key per line — you can add multiple keys for multiple machines

# Set permissions
chmod 600 ~/.ssh/authorized_keys
```

### Step 4 (Easier Alternative) — Use `ssh-copy-id`

From your **local machine**, run:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub devops_alice@your-server-ip
```

This automatically:
- Creates `~/.ssh/` on the server if it doesn't exist
- Appends your public key to `~/.ssh/authorized_keys`
- Sets correct permissions

### Step 5 — Test Key Authentication

```bash
# Connect using key (from your local machine)
ssh devops_alice@your-server-ip

# If you added a passphrase, you'll be prompted for it
# Otherwise — connects immediately without a password
```

If it connects without asking for a password — SSH key auth is working.

### SSH Directory Permissions — Critical

Wrong permissions cause SSH to **silently reject keys** with no helpful error message:

| Path | Required Permission | Why |
|------|-------------------|-----|
| `~/.ssh/` | `700` | Only owner can access the directory |
| `~/.ssh/authorized_keys` | `600` | Only owner can read/write |
| `~/.ssh/id_ed25519` (private key) | `600` | Only owner can read/write |
| `~/.ssh/id_ed25519.pub` (public key) | `644` | Owner rw, others read |

```bash
# Fix permissions if SSH keys aren't working
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
```

---

## 11. Hardening SSH Configuration

Once key authentication is working and verified, lock down SSH further.

### Edit the SSH Daemon Config

```bash
sudo vim /etc/ssh/sshd_config
```

### Key Settings to Change

```bash
# Disable root login via SSH entirely
PermitRootLogin no

# Disable password authentication — keys only
PasswordAuthentication no

# Only allow specific users to SSH in
AllowUsers devops_alice

# Disable empty passwords
PermitEmptyPasswords no

# Limit authentication attempts
MaxAuthTries 3

# Set SSH timeout (disconnect idle sessions after 5 minutes)
ClientAliveInterval 300
ClientAliveCountMax 0
```

### Apply the Changes

```bash
# Validate the config before restarting (catches syntax errors)
sudo sshd -t

# Restart SSH service to apply changes
sudo systemctl restart sshd
```

> **Critical warning:** Before setting `PasswordAuthentication no`, confirm that:
> 1. Your SSH key login works in a **new terminal window**
> 2. You can run `sudo` with the new user
>
> If you disable password auth without a working key, you will be **permanently locked out** of the server.

---

## 12. Configuring the Firewall

A properly configured firewall is the last line of defence — it controls exactly which traffic reaches your server.

### Install and Configure UFW

```bash
# UFW (Uncomplicated Firewall) — default on Ubuntu
sudo apt install ufw

# ALWAYS allow SSH first — before enabling the firewall
sudo ufw allow 22
# Or more explicitly:
sudo ufw allow OpenSSH

# Allow web traffic
sudo ufw allow 80    # HTTP
sudo ufw allow 443   # HTTPS

# Allow from a specific IP only (e.g., your office IP for admin access)
sudo ufw allow from 203.0.113.50 to any port 22

# Enable the firewall
sudo ufw enable

# Verify the rules
sudo ufw status verbose
```

**Sample output:**

```
Status: active

To                         Action      From
──                         ──────      ────
22/tcp                     ALLOW IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
443/tcp                    ALLOW IN    Anywhere
3306/tcp                   DENY IN     Anywhere
```

### Default Policy (Deny Everything Not Explicitly Allowed)

```bash
# Set default to deny all incoming, allow all outgoing
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Cloud Provider Security Groups

In addition to the server-level firewall, **cloud providers have their own firewall layer** called Security Groups (AWS) or Network Security Groups (Azure). Configure both:

```
Internet
    │
    ▼
Cloud Security Group  ← First line of defence (cloud provider)
    │  (only allows ports you define)
    ▼
Server UFW Firewall   ← Second line of defence (OS level)
    │  (additional rules)
    ▼
Your Application
```

---

## 13. Complete Server Hardening Checklist

Use this checklist every time you provision a new cloud server:

```
INITIAL SETUP
□ SSH into server as root (first time only)
□ Run: sudo apt update && sudo apt upgrade -y
□ Check active connections: ss -lpnt

USER SETUP
□ Create non-root user: sudo adduser <username>
□ Add to sudo group: sudo usermod -aG sudo <username>
□ Verify sudo works: su - <username> → sudo whoami

SSH KEY SETUP
□ Generate key pair on local machine: ssh-keygen -t ed25519
□ Create ~/.ssh/ on server: mkdir -p ~/.ssh && chmod 700 ~/.ssh
□ Add public key: vim ~/.ssh/authorized_keys (chmod 600)
□ Test key login in NEW terminal window ← critical step
□ Confirm sudo works as new user

SSH HARDENING (/etc/ssh/sshd_config)
□ Set: PermitRootLogin no
□ Set: PasswordAuthentication no
□ Set: MaxAuthTries 3
□ Validate config: sudo sshd -t
□ Restart: sudo systemctl restart sshd
□ Test connection again in new terminal ← confirm still works

FIREWALL
□ Allow SSH: sudo ufw allow 22
□ Allow needed ports (80, 443, etc.)
□ Enable: sudo ufw enable
□ Verify: sudo ufw status verbose
□ Configure cloud security group (AWS/Azure)

VALIDATION
□ Can SSH in as non-root user with key
□ Root SSH login is blocked
□ Password authentication is blocked
□ Only needed ports are open
□ sudo works for the new user
□ Close root session — you no longer need it
```

---

## 14. Quick Reference Cheat Sheet

### IaaS at a Glance

```
IaaS = Rent servers/storage/networking from cloud providers
You manage: OS, runtime, application, data
They manage: hardware, data center, virtualization

Key providers: AWS, Azure, GCP, DigitalOcean
```

### Network Diagnostics

```bash
sudo apt install net-tools    # Install netstat
netstat -lpnt                 # Active listening ports
ss -lpnt                      # Modern alternative (no install)
ss -lpnt | grep :80           # Check specific port
```

### User Management

```bash
sudo adduser <name>               # Create user (interactive)
sudo usermod -aG sudo <name>      # Grant sudo
su - <name>                       # Switch to user
sudo whoami                       # Verify sudo → prints "root"
groups <name>                     # Check group membership
id <name>                         # Show UID, GID, groups
```

### SSH Key Setup

```bash
# Local machine
ssh-keygen -t ed25519 -C "email@example.com"   # Generate keys
cat ~/.ssh/id_ed25519.pub                        # View public key
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@host  # Copy to server
ssh user@server-ip                               # Connect

# On server (as new user)
mkdir -p ~/.ssh && chmod 700 ~/.ssh
vim ~/.ssh/authorized_keys      # Paste public key
chmod 600 ~/.ssh/authorized_keys
```

### SSH Config Hardening

```bash
sudo vim /etc/ssh/sshd_config
# Set:
#   PermitRootLogin no
#   PasswordAuthentication no
#   MaxAuthTries 3

sudo sshd -t                    # Validate config (no errors)
sudo systemctl restart sshd     # Apply changes
```

### Firewall (UFW)

```bash
sudo ufw allow 22               # Allow SSH first!
sudo ufw allow 80               # HTTP
sudo ufw allow 443              # HTTPS
sudo ufw default deny incoming  # Block everything else
sudo ufw enable                 # Activate
sudo ufw status verbose         # Check rules
```

---

*End of Lesson 19*