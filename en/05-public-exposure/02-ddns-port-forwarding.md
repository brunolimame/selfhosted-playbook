# 05-02 - DDNS + Port Forwarding

## Overview

This traditional method exposes the VM directly on the internet using:
- **DDNS (Dynamic DNS)**: A service that automatically updates a DNS record when your public IP changes
- **Port Forwarding**: Redirecting ports from the router to the VM

```
Internet -> vm.meuservidor.com -> Router (Dynamic IP)
                                       |
                                    Port Forwarding
                                       |
                               VM (192.168.1.100)
```

## Security warning

This method **opens ports on the router**, exposing your network directly. Only use it if:
- You understand the security risks
- You have adequate firewall configuration
- You keep the system always updated

## 1. Configure DDNS (Dynamic DNS)

### Option 1: DuckDNS (free, simple)

1. Go to https://www.duckdns.org
2. Log in with a Google, GitHub, or Twitter account
3. Create a subdomain (e.g., `meuvm.duckdns.org`)
4. Write down the generated **token**

### Option 2: No-IP (free, requires monthly renewal)

1. Go to https://www.noip.com
2. Create a free account
3. Create a hostname (e.g., `meuvm.hopto.org`)
4. Install the No-IP client on the VM

### Install DDNS client on the VM

**For DuckDNS**:

Create an update script:

```bash
nano ~/duckdns.sh
```

```bash
#!/bin/bash
echo url="https://www.duckdns.org/update?domains=meuvm&token=YOUR_TOKEN&ip=" | curl -k -o ~/duckdns.log -K -
```

```bash
chmod +x ~/duckdns.sh
```

Add to crontab to run every 5 minutes:

```bash
crontab -e
```

Add the line:
```
*/5 * * * * /home/ubuntu/duckdns.sh
```

If you have your own domain and want to use DDNS with Cloudflare, use the Cloudflare API to update DNS. Tools like `ddclient` or `cloudflare-ddns` can help.

## 2. Configure Port Forwarding on the router

1. Access the router panel (usually `192.168.1.1`)
2. Log in (admin/admin or username/password on the router label)
3. Locate **Port Forwarding** or **Virtual Server** or **NAT**
4. Create the rules:

| External port | Internal IP | Internal port | Protocol | Description |
|--------------|------------|---------------|-----------|-----------|
| 80 | 192.168.1.100 | 80 | TCP | HTTP (redirect to HTTPS) |
| 443 | 192.168.1.100 | 443 | TCP | HTTPS (Coolify or proxy) |
| 8000 | 192.168.1.100 | 8000 | TCP | Coolify Web UI |

5. Save the settings

## 3. Configure firewall on the VM (UFW)

```bash
sudo ufw allow 22/tcp   # SSH
sudo ufw allow 80/tcp   # HTTP
sudo ufw allow 443/tcp  # HTTPS
sudo ufw allow 8000/tcp # Coolify

sudo ufw enable
sudo ufw status verbose
```

## 4. Reverse proxy (recommended)

To serve multiple applications on ports 80/443, install a reverse proxy like Nginx or Caddy:

```bash
sudo apt install -y nginx
```

Example Nginx configuration for Coolify:

```nginx
server {
    listen 80;
    server_name vm.meuservidor.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name vm.meuservidor.com;

    ssl_certificate /etc/ssl/certs/cloudflare.crt;
    ssl_certificate_key /etc/ssl/private/cloudflare.key;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

For SSL, use Let's Encrypt (certbot):
```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d vm.meuservidor.com
```

## 5. Test

1. Find your public IP: `curl ifconfig.me`
2. Access `http://YOUR_PUBLIC_IP:8000` (temporary)
3. If it works, access `https://vm.meuservidor.com`

## Limitations and risks

- **IP can change**: If DDNS fails, the service becomes inaccessible
- **Open ports**: Any vulnerability on the VM exposes your internal network
- **ISP may block**: Some providers block ports 80/443 on residential plans
- **No DDoS protection**: Attacks go directly to your connection

## Next step

[Coolify - Installation](../06-coolify/README.md)
