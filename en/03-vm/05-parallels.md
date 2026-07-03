# 03-05 - Parallels Desktop

## Overview

[Parallels Desktop](https://www.parallels.com) is a VM hypervisor for macOS, with excellent performance on both Intel and Apple Silicon (M1/M2/M3/M4) Macs.

> **Note**: Parallels is a paid product (trial available). If you need a free option on macOS, use VirtualBox.

## 1. Install Parallels Desktop

1. Download from https://www.parallels.com
2. Install like any macOS application
3. A free trial is available if you want to test before purchasing

## 2. Download the Correct Ubuntu Server ISO

Choose based on your Mac's architecture:

| Mac Type | Architecture | Download |
|----------|-------------|----------|
| Apple Silicon (M1/M2/M3/M4) | ARM64 | [Ubuntu Server for ARM](https://ubuntu.com/download/server/arm) |
| Intel | AMD64 | [Ubuntu Server (AMD64)](https://ubuntu.com/download/server) |

## 3. Create the Virtual Machine

Open Parallels Desktop and click **File > New**.

### Step-by-step:

1. Select **Install from a file or DVD** and choose the Ubuntu Server ISO
2. Parallels will auto-detect the OS as Ubuntu Server
3. Name: `ubuntu-server`
4. Check **Customize settings before installation**
5. Configure:

   **General**:
   - Name: `ubuntu-server`

   **Hardware > CPU & Memory**:
   - RAM: 4096 MB
   - CPUs: 2

   **Hardware > Hard Disk**:
   - Size: 40 GB

   **Hardware > Network**:
   - Change from **Shared Network** to **Bridged Network** (essential for the VM to have its own IP on your LAN)

6. Click **Create**

## 4. Start the VM

Click **Start** or **Continue**. The Ubuntu Server installer will begin.

## 5. Parallels-specific Notes

- **Parallels Tools**: Will be suggested automatically. Optional for CLI servers.
- **Apple Silicon Performance**: Ubuntu Server for ARM runs natively with near-native performance.
- **Shared Network vs Bridged**: Shared Network uses NAT (VM is behind the Mac's IP). **Bridged** is required for the VM to have its own IP, which is necessary for Cloudflare Tunnel.
- **Snapshots**: Available via **VM > Manage Snapshots**.
- **CLI Only**: Since Ubuntu Server has no GUI, you can close the Parallels window after starting the VM and manage it via SSH.

## Next step

[Install Ubuntu Server on the VM](./02-ubuntu-server.md)
