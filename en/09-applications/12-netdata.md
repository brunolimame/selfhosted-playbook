# 09-12 - Netdata

## What is it?

[Netdata](https://www.netdata.cloud) is a **real-time** monitoring system that collects thousands of metrics per second and displays them in interactive charts. Zero configuration: upon starting, it automatically detects and monitors all services and containers.

Main features:
- Metrics every 1 second (not every 60s like traditional tools)
- Auto-discovery of services: PostgreSQL, Redis, Nginx, Docker, and 200+ others
- Interactive web dashboard with zoomable and draggable charts
- Intelligent alerts configured automatically
- Lightweight: ~100-200 MB RAM, ~1% CPU
- Historical metrics support (configurable)

## Why Netdata in this project?

Uptime Kuma tells you **if** the service is up. Netdata tells you **why** it went down:

| Scenario | Uptime Kuma | Netdata |
|---------|-------------|---------|
| "Site is down" | Alert: down | - |
| "RAM ran out" | - | Memory graph showing the spike |
| "CPU at 100%" | - | Which process consumed it |
| "Disk full" | - | Exactly which directory |
| "Port closed" | Alert: timeout | System logs |

## Architecture

```
netdata (container)
    Port: 19999 (dashboard)
    |
    +-- Accesses /proc, /sys from host (read-only)
    +-- Accesses /var/run/docker.sock (containers)
    +-- Monitors automatically: CPU, RAM, disk, network, Docker, PostgreSQL, Redis...
```

## Prerequisites

- Docker installed on the VM
- At least 512 MB free RAM (Netdata uses ~100-200 MB)
- Special container privileges to access system metrics

## Installation

### 1. Access the VM

```bash
ssh ubuntu@192.168.1.100
```

### 2. Create directory

```bash
mkdir -p ~/netdata
cd ~/netdata
```

### 3. Create docker-compose.yml

```bash
nano docker-compose.yml
```

```yaml
services:
  netdata:
    image: netdata/netdata:stable
    container_name: netdata
    hostname: ubuntu-vm
    restart: unless-stopped
    pid: host
    network_mode: host
    cap_add:
      - SYS_PTRACE
      - SYS_ADMIN
    security_opt:
      - apparmor:unconfined
    volumes:
      - netdata_config:/etc/netdata
      - netdata_lib:/var/lib/netdata
      - netdata_cache:/var/cache/netdata
      - /:/host/root:ro,rslave
      - /etc/passwd:/host/etc/passwd:ro
      - /etc/group:/host/etc/group:ro
      - /etc/localtime:/etc/localtime:ro
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /etc/os-release:/host/etc/os-release:ro
      - /var/log:/host/var/log:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro

volumes:
  netdata_config:
  netdata_lib:
  netdata_cache:
```

> **Note**: `network_mode: host` makes Netdata use port 19999 directly on the VM IP, without port mapping.

### 4. Start

```bash
docker compose up -d
```

### 5. Verify

```bash
docker compose ps
docker compose logs -f
```

## Access

- **Local**: `http://192.168.1.100:19999`
- No login required - the dashboard opens directly with all metrics.

## Configure domain on Cloudflare Tunnel

```bash
nano ~/.cloudflared/config.yml
```

```yaml
  - hostname: netdata.meuservidor.com
    service: http://localhost:19999
```

```bash
sudo systemctl restart cloudflared
```

## What Netdata monitors automatically

### System
- CPU (per core, per process)
- RAM (used, buffer, cache, swap)
- Disk (I/O, usage, inodes)
- Network (interface by interface, protocols)
- Processes (top 10 by CPU/RAM)

### Docker
- Each container: CPU, RAM, network, disk
- Centralized logs

### Services (auto-detected)
- **PostgreSQL**: queries, connections, locks, cache hit ratio
- **Redis**: hits, misses, memory, connections
- **Nginx**: requests, connections, errors (if present)
- **MySQL/MariaDB**: queries, threads, buffer pool

## Useful pre-configured alerts

Netdata comes with intelligent alerts:

| Alert | Trigger | Suggested action |
|--------|---------|---------------|
| RAM > 80% | High memory usage | Check containers with highest consumption |
| CPU > 90% | Overloaded processor | Identify process on dashboard |
| Disk > 85% | Almost full | Run `docker system prune` |
| Swap > 50% | Insufficient RAM | Increase VM RAM |
| PostgreSQL connections > 100 | Too many connections | Check pooling or app with leak |

To configure notifications:
1. Access `http://192.168.1.100:19999`
2. Click on **Alerts > Notifications**
3. Add Telegram, Discord, Email or Slack

## Customize data retention

By default Netdata stores ~2 hours of metrics in memory. To increase:

```bash
nano docker-compose.yml
```

Add in `environment`:

```yaml
    environment:
      - NETDATA_PAGE_CACHE_SIZE=32
      - NETDATA_DBENGINE_SIZE=256
```

This increases retention to several days.

Or edit the configuration file:

```bash
docker compose exec netdata /etc/netdata/edit-config netdata.conf
```

```ini
[global]
    page cache size = 32
    dbengine multihost disk space = 256
```

## Integrations with other tools

### Grafana (if available)
Netdata can export metrics to Prometheus, which Grafana consumes:

```bash
# In the configuration file
docker compose exec netdata /etc/netdata/edit-config go.d/prometheus.conf
```

### n8n
n8n can query the Netdata API to make decisions:

```bash
# Get current CPU usage
curl -s http://192.168.1.100:19999/api/v1/data?chart=system.cpu | jq '.result[0].value[1]'
```

## Maintenance

### Update

```bash
cd ~/netdata
docker compose pull
docker compose up -d
```

### Check used space

```bash
docker run --rm -v netdata_cache:/source alpine du -sh /source
```

### Logs

```bash
docker compose logs -f --tail 100
```

## Troubleshooting

| Error | Cause | Solution |
|------|-------|---------|
| Dashboard not loading | Port not accessible | `docker compose ps` to check if it started |
| "Permission denied" | Missing privileges | Check `cap_add` and `security_opt` |
| Docker metrics empty | Socket not mounted | Check `/var/run/docker.sock` in volumes |
| High RAM usage | DBENGINE too large | Reduce `dbengine multihost disk space` |
| Charts with no data | Just started | Wait 30s for first collection |

## Next step

Back to [Applications Index](./README.md).
