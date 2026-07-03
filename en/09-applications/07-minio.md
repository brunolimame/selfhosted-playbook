# 09-07 - MinIO

## What is it?

[MinIO](https://min.io) is a high-performance object storage server compatible with the Amazon S3 API. It is the self-hosted alternative to Amazon S3.

Main features:
- 100% S3-compatible API (use existing S3 SDKs and tools)
- Web console for bucket and file management
- High performance (written in Go, ~50 MB per container)
- Erasure coding support for data protection
- Multi-tenant with buckets and access policies
- Compatible with evolution-go, Typebot, Dify and Chatwoot for media upload

## Architecture

```
minio (single container - single node)
    Port 9000 (S3 API)
    Port 9001 (Web console)
    |
    +-- /data (persistent volume)
```

## Why MinIO in the automation ecosystem?

Several applications need S3 storage for media files:

| Application | MinIO Usage |
|-----------|-------------|
| **Evolution Go** | Photos, audios, videos and documents sent/received on WhatsApp |
| **Typebot** | Image and file uploads in chatbot flows |
| **Dify** | Knowledge base documents and user-uploaded files |
| **Chatwoot** | Attachments in support conversations |
| **n8n** | Intermediate file storage in workflows |

## Prerequisites

- Docker and Docker Compose installed on the VM
- Disk with enough space for media files
- Domain for console: `minio.meuservidor.com` (optional)

## Installation

### 1. Access the VM and create directory

```bash
ssh ubuntu@192.168.1.100
mkdir -p ~/minio
cd ~/minio
```

### 2. Create docker-compose.yml

```bash
nano docker-compose.yml
```

```yaml
services:
  minio:
    image: quay.io/minio/minio:latest
    container_name: minio
    restart: unless-stopped
    command: server /data --console-address ":9001"
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - minio_data:/data
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minio-senha-forte-aqui
      MINIO_BROWSER_REDIRECT_URL: https://minio.meuservidor.com
      MINIO_SERVER_URL: https://s3.meuservidor.com
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  minio_data:
```

### 3. Start

```bash
docker compose up -d
```

### 4. Verify

```bash
docker compose ps
docker compose logs -f
```

### 5. Access the console

- **S3 API**: `http://192.168.1.100:9000`
- **Web console**: `http://192.168.1.100:9001`
- **User**: `minioadmin`
- **Password**: `minio-senha-forte-aqui`

## Configure domain on Cloudflare Tunnel

Edit config.yml:

```bash
nano ~/.cloudflared/config.yml
```

Add:

```yaml
  - hostname: minio.meuservidor.com
    service: http://localhost:9001

  - hostname: s3.meuservidor.com
    service: http://localhost:9000
```

Restart:

```bash
sudo systemctl restart cloudflared
```

## Create buckets and access keys

### Via web console

1. Access `http://192.168.1.100:9001`
2. Login with `minioadmin` / `minio-senha-forte-aqui`
3. **Buckets > Create Bucket**:
   - `evolution-media` (WhatsApp media)
   - `typebot` (Typebot uploads)
   - `dify` (knowledge base documents)
   - `chatwoot` (Chatwoot attachments)
4. **Access Keys > Create Access Key**:
   - Note the `Access Key` and `Secret Key`

### Via command line (mc client)

```bash
# Install mc client
docker compose exec minio mc alias set local http://localhost:9000 minioadmin minio-senha-forte-aqui

# Create buckets
docker compose exec minio mc mb local/evolution-media
docker compose exec minio mc mb local/typebot
docker compose exec minio mc mb local/dify
docker compose exec minio mc mb local/chatwoot

# List buckets
docker compose exec minio mc ls local
```

## Configure access policies (optional)

For buckets that need to be public (e.g., WhatsApp profile images):

```bash
docker compose exec minio mc anonymous set download local/evolution-media
```

## Integrate with applications

### Evolution Go

In the evolution-go `.env`:

```env
MINIO_ENABLED=true
MINIO_ENDPOINT=minio:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minio-senha-forte-aqui
MINIO_BUCKET=evolution-media
MINIO_SSL=false
```

### Typebot

In the Typebot `.env`:

```env
S3_ACCESS_KEY=minioadmin
S3_SECRET_KEY=minio-senha-forte-aqui
S3_BUCKET=typebot
S3_ENDPOINT=http://minio:9000
S3_SSL=false
```

### Dify

In the Dify `.env` (at `~/dify/docker/.env`):

```env
STORAGE_TYPE=s3
S3_ENDPOINT=http://minio:9000
S3_REGION=us-east-1
S3_BUCKET_NAME=dify
S3_ACCESS_KEY=minioadmin
S3_SECRET_KEY=minio-senha-forte-aqui
```

### Chatwoot

In the Chatwoot `.env`:

```env
ACTIVE_STORAGE_SERVICE=amazon
STORAGE_S3_ENDPOINT=http://minio:9000
STORAGE_S3_BUCKET=chatwoot
AWS_ACCESS_KEY_ID=minioadmin
AWS_SECRET_ACCESS_KEY=minio-senha-forte-aqui
AWS_REGION=us-east-1
```

**Important**: For containers to communicate with each other, use the service name (`minio`) instead of `localhost` or `192.168.1.100` in the endpoint, as long as they are on the same Docker network.

If the applications are on separate Docker networks, use the VM IP (`192.168.1.100`) in the endpoint.

## Maintenance

### Update

```bash
cd ~/minio
docker compose pull
docker compose up -d
```

### Logs

```bash
docker compose logs -f
```

### Backup

```bash
#!/bin/bash
# ~/minio/backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR=~/backups/minio
mkdir -p $BACKUP_DIR

# Backup data
docker run --rm -v minio_data:/source -v $BACKUP_DIR:/backup alpine tar czf /backup/minio-data-$DATE.tar.gz -C /source .

# Backup configuration (buckets, policies)
docker compose exec minio mc alias set local http://localhost:9000 minioadmin minio-senha-forte-aqui
docker compose exec minio mc admin config get local > $BACKUP_DIR/minio-config-$DATE.txt

echo "Backup completed: $DATE"
```

### Restore

```bash
docker run --rm -v minio_data:/dest -v $BACKUP_DIR:/backup alpine tar xzf /backup/minio-data-20260101.tar.gz -C /dest
docker compose restart
```

### Clean up old files (optional)

```bash
# Script to delete media older than 90 days
docker compose exec minio mc find local/evolution-media --older-than 90d --exec "mc rm {}"
```

## Troubleshooting

| Error | Cause | Solution |
|------|-------|---------|
| `Connection refused` | Container not started | `docker compose logs minio` |
| `Access Denied` | Wrong credentials | Check `MINIO_ROOT_USER`/`MINIO_ROOT_PASSWORD` |
| Bucket already exists | Duplicate name | Use another name |
| S3 incompatible | Wrong endpoint | Use `http://minio:9000` (Docker) or `http://192.168.1.100:9000` (host) |
| Disk full | Too many stored media | Configure retention/cleanup policy |

## Next step

[Security](../07-security.md) - Protect your server.
