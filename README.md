# sshuntil

Enhanced SSH connection tool that waits for SSH availability, manages host keys intelligently, and supports flexible IP notation.

## Features

- ⏱️ **Smart waiting**: Polls port 22 until SSH is ready (configurable timeout)
- 🔑 **Intelligent key management**: Only removes conflicting host keys, preserves valid ones
- 🌐 **Flexible IP notation**: Use full IPs, partial IPs, or just host numbers
- 👤 **Quick username**: Just type `sshuntil root 15` – no flags needed
- 🔍 **Subnet auto-detection**: Automatically detects your current subnet if not configured
- 🔐 **Password support**: Supports both key-based and password authentication (via sshpass)
- ⚙️ **Configurable**: System-wide and user-specific configuration files

## Installation

```bash
# Download the script
curl -o sshuntil https://raw.githubusercontent.com/efnats/sshuntil/main/sshuntil
chmod +x sshuntil

# Optional: Move to PATH
sudo mv sshuntil /usr/local/bin/

# Check version
sshuntil --version
```

### Dependencies

- `bash` (4.0+)
- `ssh` and `ssh-keyscan`
- `nc` (netcat)
- `sshpass` (optional, only for password authentication)

## Usage

### Basic Examples

```bash
# Full IP address
sshuntil 192.168.28.194

# Host number only (uses SUBNET from config or auto-detection)
sshuntil 194
# → connects to 192.168.28.194

# Two octets (replaces last two octets of SUBNET)
sshuntil 21.11
# → connects to 192.168.21.11

# Three octets (replaces last three octets of SUBNET)
sshuntil 10.0.50
# → connects to 192.10.0.50
```

### Specifying Username

```bash
# These are equivalent:
sshuntil root 15
sshuntil -l root 15
# → connects as root@192.168.28.15

# With partial IPs
sshuntil admin 21.11
# → connects as admin@192.168.21.11

# Without username: uses SSH_USER from config (default: root)
sshuntil 15
```

### Remote Commands

```bash
# Execute command on remote host
sshuntil 194 "uptime"
sshuntil root 15 "df -h"
sshuntil admin 21.11 "systemctl status nginx"
```

### How Partial IPs Work

Given a configured or auto-detected SUBNET of `192.168.28.0/24`:

| Input | Expands to | Explanation |
|-------|------------|-------------|
| `194` | `192.168.28.194` | Uses full SUBNET prefix + host number |
| `21.11` | `192.168.21.11` | Uses first two octets from SUBNET |
| `10.0.50` | `192.10.0.50` | Uses first octet from SUBNET |
| `10.0.5.100` | `10.0.5.100` | Full IP, used as-is |

## Configuration

Create a config file at `/etc/sshuntil.conf` (system-wide) or `~/.config/sshuntil.conf` (user-specific):

```bash
# Subnet for partial IP expansion (auto-detected if not set)
SUBNET="192.168.28.0/24"

# SSH username (default: root)
SSH_USER="root"

# SSH password (empty = key-based auth)
PASSWORD=""

# Custom known_hosts file location
KNOWN_HOSTS="${HOME}/.ssh/known_hosts"

# Port check interval in seconds
CHECK_INTERVAL=3

# Maximum wait time in seconds
MAX_WAIT=180
```

### Subnet Auto-Detection

If `SUBNET` is not configured, `sshuntil` automatically detects your current subnet using:
1. `ip route` (Linux)
2. `ifconfig` (macOS/BSD)
3. `hostname -I` (fallback)

The detected subnet is displayed:
```
[INFO] Subnet: 192.168.1.0/24 (via auto-detection)
```

## Host Key Management

`sshuntil` intelligently manages SSH host keys:

- **New host**: Key is added to `known_hosts`
- **Known host with matching key**: No changes, connection proceeds
- **Known host with different key**: Old key is removed, new key is added (with warning)

This prevents unnecessary key removals while still handling host key changes (e.g., reinstalled systems).

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `2` | Invalid input (bad IP format, missing config) |
| `69` | Could not fetch SSH host key |
| `111` | SSH connection failed |
| `124` | Timeout waiting for SSH port |

## Use Cases

### Live System Deployment
```bash
# Wait for freshly booted GRML live system
sshuntil 194 "grml-version"
```

### Quick Admin Access
```bash
# Connect as different users
sshuntil root 15
sshuntil admin 15
sshuntil deploy 21.50
```

### Automated Server Setup
```bash
#!/bin/bash
# Wait for server and run setup
sshuntil root 50 "apt update && apt upgrade -y"
```

### Network Range Management
```bash
# Connect to different hosts in the same subnet
for i in {10..20}; do
  sshuntil $i "hostname" &
done
wait
```

### Testing After Reboot
```bash
# Reboot and wait for SSH to come back
ssh server "sudo reboot"
sleep 5
sshuntil server "uptime"
```

## Troubleshooting

### "Could not auto-detect subnet"
Set `SUBNET` manually in `~/.config/sshuntil.conf`:
```bash
echo 'SUBNET="192.168.1.0/24"' > ~/.config/sshuntil.conf
```

### "Timeout after 180s"
Increase `MAX_WAIT` in config or check if:
- Target host is powered on
- Network connectivity is working
- Firewall allows SSH (port 22)

### Password authentication not working
Install `sshpass`:
```bash
# Debian/Ubuntu
sudo apt install sshpass

# macOS
brew install hudochenkov/sshpass/sshpass
```

## Security Notes

- Store passwords in config files only in secure environments
- Config files should have restricted permissions (`chmod 600`)
- Consider using SSH keys instead of passwords
- Host key verification is always enabled (`StrictHostKeyChecking=yes`)

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## License

MIT License - see repository for details
