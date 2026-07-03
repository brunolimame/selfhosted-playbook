# 03 - Virtual Machine (VM)

Guide to creating and configuring the virtual machine that will run the server.

## Choose your hypervisor

| Hypervisor | Platform | Cost | Guide |
|------------|----------|------|-------|
| VirtualBox | Windows, Linux, macOS | Free | [Guide](./01-virtualbox.md) |
| VMware Workstation Pro | Windows, Linux | Free (personal use) | [Guide](./04-vmware.md) |
| Parallels Desktop | macOS (Intel + Apple Silicon) | Paid | [Guide](./05-parallels.md) |

> **Note**: Ubuntu Server installation and network configuration steps are the same regardless of hypervisor.

## Steps

| Step | Description |
|------|-------------|
| [01 - VirtualBox](./01-virtualbox.md) | VirtualBox installation and VM configuration |
| [02 - Ubuntu Server](./02-ubuntu-server.md) | Operating system installation |
| [03 - Network](./03-network.md) | VM network configuration |

## Suggested specifications (any hypervisor)

| Setting | Value |
|---------|-------|
| Name | `ubuntu-server` |
| OS | Ubuntu Server LTS (64-bit) |
| RAM | 4096 MB |
| CPU | 2 cores |
| Disk | 40 GB |
| Network | Adapter in Bridge mode |

## Bridge mode network

The VM's network adapter must be in **Bridge mode**. This makes the VM receive an IP from the same network as your router, behaving as an independent device on the local network.

This is important because:
- The VM will have its own IP on the local network
- You will be able to access it via SSH through the IP
- Cloudflare Tunnel (and other methods) will work correctly

## Next step

Choose your hypervisor and create the VM:
- [VirtualBox](./01-virtualbox.md) (Windows, Linux, macOS - free)
- [VMware Workstation Pro](./04-vmware.md) (Windows, Linux - free for personal use)
- [Parallels Desktop](./05-parallels.md) (macOS - paid)
