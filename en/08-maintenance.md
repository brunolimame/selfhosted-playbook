# 08 - Maintenance

## 1. System update

### Regular updates (recommended: weekly)

```bash
ssh ubuntu@192.168.1.100
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y
```

### Check if reboot is needed

```bash
# Check if there are packages that need a reboot
cat /var/run/reboot-required

# Check time since last boot
uptime
```

## 2. Cloudflare Tunnel update

```bash
# Download the latest version
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb
sudo systemctl restart cloudflared
```

## 3. Coolify update

```bash
curl -fsSL https://cdn.coollabs.io/coolify/upgrade.sh | sudo bash
```

## 4. Docker update

```bash
sudo apt update
sudo apt upgrade -y docker-ce docker-ce-cli containerd.io
sudo systemctl restart docker
```

## 5. Backup

### Backup Coolify data

```bash
#!/bin/bash
# Backup script ~/backup.sh

BACKUP_DIR="$HOME/backups"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

mkdir -p $BACKUP_DIR

# Stop Coolify temporarily for consistent backup
sudo systemctl stop coolify

# Backup Docker volumes
docker run --rm \
  -v coolify_db:/source:ro \
  -v $BACKUP_DIR:/backup \
  ubuntu tar czf /backup/coolify-db-$DATE.tar.gz -C /source .

docker run --rm \
  -v coolify_redis:/source:ro \
  -v $BACKUP_DIR:/backup \
  ubuntu tar czf /backup/coolify-redis-$DATE.tar.gz -C /source .

# Backup Cloudflare Tunnel configuration
cp ~/.cloudflared/config.yml $BACKUP_DIR/cloudflared-config-$DATE.yml

# Restart Coolify
sudo systemctl start coolify

# Remove old backups
find $BACKUP_DIR -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete
find $BACKUP_DIR -name "*.yml" -mtime +$RETENTION_DAYS -delete

echo "Backup completed: $DATE"
```

```bash
chmod +x ~/backup.sh
```

### Schedule weekly backup

```bash
crontab -e
```

Add:
```
0 3 * * 0 /home/ubuntu/backup.sh
```

(Runs every Sunday at 03:00)

### Copy backup to another location

**To your host computer**:
```bash
scp ubuntu@192.168.1.100:~/backups/coolify-db-*.tar.gz D:\backups\
```

**To cloud (optional)**:
Install rclone and configure:
```bash
sudo apt install -y rclone
rclone config
rclone copy ~/backups remote:my-backups
```

## 6. Full VM backup (VirtualBox snapshot)

In VirtualBox (with the VM powered off):
1. Select the `ubuntu-server` VM
2. Click **Snapshots**
3. Click **Take Snapshot**
4. Name: `Before-update-YYYY-MM-DD`

To restore:
1. Select the snapshot
2. Click **Restore**

## 7. Logs and monitoring

### Check service status

```bash
# Tunnel
sudo systemctl status cloudflared

# Coolify
sudo systemctl status coolify

# Docker
docker ps

# System resources
htop

# Disk
df -h
```

### Centralized logs

```bash
# Tunnel logs
sudo journalctl -u cloudflared -n 100 --no-pager

# Coolify logs
sudo journalctl -u coolify -n 100 --no-pager

# Access logs (if using Nginx)
sudo tail -f /var/log/nginx/access.log
```

### Simple monitoring with cron

Create a health check script:

```bash
nano ~/healthcheck.sh
```

```bash
#!/bin/bash
# Check services and send alert if something is stopped

SERVICES=("cloudflared" "coolify" "docker")
DATE=$(date)

for service in "${SERVICES[@]}"; do
  if ! systemctl is-active --quiet "$service"; then
    echo "[$DATE] ERROR: $service is not running" >> ~/healthcheck.log
    # Send email or notification (optional)
    # curl -s -X POST "https://api.telegram.org/botTOKEN/sendMessage" -d "chat_id=CHAT_ID&text=ERROR: $service stopped"
  fi
done

# Check disk space
USAGE=$(df / | tail -1 | awk '{print $5}' | tr -d '%')
if [ "$USAGE" -gt 90 ]; then
  echo "[$DATE] ALERT: Disk at $USAGE% usage" >> ~/healthcheck.log
fi
```

```bash
chmod +x ~/healthcheck.sh
```

Add to crontab:
```bash
*/30 * * * * /home/ubuntu/healthcheck.sh
```

## 8. Scheduled reboot (optional)

If the VM becomes unstable, schedule a weekly reboot:

```bash
sudo crontab -e
```

```
0 4 * * 0 /sbin/reboot
```

(Reboots every Sunday at 04:00)

## 9. Common troubleshooting

| Problem | Possible cause | Solution |
|---------|---------------|----------|
| Tunnel not connecting | VM without internet | Check `ping 8.8.8.8` |
| Coolify not accessible | Container stopped | `docker ps` and `docker logs coolify` |
| Domain not working | DNS not propagated | `dig vm.myserver.com` |
| Disk full | Logs or Docker images | `docker system prune -a` |
| VM slow | Low RAM | Increase RAM in VirtualBox |
| Docker permission error | User without docker group | `sudo usermod -aG docker $USER` |

## Back to start

[Documentation index](./README.md)
