## 🖥️ Local WSL Server Info
- **Distro:** Ubuntu (WSL2)
- **Current IP:** 172.26.133.181
- **Interface:** eth0
- **Login Command:** `ssh meta@172.26.133.181`

### Troubleshooting
If SSH fails after a reboot, check the IP again with:
```bash
ip addr show eth0 | grep inet
```

## 🔄 Server Lifecycle
- **Start Service:** `sudo service ssh start`
- **Check IP:** `ip addr show eth0`
- **Connect:** `ssh meta@localhost` (or the IP)
- **Persistence:** Closing the window doesn't immediately kill the server; restarting Windows does.


## Quick Start
| Action | Command | Where |
| :--- | :--- | :--- |
| Start Server | sudo service ssh start | Ubuntu |
| Check Health | sudo service ssh status | Ubuntu |
| Load Key | ssh-add $HOME\.ssh\id_ed25519 | PowerShell |
| Enter Lab | ssh meta@localhost | PowerShell |

## Docker Lifecycle

| Command | What it does | When to use it |
| :--- | :--- | :--- |
| docker ps -a | Shows all containers | When you "lose" a container or check status |
| docker run | Creates a new container | Only the first time you set up the tool |
| docker start | Wakes up an existing container | Every morning after you restart your PC |