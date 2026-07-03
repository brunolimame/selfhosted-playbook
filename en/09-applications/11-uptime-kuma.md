# 09-11 - Uptime Kuma

## What is it?

[Uptime Kuma](https://github.com/louislam/uptime-kuma) is a self-hosted, beautiful and easy-to-use uptime monitoring tool. It is the open-source alternative to Uptime Robot, Pingdom and StatusCake.

Main features:
- HTTP(s), TCP, Ping, DNS, WebSocket, Docker Container monitoring
- Notifications via Telegram, Discord, Email, WhatsApp and 90+ services
- Public status page (share with clients)
- 20-second interval
- Multi-language (including Portuguese)
- Uptime, latency and SSL certificate graphs
- Status query API
- One-click backup and restore

## Why Uptime Kuma in this project?

With dozens of running services, you need to know when something goes down:

| Service | Monitor | Example URL |
|---------|-----------|----------------|
| Typebot | HTTP | `https://bot.meuservidor.com` |
| n8n | HTTP | `https://n8n.meuservidor.com` |
| evolution-go | HTTP | `http://192.168.1.100:4000/manager/` |
| Dify | HTTP | `https://ia.meuservidor.com` |
| Chatwoot | HTTP | `https://atendimento.meuservidor.com` |
| Coolify | HTTP | `http://192.168.1.100:8000` |
| MinIO | HTTP | `http://192.168.1.100:9000/minio/health/live` |
| PostgreSQL | TCP | `192.168.1.100:5432` |
| Internet | Ping | `8.8.8.8` |

## Architecture

```
uptime-kuma (container) -> Port 3001
    |
    +-- SQLite (internal database)
    +-- Notifications (Telegram, Discord, Email...)
    +-- Status page (optional, public)
```

## Prerequisites

- Docker installed on the VM
- At least 256 MB free RAM

## Installation

### 1. Access the VM

```bash
ssh ubuntu@192.168.1.100
```

### 2. Create directory

```bash
mkdir -p ~/uptime-kuma
cd ~/uptime-kuma
```

### 3. Create docker-compose.yml

```bash
nano docker-compose.yml
```

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:2
    container_name: uptime-kuma
    restart: unless-stopped
    ports:
      - "3001:3001"
    volumes:
      - uptime_kuma_data:/app/data

volumes:
  uptime_kuma_data:
```

### 4. Start

```bash
docker compose up -d
```

### 5. Verify

```bash
docker compose ps
docker compose logs -f
```

## Initial setup

1. Access `http://192.168.1.100:3001`
2. Create the admin user (name, email, password)
3. Select **SQLite** as the database

## Configure domain on Cloudflare Tunnel

```bash
nano ~/.cloudflared/config.yml
```

```yaml
  - hostname: status.meuservidor.com
    service: http://localhost:3001
```

```bash
sudo systemctl restart cloudflared
```

## Add monitors

### Monitor HTTP services

Click on **Add Monitor** and configure:

```
Monitor Type: HTTP(s)
Name: Typebot - Builder
URL: https://bot.meuservidor.com
Interval: 30s
Resend Notification: 3 times
Notification: Telegram (configure)
```

### Monitor TCP services (database)

```
Monitor Type: TCP Port
Name: PostgreSQL
Hostname: 192.168.1.100
Port: 5432
Interval: 60s
```

### Monitor ping (internet)

```
Monitor Type: Ping
Name: Internet - Google DNS
Hostname: 8.8.8.8
Interval: 60s
```

### Monitor SSL certificate

```
Monitor Type: HTTP(s)
Name: SSL - bot.meuservidor.com
URL: https://bot.meuservidor.com
Resend Notification: 1x per day (only if changed)
```

## Configure notifications

### Telegram

1. Click on **Settings > Notifications > Add Notification**
2. Type: **Telegram**
3. Bot Token: (token from @BotFather)
4. Chat ID: (get with `@userinfobot` or send `/start` to the bot then `https://api.telegram.org/botTOKEN/getUpdates`)
5. Test the notification

### Discord

1. Type: **Discord**
2. Webhook URL: (create in Discord: Channel Settings > Integrations > Webhooks)
3. Test

### Email (SMTP)

1. Type: **SMTP**
2. Host, Port, User, Pass according to your email provider
3. Test

## Create public Status Page

1. Click on **Status Page > Add Status Page**
2. Slug: `status` (will be at `https://status.meuservidor.com/status`)
3. Title: `Service Status`
4. Select the monitors to display
5. Enable **Publish**
6. Share the link with clients: `https://status.meuservidor.com/status`

## Maintenance

### Update

```bash
cd ~/uptime-kuma
docker compose pull
docker compose up -d
```

### Backup

```bash
# Full SQLite backup
docker run --rm -v uptime_kuma_data:/source -v ~/backups:/backup alpine tar czf /backup/uptime-kuma-$(date +%Y%m%d).tar.gz -C /source .
```

### Export/Import monitors

In the panel: **Settings > Backup > Create Backup** (JSON file with all configuration).

## Troubleshooting

| Error | Cause | Solution |
|------|-------|---------|
| Monitor shows "down" | Service actually down | Check if the service is running |
| Notification not arriving | Wrong token/configuration | Test configuration on notification screen |
| SSL certificate expired | Certificate expired | Check Cloudflare or certbot |
| Request timeout | Slow service or firewall | Increase monitor timeout to 30s |
| Corrupted backup | Corrupted SQLite | Stop container, copy `kuma.db`, run `sqlite3 kuma.db .dump` |

## Next step

[Netdata](./12-netdata.md) - Real-time VM monitoring.
