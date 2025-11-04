# sshuntil

**sshuntil** waits until an SSH host becomes reachable, cleans up old `known_hosts` entries, automatically trusts the new host key, and connects.

Perfect for automated deployments, VM resets, or network devices that take a while to boot.

---

## 🚀 Features
- Wait until SSH port is reachable  
- Remove outdated host key from `known_hosts`  
- Preload new host key via `ssh-keyscan` (no prompts)  
- Optional password login via `sshpass`  
- Supports shorthand host numbers with configurable subnet  
- Config file support (`~/.config/sshuntil.conf`)  

---

## ⚙️ Installation
Clone the repository and install the script:
```bash
sudo cp sshuntil /usr/local/bin/
sudo chmod +x /usr/local/bin/sshuntil
```

If you plan to use password authentication, install **sshpass**:
```bash
sudo apt install sshpass
```

---

## 🧩 Configuration

Example: `~/.config/sshuntil.conf`

```bash
SUBNET="192.168.28.0/24"
SSH_USER="root"
PASSWORD=""                   # optional, requires sshpass if used
KNOWN_HOSTS="${HOME}/.ssh/known_hosts"
CHECK_INTERVAL=3
MAX_WAIT=180
```

---

## 🖥️ Usage
```bash
sshuntil 192.168.28.194
sshuntil 194
sshuntil 194 "uptime"
```

---

## 📦 Dependencies
- `bash`
- `nc` (netcat)
- `ssh`
- `ssh-keyscan`
- `ssh-keygen`
- **`sshpass`** (optional, required for password-based login)

---

## 📄 License
MIT License  
© 2025 efnats
