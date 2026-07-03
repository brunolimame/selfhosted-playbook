# 05-01 - Cloudflare Tunnel

## Overview

Cloudflare Tunnel creates a secure, encrypted tunnel between your VM and the Cloudflare edge network. The `cloudflared` (agent) runs on the VM and establishes outbound connections to Cloudflare. No ports need to be opened on the router.

```
Internet -> vm.meuservidor.com -> Cloudflare DNS -> Cloudflare Edge
                                                         |
                     VM (cloudflared) <--- outbound tunnel -+
```

## 1. Install cloudflared on the VM

Access the VM via SSH:

```bash
ssh ubuntu@192.168.1.100
```

### Download and installation

```bash
# Download cloudflared
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb

# Install
sudo dpkg -i cloudflared-linux-amd64.deb

# Verify
cloudflared version
```

## 2. Authenticate with Cloudflare

```bash
cloudflared tunnel login
```

This will open a URL in the terminal. Since this is a VM without a browser, copy the URL and open it in your host's browser.

1. Copy the URL displayed in the terminal
2. Paste it into your host computer's browser
3. Log in to Cloudflare (if necessary)
4. Select the domain you added to Cloudflare
5. Click **Authorize**

After authorization, the certificate file will be downloaded automatically to `~/.cloudflared/cert.pem`.

## 3. Create the tunnel

```bash
cloudflared tunnel create meu-tunel
```

This creates a tunnel with a unique ID (e.g., `abcdef01-1234-5678-9abc-def012345678`). Write down this ID.

Expected output:
```
Created tunnel meu-tunel with id abcdef01-1234-5678-9abc-def012345678
```

## 4. Configure the tunnel

Create the configuration file:

```bash
nano ~/.cloudflared/config.yml
```

Content:

```yaml
tunnel: meu-tunel
credentials-file: /home/ubuntu/.cloudflared/abcdef01-1234-5678-9abc-def012345678.json

ingress:
  - hostname: vm.meuservidor.com
    service: http://localhost:8000
  - service: http_status:404
```

Explanation:
- `tunnel`: name of the created tunnel
- `credentials-file`: path to the tunnel credentials file (adjust the filename)
- `ingress`: maps the hostname `vm.meuservidor.com` to Coolify running on port `8000` on the VM
- The last line is a catch-all rule that returns 404

## 5. Create DNS record

Now point the Cloudflare DNS to the tunnel:

```bash
cloudflared tunnel route dns meu-tunel vm.meuservidor.com
```

This creates a CNAME record in Cloudflare pointing `vm.meuservidor.com` to the tunnel ID.

## 6. Run the tunnel as a service

### Test first

```bash
cloudflared tunnel run meu-tunel
```

If it works, stop with `Ctrl+C` and configure it as a service.

### Install as a systemd service

```bash
sudo cloudflared service install
```

Or manually:

```bash
sudo nano /etc/systemd/system/cloudflared.service
```

Content:

```ini
[Unit]
Description=Cloudflare Tunnel
After=network.target

[Service]
Type=simple
User=ubuntu
ExecStart=/usr/bin/cloudflared tunnel run meu-tunel
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
```

Check status:

```bash
sudo systemctl status cloudflared
```

## 7. Adjust SSL/TLS in Cloudflare

In the Cloudflare dashboard:
1. Go to **SSL/TLS > Overview**
2. Select **Full (strict)**
3. Go to **Edge Certificates** and enable **Always Use HTTPS**

## 8. Test access

In your browser, go to:

```
https://vm.meuservidor.com
```

If Coolify is not yet installed, you will see a connection refused error. This is normal - proceed to the Coolify installation.

## Tunnel maintenance

### View logs
```bash
sudo journalctl -u cloudflared -f
```

### Update cloudflared
```bash
sudo dpkg -i cloudflared-linux-amd64.deb
sudo systemctl restart cloudflared
```

### Remove the tunnel (if needed)
```bash
cloudflared tunnel delete meu-tunel
```

## Cloudflare Free plan limitations
- **No video streaming** (violates the terms)
- **Maximum 100 MB per request** for proxy downloads
- **3 free Page Rules**

## Next step

[Coolify - Installation](../06-coolify/README.md)
