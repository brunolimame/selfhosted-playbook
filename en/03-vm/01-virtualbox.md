# 03-01 - VirtualBox Installation and VM Creation

## 1. Install VirtualBox

### Windows
1. Go to https://www.virtualbox.org/wiki/Downloads
2. Download `VirtualBox-x.x.x-xxx-Win.exe`
3. Run the installer as Administrator
4. Keep the default options and proceed
5. If a network warning appears, confirm the driver installation

> **Important**: Disable Hyper-V if it is active (WSL, Docker Desktop may conflict). Run in PowerShell as admin:
> ```powershell
> Disable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
> ```
> And restart the computer.

### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install -y virtualbox virtualbox-ext-pack
```

### macOS
1. Download `VirtualBox-x.x.x-xxx-OSX.dmg`
2. Open the file and drag VirtualBox to the Applications folder
3. If macOS blocks it, go to `System Preferences > Security & Privacy` and allow it

## 2. Download Ubuntu Server ISO

Download the latest LTS version:
```
https://ubuntu.com/download/server
```

File: `ubuntu-24.04-live-server-amd64.iso` (example)

## 3. Create the Virtual Machine

Open VirtualBox and click **New**.

### VM Settings

| Field | Value |
|-------|-------|
| Name | `ubuntu-server` |
| Machine folder | Leave default |
| ISO Image | `ubuntu-24.04-live-server-amd64.iso` (select now) |
| Type | Linux |
| Version | Ubuntu (64-bit) |
| RAM | 4096 MB |
| CPU | 2 CPUs |
| Hard disk | Create a virtual hard disk now |
| Disk size | 40 GB |
| Disk type | VDI (VirtualBox Disk Image) |
| Storage | Dynamically allocated |

On the creation screen, click **Finish**.

## 4. Adjust VM settings

Select the `ubuntu-server` VM and click **Settings**:

### Network
- **Network** tab
- **Adapter 1**: Enabled
- **Attached to**: `Bridged Adapter`
- **Name**: Select your physical network adapter (Wi-Fi or Ethernet)

### Processor
- **System** tab > **Processor**
- Sliders: 2 CPUs, 100% execution

### Video
- **Video** tab
- Video memory: 16 MB (minimum required for server)
- 3D acceleration: disabled

Click **OK**.

## 5. Verification

Your `ubuntu-server` VM should appear in the VirtualBox list with the configuration:
- Powered off
- Ubuntu (64-bit)
- 4096 MB RAM
- Bridge network

## Next step

[Install Ubuntu Server on the VM](./02-ubuntu-server.md) - Start the VM and follow the installer.
