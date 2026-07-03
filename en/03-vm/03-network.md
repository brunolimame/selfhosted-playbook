# 03-03 - VM Network Configuration

## 1. Configure fixed (static) IP

By default the VM uses DHCP. To prevent the IP from changing (which would break Cloudflare Tunnel), configure a static IP.

### 1.1 Find network information

On the VM, run:

```bash
ip route show default
ip a
```

Note:
- **Interface**: Ex: `enp0s3`
- **Gateway**: Ex: `192.168.1.1` (default route)
- **Current IP**: Ex: `192.168.1.100`
- **DNS**: Ex: `8.8.8.8` or the router IP

### 1.2 Edit netplan

Ubuntu Server uses netplan to configure the network. Edit the configuration file:

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

Or, if the file has another name (ex: `50-cloud-init.yaml`):

```bash
sudo ls /etc/netplan/
```

Replace the content with something like:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8
```

Adjust the values according to your local network.

### 1.3 Apply the configuration

```bash
sudo netplan apply
```

### 1.4 Verify

```bash
ip a
ping -c 3 8.8.8.8
ping -c 3 google.com
```

## 2. Configure DHCP reservation (optional)

If you prefer to keep DHCP but ensure the same IP always, configure **DHCP Reservation** on the router:

1. Access the router panel (usually `192.168.1.1`)
2. Find the **DHCP Reservation** or **Static DHCP** section
3. Add the VM MAC address with the desired IP
4. Save and restart the VM network

To find the VM MAC address:

```bash
ip link show enp0s3
```

Look for `link/ether xx:xx:xx:xx:xx:xx`.

## 3. Configure hostname

The hostname was already set to `ubuntu-vm` during installation. To confirm:

```bash
hostnamectl
```

If you need to change it:

```bash
sudo hostnamectl set-hostname ubuntu-vm
```

And add to `/etc/hosts`:

```bash
echo "127.0.1.1 ubuntu-vm" | sudo tee -a /etc/hosts
```

## 4. Test full connectivity

```bash
# Internal test
ping -c 2 127.0.0.1

# Gateway test
ping -c 2 192.168.1.1

# Internet test
ping -c 2 1.1.1.1

# DNS test
ping -c 2 google.com
```

All good? You are ready for the next step.

## 5. Shut down the VM safely

Always shut down the VM from the terminal:

```bash
sudo shutdown now
```

Do not close the VirtualBox window directly.

## Next step

[Domain and DNS](../04-domain.md) - Configure your domain on Cloudflare.
