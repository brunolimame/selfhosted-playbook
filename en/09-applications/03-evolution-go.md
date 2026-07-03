# 09-03 - Evolution Go

## What is it?

[Evolution Go](https://github.com/evolution-foundation/evolution-go) is a high-performance API for WhatsApp integration, written in Go. It is part of the Evolution Foundation ecosystem.

Main features:
- Send and receive text, media, and document messages
- Manage multiple instances (numbers) simultaneously
- Webhooks for real-time events
- Support for RabbitMQ, NATS, and WebSocket queues
- Media storage in MinIO/S3
- Integrated Swagger documentation
- QR Code for device pairing

## Architecture

```
evolution-go (container)
    Port: 4000
    |
    +-- PostgreSQL (authentication, users, messages)
    +-- (Optional) RabbitMQ for event queues
    +-- (Optional) MinIO for media storage
```

## Licensing

Evolution Go requires a license to function. On first start:
1. Access `http://localhost:4000/manager/`
2. Enter your `GLOBAL_API_KEY`
3. Complete the activation flow

To obtain a license, visit: https://evolutionfoundation.com.br

## Prerequisites

- Docker and Docker Compose installed on the VM
- At least 512 MB free RAM
- PostgreSQL 15+
- A valid Evolution API license

## Installation

### 1. Access the VM

```bash
ssh ubuntu@192.168.1.100
```

### 2. Create project directory

```bash
mkdir -p ~/evolution-go
cd ~/evolution-go
```

### 3. Create .env file

```bash
nano .env
```

```env
# Server port
SERVER_PORT=4000

# Client identifier
CLIENT_NAME=evolution

# Global API key (replace with a strong UUID!)
GLOBAL_API_KEY=your-unique-uuid-here

# PostgreSQL
POSTGRES_AUTH_DB=postgresql://postgres:password@postgres:5432/evogo_auth?sslmode=disable
POSTGRES_USERS_DB=postgresql://postgres:password@postgres:5432/evogo_users?sslmode=disable

# Message storage (disabled for safety until configured)
DATABASE_SAVE_MESSAGES=false

# Logs
WADEBUG=INFO
LOGTYPE=console

# Reconnect instances on startup
CONNECT_ON_STARTUP=false

# Webhooks (configure later)
# WEBHOOK_URL=https://n8n.meuservidor.com/webhook/whatsapp
```

### 4. Create docker-compose.yml

```bash
nano docker-compose.yml
```

```yaml
services:
  postgres:
    image: postgres:15-alpine
    container_name: evogo-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - evogo_postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  evolution-go:
    image: evoapicloud/evolution-go:latest
    container_name: evolution-go
    restart: unless-stopped
    ports:
      - "4000:4000"
    environment:
      SERVER_PORT: ${SERVER_PORT}
      CLIENT_NAME: ${CLIENT_NAME}
      GLOBAL_API_KEY: ${GLOBAL_API_KEY}
      POSTGRES_AUTH_DB: ${POSTGRES_AUTH_DB}
      POSTGRES_USERS_DB: ${POSTGRES_USERS_DB}
      DATABASE_SAVE_MESSAGES: ${DATABASE_SAVE_MESSAGES}
      WADEBUG: ${WADEBUG}
      LOGTYPE: ${LOGTYPE}
      CONNECT_ON_STARTUP: ${CONNECT_ON_STARTUP}
    volumes:
      - evogo_data:/app/dbdata
      - evogo_logs:/app/logs
    depends_on:
      postgres:
        condition: service_healthy

volumes:
  evogo_postgres_data:
  evogo_data:
  evogo_logs:
```

### 5. Initialize the databases

Evolution Go expects the `evogo_auth` and `evogo_users` databases to exist. Create them in PostgreSQL before starting:

```bash
# Run inside the PostgreSQL container
docker compose exec postgres psql -U postgres -c "CREATE DATABASE evogo_auth;"
docker compose exec postgres psql -U postgres -c "CREATE DATABASE evogo_users;"
```

### 6. Start

```bash
cd ~/evolution-go
docker compose up -d
```

### 7. Verify

```bash
docker compose ps
docker compose logs -f evolution-go
```

## Configure domain in Cloudflare Tunnel

Edit config.yml:

```bash
nano ~/.cloudflared/config.yml
```

Add:

```yaml
  - hostname: whatsapp.meuservidor.com
    service: http://localhost:4000
```

Restart:

```bash
sudo systemctl restart cloudflared
```

## First use

### 1. Activate license

Access `http://192.168.1.100:4000/manager/` and follow the activation flow with your `GLOBAL_API_KEY`.

### 2. Create an instance

```bash
curl -X POST http://192.168.1.100:4000/instance/create \
  -H "Content-Type: application/json" \
  -H "apiKey: YOUR_GLOBAL_API_KEY" \
  -d '{
    "instanceName": "meu-whatsapp",
    "qrcode": true
  }'
```

### 3. Get QR Code

```bash
curl -X GET http://192.168.1.100:4000/instance/qrcode \
  -H "apiKey: YOUR_GLOBAL_API_KEY" \
  -d '{"instanceName": "meu-whatsapp"}'
```

Access the returned URL or open the QR Code image in your browser.
Scan it with WhatsApp on your phone: **Linked Devices > Link a Device**.

### 4. Send a message

```bash
curl -X POST http://192.168.1.100:4000/message/sendText \
  -H "Content-Type: application/json" \
  -H "apiKey: YOUR_GLOBAL_API_KEY" \
  -d '{
    "number": "5511999999999",
    "options": {
      "delay": 1000,
      "presence": "composing"
    },
    "textMessage": {
      "text": "Hello from Evolution Go!"
    }
  }'
```

## Advanced configuration

### With RabbitMQ (for event queues)

```yaml
  rabbitmq:
    image: rabbitmq:3.13-alpine
    container_name: evogo-rabbitmq
    restart: unless-stopped
    volumes:
      - evogo_rabbitmq_data:/var/lib/rabbitmq
```

Add to `.env`:
```env
AMQP_ENABLED=true
AMQP_URL=amqp://guest:guest@rabbitmq:5672
AMQP_GLOBAL_EXCHANGE=true
AMQP_GLOBAL_EVENTS=presence.update,message.upsert,message.update,connection.update
```

### With MinIO (for media storage)

```yaml
  minio:
    image: minio/minio:latest
    container_name: evogo-minio
    restart: unless-stopped
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    volumes:
      - evogo_minio_data:/data
```

Add to `.env`:
```env
MINIO_ENABLED=true
MINIO_ENDPOINT=minio:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_BUCKET=evolution-media
```

## Integration with n8n

n8n can consume Evolution Go events via webhook:

1. In Evolution Go, configure the webhook in `.env`:
```env
WEBHOOK_URL=https://n8n.meuservidor.com/webhook/whatsapp
```

2. In n8n, create a workflow with a **Webhook** node configured for `/webhook/whatsapp`

3. Use the **HTTP Request** node in n8n to send messages via Evolution Go.

## Maintenance

### Update

```bash
cd ~/evolution-go
docker compose pull
docker compose up -d
```

### Logs

```bash
docker compose logs -f evolution-go
```

### Backup

```bash
#!/bin/bash
# ~/evolution-go/backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR=~/backups/evolution-go
mkdir -p $BACKUP_DIR

# Database backups
docker compose exec -T postgres pg_dump -U postgres evogo_auth > $BACKUP_DIR/evogo-auth-$DATE.sql
docker compose exec -T postgres pg_dump -U postgres evogo_users > $BACKUP_DIR/evogo-users-$DATE.sql

gzip $BACKUP_DIR/*.sql

echo "Backup completed: $DATE"
```

### List instances

```bash
curl -X GET http://192.168.1.100:4000/instance/fetchInstances \
  -H "apiKey: YOUR_GLOBAL_API_KEY"
```

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| `License not activated` | License not activated | Access `/manager/` and activate |
| `QR Code expired` | Time exceeded | Generate a new QR Code |
| `Connection refused` | Number temporarily banned | Wait 24h or change number |
| `ECONNREFUSED postgres` | PostgreSQL did not start | `docker compose logs postgres` |
| `401 Unauthorized` | Invalid API Key | Check `GLOBAL_API_KEY` in `.env` |

## Next step

[Security](../07-security.md) - Protect your server.
