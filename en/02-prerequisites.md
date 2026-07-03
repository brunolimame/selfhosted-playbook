# 02 - Prerequisites

## Hardware

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| RAM (host) | 8 GB | 16 GB+ |
| RAM for VM | 2 GB | 4 GB |
| Disk (host) | 30 GB free | 80 GB+ |
| Disk for VM | 20 GB | 40 GB+ |
| Processor | 2 cores | 4+ cores |

## Software

### VirtualBox (free)
- **Windows**: Download from https://www.virtualbox.org/wiki/Downloads (Windows hosts)
- **Linux**: `sudo apt install virtualbox` (Ubuntu/Debian)
- **macOS**: Download from https://www.virtualbox.org/wiki/Downloads (OS X hosts)

### Ubuntu Server ISO
- Download the latest LTS version: https://ubuntu.com/download/server
- Example: `ubuntu-24.04-live-server-amd64.iso`

### SSH Client
- **Windows**: PowerShell or [Windows Terminal](https://github.com/microsoft/terminal) (native SSH since Windows 10 1809)
- **Linux/macOS**: Native SSH in terminal

## Accounts

### Cloudflare (free)
- Create an account at https://dash.cloudflare.com/sign-up
- Required for DNS and Cloudflare Tunnel

### Domain registration
- Own domain (ex: `mydomain.com`)
- Recommended: Cloudflare Registrar, Namecheap, Registro.br (.com.br)
- Typical cost: $10-15/year

## Network

- Administrative access to the router (to configure VM network)
- Stable internet connection
- (Optional) Ability to configure DHCP reservation on the local network

## Quick check

Before starting, run on your host terminal:

**Windows (PowerShell)**:
```powershell
# Check if Hyper-V is disabled (can conflict with VirtualBox)
Get-WindowsOptionalFeature -FeatureName Microsoft-Hyper-V-All
```

**Linux/macOS**:
```bash
# Check virtualization support
egrep -c '(vmx|svm)' /proc/cpuinfo
```

## Next step

[Creating the VM](./03-vm/README.md) - Install VirtualBox and create the virtual machine.
