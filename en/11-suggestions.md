# 11 - Next Application Suggestions

Based on the already documented ecosystem, here are suggestions for what to add next, organized by category.

## Monitoring and Observability

### Uptime Kuma
Uptime monitor with beautiful dashboard, notifications and public status page.

```
docker compose:
  - port: 3001
  - image: louislam/uptime-kuma
  - depends on: nothing
```

Why add it: Monitors all VM applications (Coolify, Typebot, n8n, evolution-go) and sends an alert if something goes down.

### Netdata
Real-time monitoring of CPU, RAM, disk, network for the entire VM.

```
docker compose:
  - port: 19999
  - image: netdata/netdata
  - depends on: nothing
```

Why add it: Complete visibility of VM resources, essential to know if RAM or CPU is lacking.

### Sentry (self-hosted)
Error tracking for applications. Capture exceptions from Typebot, n8n, Dify.

```
docker compose:
  - port: 9000
  - image: getsentry/sentry
  - depends on: postgres, redis
```

Why add it: Know when an application broke and why, with full stack trace.

## Database

### pgAdmin
Web interface for PostgreSQL administration. Useful for all apps that use Postgres.

```
docker compose:
  - port: 5050
  - image: dpage/pgadmin4
```

### Redis Commander
Web interface for Redis administration (used by Dify, Chatwoot, evolution-api).

```
docker compose:
  - port: 8081
  - image: rediscommander/redis-commander
```

## CI/CD and Git

### Gitea / Forgejo
Self-hosted Git server (like GitHub). Allows hosting private repositories and integrates with Coolify.

```
docker compose:
  - port: 3000
  - image: gitea/gitea
  - depends on: postgres
```

Why add it: Host application repositories locally, integrating with Coolify for automatic deployment.

### Woodpecker CI
Lightweight CI/CD pipeline that integrates with Gitea. Runs tests and deploys automatically.

```
docker compose:
  - port: 8000
  - image: woodpeckerci/woodpecker-server
  - depends on: postgres, gitea
```

## Communication

### Mattermost
Self-hosted team chat (alternative to Slack). Integrates with n8n for notifications.

```
docker compose:
  - port: 8065
  - image: mattermost/mattermost
  - depends on: postgres
```

## DNS and Proxy

### AdGuard Home
Ad and tracker blocking at the DNS level for the entire network.

```
docker compose:
  - port: 80/3000
  - image: adguard/adguardhome
```

### Nginx Proxy Manager
Web interface for managing reverse proxies and SSL certificates. Simpler alternative to Cloudflare Tunnel.

```
docker compose:
  - port: 80/81/443
  - image: jc21/nginx-proxy-manager
  - depends on: nothing
```

## Backup

### Duplicati
Automatic backup with encryption to cloud (Google Drive, S3, etc.).

```
docker compose:
  - port: 8200
  - image: linuxserver/duplicati
```

### BorgBackup + Borgmatic
Efficient backup with deduplication and compression.

```bash
sudo apt install borgmatic
```

## Priority summary

| Priority | App | Reason |
|----------|-----|--------|
| High | **Uptime Kuma** | Know if services are online |
| High | **Netdata** | Monitor VM resources |
| Medium | **Gitea** | Host repositories locally |
| Medium | **pgAdmin** | Manage PostgreSQL databases |
| Low | **AdGuard Home** | Block ads on the network |
| Low | **Duplicati** | Cloud backup |

## Next step

[Development Tools](./dev-tools.md) - Useful tools and services for development.
