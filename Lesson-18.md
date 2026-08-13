# Lesson 18: Networking Basics for DevOps Engineers

> Networking is the **invisible backbone of DevOps**. Every deployment, container, CI/CD pipeline, SSH connection, and cloud service runs on networking. Without a solid grasp of these fundamentals, debugging failed deployments, configuring firewalls, managing containers, or designing cloud infrastructure becomes guesswork. This lesson covers the networking concepts every DevOps engineer must understand — from IP addressing to DNS, protocols, and ports.

---

## Table of Contents

1. [Why Networking Matters in DevOps](#1-why-networking-matters-in-devops)
2. [LAN — Local Area Network](#2-lan--local-area-network)
3. [IP Addressing (IPv4)](#3-ip-addressing-ipv4)
4. [Switch — Internal Traffic Manager](#4-switch--internal-traffic-manager)
5. [Router & Gateway](#5-router--gateway)
6. [Subnet & Subnet Masking](#6-subnet--subnet-masking)
7. [CIDR Notation](#7-cidr-notation)
8. [NAT — Network Address Translation](#8-nat--network-address-translation)
9. [Firewall](#9-firewall)
10. [Ports](#10-ports)
11. [DNS — Domain Name Service](#11-dns--domain-name-service)
12. [OSI Model](#12-osi-model)
13. [TCP vs UDP Protocols](#13-tcp-vs-udp-protocols)
14. [Essential Networking Commands](#14-essential-networking-commands)
15. [Networking in DevOps Context](#15-networking-in-devops-context)
16. [Quick Reference Cheat Sheet](#16-quick-reference-cheat-sheet)

---

## 1. Why Networking Matters in DevOps

```
Every DevOps task touches networking:

  git push          →  TCP connection to GitHub over HTTPS (port 443)
  ssh user@server   →  TCP connection to remote server (port 22)
  docker run        →  Container gets its own virtual network interface
  kubectl apply     →  Pod-to-pod communication across cluster network
  CI/CD pipeline    →  Fetches dependencies, pushes artifacts over network
  curl api.service  →  DNS resolution + TCP + HTTP request
```

| DevOps Task | Networking Concept Involved |
|-------------|----------------------------|
| Deploy to cloud server | IP addressing, SSH, Firewall rules |
| Configure Docker networking | Subnets, NAT, Port forwarding |
| Set up Kubernetes | DNS, Load balancing, VPC, CNI plugins |
| Debug a failed connection | Ports, Firewall, DNS resolution |
| Configure CI/CD | HTTPS, TLS, DNS, Proxy settings |
| Set up monitoring | Ports, IP whitelisting, Firewall |

---

## 2. LAN — Local Area Network

A **LAN (Local Area Network)** is a network of devices connected together within a limited physical area — a home, office, or data center.

```
┌─────────────────────────────────────────────────────────┐
│                   HOME / OFFICE LAN                     │
│                                                         │
│   📱 Mobile          💻 Laptop          🖨️ Printer     │
│  192.168.0.101     192.168.0.102      192.168.0.103     │
│        │                 │                  │           │
│        └─────────────────┼──────────────────┘           │
│                          │                              │
│                     🔀 SWITCH                           │
│                          │                              │
│                     🌐 ROUTER ──────► Internet (WAN)    │
│                    192.168.0.1                          │
└─────────────────────────────────────────────────────────┘
```

### Key Facts

- All devices in a LAN share the **same network range** (e.g., `192.168.0.x`)
- Each device gets a **unique IP address** within that range
- Devices communicate with each other through the **Switch**
- Devices communicate with the outside world through the **Router**

> **WAN (Wide Area Network):** The network connecting multiple LANs across large distances. The Internet is the largest WAN. When your app communicates across regions or data centers, it's using a WAN.

---

## 3. IP Addressing (IPv4)

Every device on a network needs a unique identifier — its **IP address** (Internet Protocol address).

### IPv4 Structure

```
IP Address:   192  .  168  .   0   .   1
              ────    ────    ───    ───
              Octet1  Octet2  Octet3  Octet4

- 4 octets (groups), each separated by a dot
- Each octet = 8 bits → range: 0 to 255
- Total bits: 4 × 8 = 32 bits
- Total possible IPv4 addresses: 2³² = 4,294,967,296
```

### Binary Representation

The computer works in binary — humans read decimal:

| Decimal | Binary |
|---------|--------|
| `0` | `00000000` |
| `1` | `00000001` |
| `127` | `01111111` |
| `192` | `11000000` |
| `255` | `11111111` |

So `192.168.0.1` in binary is:
```
11000000.10101000.00000000.00000001
```

### Private vs Public IP Addresses

| Type | Range | Used For |
|------|-------|---------|
| **Private** | `10.0.0.0 – 10.255.255.255` | Internal networks (LAN) |
| **Private** | `172.16.0.0 – 172.31.255.255` | Internal networks (LAN) |
| **Private** | `192.168.0.0 – 192.168.255.255` | Home/office networks |
| **Public** | Everything else | Internet-facing addresses |
| **Loopback** | `127.0.0.1` | Refers to the device itself ("localhost") |

> **DevOps relevance:** Cloud servers have both a **private IP** (within the cloud VPC) and a **public IP** (internet-facing). Internal services communicate over private IPs; users reach your app via the public IP.

### IPv4 vs IPv6

| Feature | IPv4 | IPv6 |
|---------|------|------|
| Bit length | 32-bit | 128-bit |
| Example | `192.168.0.1` | `2001:0db8:85a3::8a2e:0370:7334` |
| Total addresses | ~4.3 billion | ~340 undecillion |
| Status | Dominant (but running out) | Increasingly adopted |

---

## 4. Switch — Internal Traffic Manager

A **Switch** connects devices within the same LAN and manages communication between them.

```
Device A (192.168.0.101) ──┐
Device B (192.168.0.102) ──┤──► SWITCH ──► Routes to correct device
Device C (192.168.0.103) ──┘
```

- The switch keeps a **MAC address table** — it knows which physical port each device is connected to
- When Device A sends data to Device B, the switch forwards it **only to Device B** — not to everyone (unlike older hubs)
- Switches operate at **Layer 2** (Data Link) of the OSI model
- They work with **MAC addresses** (hardware addresses), not IP addresses

> **DevOps context:** In cloud environments and Kubernetes, virtual switches handle traffic between containers and pods on the same node.

---

## 5. Router & Gateway

A **Router** connects different networks together — most importantly, it connects your LAN to the internet (WAN).

```
                  LAN                         WAN
┌─────────────────────┐        ┌──────────────────────────┐
│  192.168.0.101      │        │                          │
│  192.168.0.102  ────┼──────► │  ROUTER  ───────────►  Internet │
│  192.168.0.103      │        │  (Gateway)               │
└─────────────────────┘        └──────────────────────────┘
                            192.168.0.1 (LAN side)
                            203.0.113.5 (Public IP, WAN side)
```

### Gateway

- The **Gateway** is the IP address of the router — it is the **exit point** for all traffic leaving your LAN
- By default, this is typically `192.168.0.1` or `192.168.1.1` in home networks
- Any traffic destined for an IP outside the local network is sent to the Gateway first

```bash
# Check your gateway address in Linux
ip route show
# Output: default via 192.168.0.1 dev eth0
#                      ─────────── → this is your gateway

# Or
route -n
```

### What Happens When You Visit facebook.com

```
1. Your device (192.168.0.101) sends request to Gateway (192.168.0.1)
2. Router receives request, applies NAT (replaces your IP with its public IP)
3. Router forwards request to Facebook's server over the internet
4. Facebook's server responds to the router's public IP
5. Router translates back and delivers response to your device
```

---

## 6. Subnet & Subnet Masking

### The Problem Subnets Solve

When a switch receives data, how does it know if the destination is **inside** the LAN (send directly) or **outside** (send to router)? The answer is the **Subnet Mask**.

### What is a Subnet?

A **Subnet** (subnetwork) is a logical division of an IP network. All devices in the same subnet share an IP address range and can communicate directly without going through a router.

```
IP Address:     192.168.0.105
Subnet Mask:    255.255.255.0
                ─────────────
                Fixed part  │ Free part
                192.168.0   │ .0–255

All devices with 192.168.0.x are in the SAME subnet (same LAN).
A device with 192.168.1.x is in a DIFFERENT subnet.
```

### How Subnet Masking Works

The subnet mask uses `255` to mark the **fixed (network) portion** and `0` to mark the **free (host) portion**:

| Subnet Mask | Fixed Bits | Free Bits | Hosts Available |
|-------------|-----------|-----------|----------------|
| `255.0.0.0` | First 8 bits | Last 24 bits | 16,777,214 |
| `255.255.0.0` | First 16 bits | Last 16 bits | 65,534 |
| `255.255.255.0` | First 24 bits | Last 8 bits | 254 |
| `255.255.255.128` | First 25 bits | Last 7 bits | 126 |

### Example

```
IP:      192 . 168 . 0  . 50
Mask:    255 . 255 . 255 . 0
         ─────────────────── ──
         FIXED (network part) FREE (host part)

→ Network address: 192.168.0.0
→ Valid hosts:     192.168.0.1 – 192.168.0.254
→ Broadcast:       192.168.0.255
```

### Each Device Needs 3 Things to Communicate

```
┌─────────────────────────────────────────┐
│  1. IP Address   → Who am I?            │
│  2. Subnet Mask  → What's my network?   │
│  3. Gateway      → How do I reach WAN?  │
└─────────────────────────────────────────┘
```

---

## 7. CIDR Notation

**CIDR (Classless Inter-Domain Routing)** is a shorthand way to express an IP address and its subnet mask together.

```
192.168.0.0/24
            ──
            └── Number of FIXED bits in the subnet mask
```

| CIDR | Subnet Mask | Fixed Bits | Available Hosts |
|------|------------|-----------|----------------|
| `/8` | `255.0.0.0` | 8 | 16,777,214 |
| `/16` | `255.255.0.0` | 16 | 65,534 |
| `/24` | `255.255.255.0` | 24 | 254 |
| `/25` | `255.255.255.128` | 25 | 126 |
| `/32` | `255.255.255.255` | 32 | 1 (single host) |

### Real Examples

```
192.168.0.0/16  →  192.168.x.x  → First 16 bits fixed, last 16 free
192.168.0.0/24  →  192.168.0.x  → First 24 bits fixed, last 8 free
10.0.0.0/8      →  10.x.x.x     → First 8 bits fixed, last 24 free
```

> **DevOps relevance:** CIDR is used everywhere in cloud networking:
> - AWS VPC: `10.0.0.0/16` (65,534 IP addresses for your cloud network)
> - Kubernetes pod network: `10.244.0.0/16`
> - Security group rules: allow traffic from `0.0.0.0/0` (all IPs) or `192.168.1.0/24` (specific subnet)

---

## 8. NAT — Network Address Translation

**NAT** is the mechanism that allows multiple devices on a private LAN to share a single public IP address when accessing the internet.

```
                    LAN (Private)              Internet (Public)
┌────────────────────────────┐    ┌────────────────────────────┐
│  📱  192.168.0.101         │    │                            │ 
│  💻  192.168.0.102  ──────────►  Router: 203.0.113.5  ───►   │ Facebook
│  🖥️  192.168.0.103         │    │  (acts on behalf of all    │ Server
│                            │    │   private devices)         │
└────────────────────────────┘    └────────────────────────────┘

Facebook only sees 203.0.113.5 — it never sees your private IP.
```

### How NAT Works Step by Step

```
1. Your laptop (192.168.0.102) requests facebook.com
2. Router receives request → records: "102 wants to reach Facebook"
3. Router replaces source IP with its own public IP (203.0.113.5)
4. Facebook server responds to 203.0.113.5
5. Router checks its translation table → routes response to 192.168.0.102
```

### Why NAT Matters

| Benefit | Explanation |
|---------|------------|
| **Security** | Private IPs are hidden from the internet — attackers can't directly target internal devices |
| **IP conservation** | Thousands of devices share one public IP — solves IPv4 address exhaustion |
| **Isolation** | External parties cannot initiate connections to internal devices (unless port forwarding is configured) |

> **DevOps context:** In cloud environments (AWS, Azure, GCP), instances in private subnets use a **NAT Gateway** to access the internet (e.g., to download packages) without being directly reachable from the internet.

---

## 9. Firewall

A **Firewall** is a security system that monitors and controls incoming and outgoing network traffic based on predefined rules.

```
                     FIREWALL
Internet ──────────► [  RULES  ] ──────────► Your Server
                     │         │
                     │ ALLOW:  │
                     │  :80    │  ← HTTP web traffic
                     │  :443   │  ← HTTPS web traffic
                     │  :22 from my IP only │  ← SSH
                     │         │
                     │ DENY:   │
                     │  :3306  │  ← MySQL — block external access
                     │  all other ports │
```

### Firewall Rules Define

- Which **IP addresses** are allowed to connect
- Which **ports** are open or closed
- Which **direction** (inbound/outbound) traffic is permitted
- Which **protocols** (TCP/UDP) are allowed

### Types of Firewalls in DevOps

| Type | Where Used | Example |
|------|-----------|---------|
| **Host-based firewall** | On the server itself | `ufw`, `iptables` on Linux |
| **Network firewall** | Between networks | AWS Security Groups, Azure NSG |
| **Cloud firewall** | Cloud provider level | AWS Security Groups, GCP Firewall Rules |

### Linux Firewall Commands (`ufw`)

```bash
# Enable firewall
sudo ufw enable

# Allow SSH (always do this BEFORE enabling ufw on a remote server!)
sudo ufw allow 22

# Allow HTTP and HTTPS
sudo ufw allow 80
sudo ufw allow 443

# Allow from specific IP only
sudo ufw allow from 192.168.1.50 to any port 22

# Deny a port
sudo ufw deny 3306

# View current rules
sudo ufw status verbose

# Delete a rule
sudo ufw delete allow 80
```

> **Critical DevOps Rule:** Always allow port 22 (SSH) **before** enabling a firewall on a remote server. Locking yourself out of SSH means losing access to the server entirely.

---

## 10. Ports

A **Port** is a virtual communication endpoint on a device. If an IP address is like a building's street address, a port is like the specific door (room number) inside that building.

```
Server IP: 203.0.113.5

  Port 22   → SSH daemon     (remote terminal access)
  Port 80   → HTTP server    (web traffic, unencrypted)
  Port 443  → HTTPS server   (web traffic, encrypted)
  Port 3306 → MySQL database
  Port 5432 → PostgreSQL database
  Port 6379 → Redis cache
  Port 27017 → MongoDB
  Port 8080 → Alternative HTTP / application servers
```

### Well-Known Port Numbers

| Port | Protocol/Service | Used By |
|------|-----------------|---------|
| `22` | SSH | Remote server access |
| `25` | SMTP | Email sending |
| `53` | DNS | Domain name resolution |
| `80` | HTTP | Web servers (unencrypted) |
| `443` | HTTPS | Web servers (encrypted/TLS) |
| `3306` | MySQL | MySQL database |
| `5432` | PostgreSQL | PostgreSQL database |
| `6379` | Redis | Redis cache/message broker |
| `8080` | HTTP-alt | Dev servers, proxies |
| `27017` | MongoDB | MongoDB database |
| `2375/2376` | Docker | Docker daemon |
| `6443` | Kubernetes API | kubectl, cluster control |

### Port Rules

- Each port number is **unique per device** — only one process can bind to a port at a time
- Ports range from `0–65535`
- Ports `0–1023` are **well-known/system ports** (require root to bind)
- Ports `1024–49151` are **registered ports** (application use)
- Ports `49152–65535` are **dynamic/ephemeral ports** (temporary client connections)

### Check What's Running on a Port

```bash
# Check which process is using port 80
sudo ss -lpnt | grep :80
sudo netstat -lpnt | grep :80

# Check if a port is open on a remote server
telnet server-ip 80
nc -zv server-ip 80

# Scan open ports
nmap -p 22,80,443 server-ip
```

### Port Forwarding

Port forwarding directs traffic arriving at a specific port to another IP/port — commonly used to expose internal services:

```
External request → Router Port 8080 → Internal Server 192.168.0.10:3000
```

> **DevOps use case:** Kubernetes `kubectl port-forward` maps a pod's port to your local machine for debugging:
> ```bash
> kubectl port-forward pod/my-pod 8080:80
> # Local port 8080 → Pod port 80
> ```

---

## 11. DNS — Domain Name Service

### The Problem DNS Solves

Servers are identified by IP addresses, but humans can't memorize `172.217.16.46` for every website. DNS acts as the internet's **phone book** — translating human-readable names into machine-readable IP addresses.

```
You type:  www.google.com
              │
              ▼
         DNS Resolution
              │
              ▼
    172.217.16.46  ← actual IP your browser connects to
```

### How DNS Resolution Works

```
Browser: "What's the IP for google.com?"
  │
  ▼
1. CHECK local cache (already visited recently?)
  │ Not cached →
  ▼
2. ASK Recursive Resolver (your ISP's DNS server or 8.8.8.8)
  │ Not cached →
  ▼
3. ASK Root Name Server (one of 13 root servers, a–m)
  │ "I don't know google.com but .com TLD server does"
  ▼
4. ASK TLD Name Server (.com)
  │ "I don't have it but Google's authoritative server does"
  ▼
5. ASK Authoritative Name Server (Google's DNS)
  │ "google.com = 172.217.16.46"
  ▼
6. Answer returned → browser caches it → connects to IP
```

### DNS Hierarchy

```
                         . (Root)
                         │
        ┌────────────────┼────────────────┐
       .com            .org             .edu         ← TLDs
        │
  ┌─────┼─────┐
google  facebook  amazon               ← Second-level domains
  │
  ├── www.google.com
  ├── mail.google.com
  └── drive.google.com                 ← Subdomains
```

### Top-Level Domains (TLDs)

| TLD | Purpose |
|-----|---------|
| `.com` | Commercial businesses |
| `.org` | Non-profit organizations |
| `.edu` | Educational institutions |
| `.net` | Network infrastructure |
| `.gov` | Government entities |
| `.mil` | Military |
| `.pk`, `.uk`, `.us` | Country-specific (geographic TLDs) |

> **ICANN (Internet Corporation for Assigned Names and Numbers)** manages TLD development, domain space architecture, and authorizes **domain registrars** — the companies (like GoDaddy, Namecheap) that sell and assign domain names.

### Subdomains

Subdomains are prefixes added to a domain, creating separate addresses under the same domain:

```
google.com         → main website
www.google.com     → same (www is a subdomain)
mail.google.com    → Gmail
drive.google.com   → Google Drive
api.google.com     → API endpoints
```

In DevOps, subdomains are commonly used to separate environments:

```
app.mycompany.com        → production
staging.mycompany.com    → staging environment
api.mycompany.com        → API server
grafana.mycompany.com    → monitoring dashboard
```

### DNS Record Types

| Record | Purpose | Example |
|--------|---------|---------|
| `A` | Maps domain → IPv4 address | `google.com → 172.217.16.46` |
| `AAAA` | Maps domain → IPv6 address | `google.com → 2607:f8b0::...` |
| `CNAME` | Alias — maps domain → another domain | `www.google.com → google.com` |
| `MX` | Mail exchange — where to send email | `google.com → mail.google.com` |
| `TXT` | Text records — verification, SPF | Domain ownership verification |
| `NS` | Name server — who manages this domain's DNS | `google.com → ns1.google.com` |

### DNS Commands

```bash
# Look up the IP for a domain
nslookup google.com
dig google.com

# Look up a specific record type
dig google.com MX
dig google.com NS
dig google.com CNAME

# Reverse DNS — find domain from IP
dig -x 8.8.8.8

# Flush DNS cache (Linux)
sudo systemd-resolve --flush-caches

# Check which DNS server you're using
cat /etc/resolv.conf
```

---

## 12. OSI Model

The **OSI (Open Systems Interconnection) Model** is a 7-layer framework that standardizes how different computer systems communicate. As a DevOps engineer, understanding OSI helps you **pinpoint exactly where a network problem is occurring**.

```
┌─────┬──────────────────┬──────────────────────────────────────────┐
│  7  │  Application     │  User-facing protocols: HTTP, FTP, SSH,  │
│     │                  │  DNS, SMTP, DHCP                         │
├─────┼──────────────────┼──────────────────────────────────────────┤
│  6  │  Presentation    │  Data formatting, encryption, compression│
│     │                  │  TLS/SSL encryption lives here           │
├─────┼──────────────────┼──────────────────────────────────────────┤
│  5  │  Session         │  Manages sessions between apps           │
│     │                  │  Establishes, maintains, terminates      │
├─────┼──────────────────┼──────────────────────────────────────────┤
│  4  │  Transport       │  TCP (reliable) / UDP (fast)             │
│     │                  │  Port numbers, flow control              │
├─────┼──────────────────┼──────────────────────────────────────────┤
│  3  │  Network         │  IP addressing and routing               │
│     │                  │  Routers operate here                    │
├─────┼──────────────────┼──────────────────────────────────────────┤
│  2  │  Data Link       │  MAC addresses, frames                   │
│     │                  │  Switches operate here                   │
├─────┼──────────────────┼──────────────────────────────────────────┤
│  1  │  Physical        │  Cables, Wi-Fi signals, hardware         │
│     │                  │  Raw bits over physical medium           │
└─────┴──────────────────┴──────────────────────────────────────────┘
```

**Memory aid:** "**P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way" (Physical → Data Link → Network → Transport → Session → Presentation → Application)

### Why OSI Matters for Troubleshooting

```
Problem: "Can't reach my web server"

Layer 1 — Physical:   Is the server on? Network cable connected?
Layer 2 — Data Link:  Is the NIC recognized? MAC address issue?
Layer 3 — Network:    Can you ping the server IP? Routing issue?
Layer 4 — Transport:  Is port 80 open? Firewall blocking TCP?
Layer 7 — Application: Is nginx/apache running? Config error?
```

By testing layer by layer, you isolate where the fault is.

---

## 13. TCP vs UDP Protocols

**TCP** and **UDP** are the two main transport-layer protocols. They define how data is sent across a network.

### TCP — Transmission Control Protocol

TCP is **connection-oriented** and **reliable**. Before data is sent, a connection is established via a **3-way handshake**:

```
Client                          Server
  │                               │
  │──── SYN ──────────────────►   │   "I want to connect"
  │                               │
  │  ◄─── SYN-ACK ──────────────  │   "OK, I acknowledge"
  │                               │
  │──── ACK ──────────────────►   │   "Great, connected!"
  │                               │
  │  ══════ DATA TRANSFER ══════  │
```

- **Guaranteed delivery** — if a packet is lost, it is re-sent
- **Ordered** — packets are reassembled in the correct sequence
- **Slower** — overhead from acknowledgements and handshake
- **Used for:** HTTP/HTTPS, SSH, FTP, email, databases

### UDP — User Datagram Protocol

UDP is **connectionless** and **fast**. No handshake — data is simply sent:

```
Client                          Server
  │                               │
  │──── DATA ─────────────────►   │  (no acknowledgement)
  │──── DATA ─────────────────►   │  (no guarantee of delivery)
  │──── DATA ─────────────────►   │  (no order guarantee)
```

- **No guaranteed delivery** — lost packets are not re-sent
- **Unordered** — packets may arrive out of sequence
- **Faster** — no handshake overhead
- **Used for:** DNS lookups, video streaming, online gaming, VoIP, live broadcasts

### TCP vs UDP Comparison

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented (handshake) | Connectionless |
| Reliability | Guaranteed delivery | No guarantee |
| Order | Packets in order | May arrive out of order |
| Speed | Slower | Faster |
| Overhead | High (ACKs, handshake) | Low |
| Error checking | Yes | Minimal |
| Use cases | HTTP, SSH, FTP, DB | DNS, streaming, gaming, VoIP |

### The Analogy

```
TCP = Registered mail
      → You get confirmation it was delivered
      → If lost, it gets re-sent
      → Slower, but reliable

UDP = Dropping flyers from an airplane
      → Fast, no confirmation
      → Some may not land where intended
      → Doesn't matter if a few are lost
```

> **DevOps relevance:** When debugging connectivity issues, knowing whether your service uses TCP or UDP determines which tools to use and what symptoms to expect. Most infrastructure traffic (SSH, HTTP, DB) uses TCP.

---

## 14. Essential Networking Commands

These are the commands DevOps engineers use daily to diagnose and troubleshoot network issues.

### Connectivity Testing

```bash
# Test if a host is reachable (ICMP)
ping google.com
ping 8.8.8.8

# Trace the route packets take to a destination
traceroute google.com       # Linux
tracert google.com          # Windows

# Test if a specific port is open
nc -zv server-ip 80         # netcat — port open check
telnet server-ip 22         # test SSH port
curl -v http://server-ip    # test HTTP

# Test DNS resolution
nslookup google.com
dig google.com
host google.com
```

### Network Information

```bash
# Show your IP address and network interfaces
ip addr show
ifconfig               # older command

# Show routing table (find your gateway)
ip route show
route -n

# Show active connections and listening ports
ss -tulpn              # modern
netstat -tulpn         # older (needs net-tools)

# Show DNS configuration
cat /etc/resolv.conf

# Show hostname
hostname
hostname -I            # show all IPs
```

### Firewall Management

```bash
# UFW (Uncomplicated Firewall) — Ubuntu/Debian
sudo ufw status
sudo ufw allow 22
sudo ufw allow 80/tcp
sudo ufw deny 3306
sudo ufw enable

# iptables (lower level, all distributions)
sudo iptables -L -n -v          # list rules
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

### Download & HTTP Testing

```bash
# Make HTTP requests
curl http://example.com
curl -I http://example.com      # headers only
curl -v https://example.com     # verbose (shows TLS handshake)

# Download files
wget https://example.com/file.tar.gz

# Follow redirects and show status
curl -L -o /dev/null -w "%{http_code}" http://example.com
```

---

## 15. Networking in DevOps Context

### Cloud Networking (AWS Example)

```
┌──────────────────────────────────────────────────┐
│               AWS VPC (10.0.0.0/16)              │
│                                                  │
│  ┌─────────────────┐    ┌─────────────────┐      │
│  │ Public Subnet   │    │ Private Subnet  │      │
│  │ 10.0.1.0/24     │    │ 10.0.2.0/24     │      │
│  │                 │    │                 │      │
│  │  Web Server     │    │  Database       │      │
│  │  (has public IP)│    │  (no public IP) │      │
│  └────────┬────────┘    └────────┬────────┘      │
│           │                      │               │
│    Internet Gateway        NAT Gateway           │
└───────────┼──────────────────────┼───────────────┘
            │                      │
          Internet             (outbound only)
```

- **Public subnet:** Resources that need to be reachable from internet (web servers, load balancers)
- **Private subnet:** Resources that should not be directly reachable (databases, app servers)
- **Internet Gateway:** Allows public subnet resources to reach the internet
- **NAT Gateway:** Allows private subnet resources to initiate outbound internet connections (e.g., `apt update`) without being reachable from outside

### Docker Networking

```bash
# List Docker networks
docker network ls

# Inspect a network
docker network inspect bridge

# Create a custom network
docker network create my-app-network

# Run containers on the same network (they can reach each other by name)
docker run --network my-app-network --name db postgres
docker run --network my-app-network --name app my-app
# Now 'app' can reach 'db' using hostname 'db'
```

### Kubernetes Networking

- Every Pod gets its own IP address
- Pods on the same node communicate via virtual switches
- Services provide stable DNS names and IPs for Pod groups
- `ClusterIP`: internal-only service
- `NodePort`: exposes service on each node's IP
- `LoadBalancer`: provisions a cloud load balancer with public IP

```bash
# Check pod IPs
kubectl get pods -o wide

# Check services
kubectl get services

# Port forward for local debugging
kubectl port-forward svc/my-service 8080:80
```

---

## 16. Quick Reference Cheat Sheet

### IP & Subnet

```
Private ranges:   10.x.x.x / 172.16–31.x.x / 192.168.x.x
Loopback:         127.0.0.1 (localhost)
CIDR /24:         254 usable hosts (192.168.0.1–254)
CIDR /16:         65,534 usable hosts
CIDR /8:          16,777,214 usable hosts
```

### Common Ports

| Port | Service |
|------|---------|
| 22 | SSH |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 27017 | MongoDB |
| 8080 | HTTP-alt |
| 6443 | Kubernetes API |

### Diagnostic Commands

```bash
ping <host>                  # Reachability
traceroute <host>            # Route path
dig <domain>                 # DNS lookup
nslookup <domain>            # DNS lookup
ss -tulpn                    # Open ports & listeners
ip addr show                 # Network interfaces & IPs
ip route show                # Routing table / gateway
curl -v http://<host>        # HTTP test
nc -zv <host> <port>         # Port open test
```

### TCP vs UDP at a Glance

| | TCP | UDP |
|--|-----|-----|
| Reliable? | Yes | No |
| Fast? | Slower | Faster |
| Use for | HTTP, SSH, DB | DNS, streaming |

### OSI Layers (Top → Bottom)

```
7 Application  → HTTP, SSH, DNS, FTP
6 Presentation → TLS/SSL, encryption
5 Session      → Connection sessions
4 Transport    → TCP / UDP, ports
3 Network      → IP, routing, routers
2 Data Link    → MAC, Ethernet, switches
1 Physical     → Cables, Wi-Fi, hardware
```

---

*End of Lesson 19*