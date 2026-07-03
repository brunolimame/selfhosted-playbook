# 07 - Security

## 1. Firewall (UFW)

Enable the firewall on the VM and allow only the necessary ports:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH (essential)
sudo ufw allow 22/tcp comment 'SSH'

# If using DDNS + Port Forwarding (alternative method)
# sudo ufw allow 80/tcp comment 'HTTP'
# sudo ufw allow 443/tcp comment 'HTTPS'
# sudo ufw allow 8000/tcp comment 'Coolify'

sudo ufw enable
sudo ufw status verbose
```

With Cloudflare Tunnel, no HTTP ports need to be opened. Only SSH for administration.

## 2. SSH with key (disable password)

### Generate key on your host (if you don't already have one)

**Windows (PowerShell)**:
```powershell
ssh-keygen -t ed25519 -C "your-key"
```

**Linux/macOS**:
```bash
ssh-keygen -t ed25519 -C "your-key"
```

### Copy key to the VM

```bash
ssh-copy-id ubuntu@192.168.1.100
```

Or manually:
```bash
cat ~/.ssh/id_ed25519.pub | ssh ubuntu@192.168.1.100 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### Disable password login

```bash
sudo nano /etc/ssh/sshd_config
```

Change:
```
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
```

```bash
sudo systemctl restart sshd
```

## 3. Automatic updates

```bash
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

Select **Yes** to update automatically.

## 4. Fail2ban (brute force protection)

```bash
sudo apt install -y fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

## 5. Coolify - Security settings

In the Coolify panel:
1. **Settings > Registration**: Disable public registration
2. **Settings > Instance**: Force HTTPS if behind a proxy
3. **Settings > Token**: Generate an API token with minimum permissions
4. **Applications**: Avoid exposing management ports (ex: 8000) directly

## 6. Cloudflare - WAF (Web Application Firewall)

In the Cloudflare dashboard:
1. **Security > WAF**:

**Block direct IP access**:
Custom rule:
- Field: `IP Source Address`
- Operator: `equals`
- Value: `192.168.1.100`
- Action: `Block`

**Rate Limiting** (free: 10 rules):
- Protects against brute force attacks
- Limit: 100 requests per minute per IP

## 7. Cloudflare Tunnel - Restricted access

In the cloudflared `config.yml` file, you can add authentication:

```yaml
ingress:
  - hostname: admin.vm.myserver.com
    service: http://localhost:8000
    originRequest:
      accessToken: "your-token-here"
  - service: http_status:404
```

Or use **Cloudflare Access** to protect routes with SSO authentication:
1. In Cloudflare, go to **Access > Applications**
2. Create an application for `vm.myserver.com`
3. Configure access policies (email, SSO, etc.)

## 8. Encrypted backups

```bash
# Backup of Coolify directory
sudo tar czf ~/backup-coolify-$(date +%Y%m%d).tar.gz /var/lib/docker/volumes/coolify_* --transform 's,.*/,,'
```

Encrypt with GPG:
```bash
gpg --symmetric --cipher-algo AES256 backup-coolify-*.tar.gz
```

## 9. Security checklist

- [ ] UFW firewall active (only SSH allowed)
- [ ] SSH login by key only
- [ ] Root login disabled
- [ ] Unattended upgrades active
- [ ] Fail2ban installed and running
- [ ] Coolify with public registration disabled
- [ ] Cloudflare SSL/TLS on Full (strict)
- [ ] Cloudflare WAF configured
- [ ] Automated backups
- [ ] Docker accessible only via socket (`/var/run/docker.sock`)

## Next step

[Maintenance](../08-maintenance.md) - Backup, updates and monitoring.
