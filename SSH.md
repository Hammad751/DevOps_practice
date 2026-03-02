
## Useful SSH Commands

| Task | Command | Description |
| :--- | :--- | :--- |
| Generate Key | `ssh-keygen -t ed25519` | Creates a secure key |
| Copy to Server | `ssh-copy-id user@host` | Installs key on remote |

### For finding the ssh-key fingerprit
```bash
ssh-keygen -lf ~/.ssh/id_ed25519.pub
```
### Important Note
> Always backup your `~/.ssh` folder before making major changes!

## ☁️ Cloud Deployment (DigitalOcean/AWS)
- **What to upload:** The full content of `id_ed25519.pub`.
- **Command to copy:** `cat ~/.ssh/id_ed25519.pub`
- **Note:** Never upload the fingerprint; cloud providers require the full public key string to authorize your session.

## 🔍 Troubleshooting: Connection Refused
- [ ] **Check Service:** `sudo service ssh status` (Must be running).
- [ ] **Check Port:** Ensure `sshd_config` is set to `Port 22`.
- [ ] **Firewall:** Ensure Windows Firewall allows Inbound TCP on Port 22.
- [ ] **WSL specific:** Try connecting to `localhost` instead of the 172.x.x.x IP.

## 🛑 Common SSH Issues & Fixes

### 1. Identity File Not Accessible
- **Cause:** PowerShell cannot resolve the `~` path for WSL files.
- **Fix:** Use the full network path: `\\wsl.localhost\DistroName\home\user\.ssh\id_file`

### 2. Authenticity of Host can't be established
- **Cause:** First-time connection to a new IP/Host.
- **Fix:** Type `yes` to add the fingerprint to `known_hosts`.

### 3. Connection Refused
- **Cause:** `sshd` service is not running in Linux.
- **Fix:** Run `sudo service ssh start` in the terminal.

## 🤝 The First Handshake (Host Verification)
When connecting to a server for the first time, SSH asks:
*"The authenticity of host ... can't be established. Continue connecting?"*

- **Action:** Type `yes`.
- **Result:** The server's public key is saved in `~/.ssh/known_hosts`.
- **Why?** This prevents "Man-in-the-Middle" attacks. If this message appears again for a server you've already visited, someone might be intercepting your connection!

## 🔐 Password & Permission Fixes
- [ ] **Reset Password:** Use `wsl -u root` then `passwd <username>`.
- [ ] **Enable Passwords:** Set `PasswordAuthentication yes` in `/etc/ssh/sshd_config`.
- [ ] **Restart SSH:** Always run `sudo service ssh restart` after config changes.

## ✅ Success: Local SSH Connection
- [x] SSH Server running on WSL2
- [x] Successfully logged in from Windows PowerShell
- [x] Host fingerprint verified and saved to `known_hosts`

### 💡 Lesson Learned
Windows PowerShell cannot easily read identity files directly from `\\wsl.localhost`. 
**Best Practice:** Store a copy of the private key in the Windows `$HOME\.ssh\` folder for host-to-guest connections.

## 📁 Moving Keys between WSL and Windows
- **WSL to Windows Path:** `/mnt/c/Users/<Username>/.ssh/`
- **Windows to WSL Path:** `\\wsl$\<DistroName>\home\<User>\`

> [!CAUTION]
> **Windows Permission Error:** If you get "Permissions for id_ed25519 are too open," use the `icacls` command to restrict access to only your user profile.

## 🛠️ Why am I still being asked for a password?
1. **Server-side:** Is the public key in `~/.ssh/authorized_keys`?
2. **Permissions:** Does the server have `chmod 600` on the authorized_keys file?
3. **Client-side:** Is the Windows file permission restricted to ONLY my user?
4. **Debug:** Use `ssh -v` to see the internal logs of the connection attempt.

## 🔒 The "Authorized Keys" Concept
For a server to allow a Key Login, the **Public Key** (`.pub`) must be appended to the `~/.ssh/authorized_keys` file on the server.

- **Analogy:** The Private Key is your physical key; the `authorized_keys` file is the lock on the door. You must install the lock before your key will work.
- **Permission Requirement:** The `authorized_keys` file MUST be `600` (Read/Write for owner only) or SSH will reject it for security reasons.

## 🪟 Windows SSH Agent Setup
By default, the SSH Agent is disabled on Windows. 

**Commands to enable:**
1. `Set-Service ssh-agent -StartupType Automatic`
2. `Start-Service ssh-agent`
3. `ssh-add <path_to_key>`

**Benefit:** Stores the passphrase in memory so you don't have to type it for every new terminal connection.