# 03-04 - VMware Workstation

## Overview

[VMware Workstation Pro](https://www.vmware.com/products/workstation-pro.html) is now **free for personal use**. It is a powerful alternative to VirtualBox for running virtual machines on Windows and Linux.

## 1. Download and Install VMware Workstation Pro

1. Download from https://www.vmware.com/products/workstation-pro.html
2. Run the installer (Windows) or follow the Linux instructions
3. During installation, ensure the VMnet bridge driver is enabled (required for bridged networking)

## 2. Download Ubuntu Server ISO

Download the latest LTS version:

```
https://ubuntu.com/download/server
```

File: `ubuntu-24.04-live-server-amd64.iso` (example)

## 3. Create the Virtual Machine

Open VMware Workstation and click **Create a New Virtual Machine**.

### Step-by-step:

1. Select **Typical (recommended)**
2. Choose **Installer disc image file (iso)** and select the Ubuntu Server ISO
3. **Guest Operating System**: Linux > Ubuntu 64-bit
4. **Virtual Machine Name**: `ubuntu-server`
5. **Disk Capacity**: 40 GB, **Split virtual disk into multiple files**
6. **Customize Hardware**:
   - **Memory**: 4096 MB
   - **Processors**: 2 cores
   - **Network Adapter**: **Bridged** (Automatic)
   - Remove unnecessary devices: Sound Card, Printer, USB (optional)
7. Click **Finish**

## 4. Start the VM

Select the VM and click **Power on this virtual machine**. The Ubuntu Server installer will start.

## 5. VMware-specific Notes

- **VMware Tools**: Not essential for Ubuntu Server CLI. You can skip it.
- **Bridged Networking**: Works out-of-the-box in VMware. The VM will receive its own IP from your router via DHCP.
- **Snapshots**: Available via **VM > Snapshot > Take Snapshot**. Useful before system updates.
- **Copy/Paste**: You can enable drag-and-drop and copy/paste in VM settings if desired (not needed for CLI work).

## Next step

[Install Ubuntu Server on the VM](./02-ubuntu-server.md)
