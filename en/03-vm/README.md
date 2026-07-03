# 03 - Virtual Machine (VM)

Guide to creating and configuring the virtual machine that will run the server.

## Steps

| Step | Description |
|------|-------------|
| [01 - VirtualBox](./01-virtualbox.md) | VirtualBox installation and VM configuration |
| [02 - Ubuntu Server](./02-ubuntu-server.md) | Operating system installation |
| [03 - Network](./03-network.md) | VM network configuration |

## Overview

The VM will run Ubuntu Server LTS (no graphical interface) inside VirtualBox. All interaction with it will be via SSH.

## Suggested specifications

| Setting | Value |
|---------|-------|
| Name | `ubuntu-server` |
| Type | Linux / Ubuntu (64-bit) |
| RAM | 4096 MB |
| CPU | 2 cores |
| Disk | 40 GB (dynamic) |
| Network | Adapter in Bridge mode |

## Bridge mode network

The VM's network adapter must be in **Bridge mode**. This makes the VM receive an IP from the same network as your router, behaving as an independent device on the local network.

This is important because:
- The VM will have its own IP on the local network
- You will be able to access it via SSH through the IP
- Cloudflare Tunnel (and other methods) will work correctly

## Next step

[Install VirtualBox and create the VM](./01-virtualbox.md)
