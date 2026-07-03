# 03-02 - Ubuntu Server Installation

## 1. Start the installation

1. In VirtualBox, select the `ubuntu-server` VM
2. Click **Start**
3. The VM will boot from the Ubuntu Server ISO
4. On the initial menu, select **Try or Install Ubuntu Server** and wait

## 2. Installation steps

### 2.1 Language
- Select `English` (the system language; configure pt_BR locale later if desired)
- Enter to continue

### 2.2 Keyboard layout
- Select `Portuguese (Brazil)` or `Portuguese`
- Enter to continue

### 2.3 Network connections
- The installer should automatically detect the bridge mode adapter
- Note the displayed **IP** (ex: `192.168.1.100`). You will use this IP to access via SSH during configuration
- Select **Done** and Enter

### 2.4 Configure Proxy
- Leave blank
- **Done**

### 2.5 Configure Ubuntu Archive Mirror
- Leave the default (`http://archive.ubuntu.com/ubuntu`)
- **Done**

### 2.6 Guided storage configuration
- Select **Use An Entire Disk**
- Select the disk (ex: `VBOX_HARDISK 40.0 GB`)
- **Done**

### 2.7 Storage configuration confirmation
- Confirm with **Continue**
- This will partition the disk automatically

### 2.8 Profile setup
| Field | Suggested value |
|-------|----------------|
| Your name | `Ubuntu VM` |
| Your server's name | `ubuntu-vm` |
| Pick a username | `ubuntu` |
| Password | Choose a strong password |
| Confirm password | Repeat the password |

> Important: Keep this username and password safe. It will be used for SSH and sudo.

### 2.9 Ubuntu Pro
- Select **Skip for now** (no need for Ubuntu Pro)
- **Continue**

### 2.10 SSH Setup
- Check **Install OpenSSH server** (with Enter)
- Select **Done**

### 2.11 Feature Server Snaps
- Do not select any snap
- **Done**

### 2.12 Installation
- The installer will copy the files
- At the end, select **Reboot Now**

### 2.13 Eject the ISO
- When VirtualBox asks to remove the media, click **Devices > Optical Drives > Remove disk from virtual drive**
- Or press Enter to remove automatically

## 3. First login

The VM will restart. Log in with the username and password defined.

```bash
ubuntu-vm login: ubuntu
Password: [your password]
```

## 4. Update the system

As soon as you log in, update the packages:

```bash
sudo apt update && sudo apt upgrade -y
```

## 5. Check IP

Confirm the VM IP:

```bash
ip a
```

The IP will be on the `enp0s3` interface or similar (usually `192.168.x.x`).

## 6. Test SSH (from your host)

On your **host** terminal (not the VM), test the SSH connection:

**Windows (PowerShell)**:
```powershell
ssh ubuntu@192.168.1.100
```

**Linux/macOS**:
```bash
ssh ubuntu@192.168.1.100
```

Replace `192.168.1.100` with the VM IP.

From now on, all configuration can be done via SSH, without needing the VirtualBox window.

## Next step

[VM Network Configuration](./03-network.md) - Configure fixed IP and hostname.
