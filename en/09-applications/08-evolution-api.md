# 09-08 - Evolution API (Node.js)

## What is it?

The [Evolution API](https://github.com/evolution-foundation/evolution-api) is the Node.js/TypeScript version of the Evolution ecosystem. Unlike [evolution-go](./03-evolution-go.md) (lightweight and focused on WhatsApp via whatsmeow), this version offers:

- **Multi-provider**: Baileys (WhatsApp Web), Meta Cloud API (WhatsApp Business)
- **Future support**: Instagram and Messenger (in development)
- **Web management**: Included admin panel (evolution-manager)
- **Native integrations**: Typebot, Chatwoot, Dify built into the API
- **Multi-database**: PostgreSQL and MySQL
- **More events**: WebSocket, Socket.io, RabbitMQ, Kafka, SQS, NATS, Pusher

## When to use evolution-api vs. evolution-go?

| Feature | evolution-go | evolution-api (Node.js) |
|---------------|-------------|------------------------|
| Performance | High (Go) | Medium (Node.js) |
| Size | ~50 MB | ~200 MB |
| Admin panel | No (via API) | Yes (evolution-manager) |
| Native integrations | Manual webhooks | Typebot, Chatwoot, Dify built-in |
| Instagram/Messenger | No | Future |
| Database | PostgreSQL | PostgreSQL or MySQL |
| Cache | No | Redis |
| Licensing | Yes | Yes |

## Architecture

```
evolution-api (container)      -> Port 8080 (API)
evolution-manager (container)  -> Port 3000 (Web panel)
     |
     +-- PostgreSQL (data)
     +-- Redis (cache/queues)
```

## Prerequisites

- Docker and Docker Compose
- At least 2 GB RAM
- PostgreSQL 15+
- Redis 7+
- Evolution API license (see https://evolutionfoundation.com.br)

## Installation

### 1. Access the VM and create directory

```bash
ssh ubuntu@192.168.1.100
mkdir -p ~/evolution-api
cd ~/evolution-api
```

### 2. Create docker-compose.yml

```bash
nano docker-compose.yml
```

```yaml
services:
  redis:
    image: redis:7-alpine
    container_name: evo-redis
    restart: unless-stopped
    volumes:
      - evo_redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  postgres:
    image: postgres:15-alpine
    container_name: evo-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: evolution
      POSTGRES_PASSWORD: senha-forte-aqui
      POSTGRES_DB: evolution
    volumes:
      - evo_postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U evolution"]
      interval: 10s
      timeout: 5s
      retries: 5

  api:
    image: evoapicloud/evolution-api:latest
    container_name: evolution-api
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - evo_instances:/evolution/instances
    env_file: .env
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy

  manager:
    image: evoapicloud/evolution-manager:latest
    container_name: evolution-manager
    restart: unless-stopped
    ports:
      - "3000:80"
    depends_on:
      - api

volumes:
  evo_redis_data:
  evo_postgres_data:
  evo_instances:
```

### 3. Create .env

```bash
nano .env
```

```env
# === Database ===
DATABASE_PROVIDER=postgresql
DATABASE_URL=postgresql://evolution:senha-forte-aqui@postgres:5432/evolution

# === Redis ===
REDIS_URI=redis://redis:6379
CACHE_TTL=86400

# === Authentication ===
AUTHENTICATION_TYPE=apikey
GLOBAL_API_KEY=seu-uuid-unico-aqui

# === Port ===
PORT=8080

# === Enabled providers ===
PROVIDER_BAILEYS_ENABLED=true
PROVIDER_META_ENABLED=false

# === Webhooks ===
WEBHOOK_GLOBAL_URL=
WEBHOOK_GLOBAL_ENABLED=false

# === Integrations (optional) ===
# TYPEBOT_URL=https://bot.meuservidor.com
# CHATWOOT_URL=https://atendimento.meuservidor.com
# DIFY_URL=https://ia.meuservidor.com
```

### 4. Start

```bash
docker compose up -d
```

### 5. Verify

```bash
docker compose ps
docker compose logs -f api
```

Wait for: `Server is running on port 8080`

### 6. Access the panel

- **API**: `http://192.168.1.100:8080`
- **Manager**: `http://192.168.1.100:3000`

## Configure domain on Cloudflare Tunnel

```bash
nano ~/.cloudflared/config.yml
```

```yaml
  - hostname: api.evolution.meuservidor.com
    service: http://localhost:8080
  - hostname: manager.evolution.meuservidor.com
    service: http://localhost:3000
```

```bash
sudo systemctl restart cloudflared
```

## Key differences from evolution-go

### 1. Native Typebot integration

In evolution-api, Typebot is configured directly on endpoints, without needing n8n:

```bash
curl -X POST http://localhost:8080/typebot/create/meu-whatsapp \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer SUA_API_KEY" \
  -d '{
    "typebot": {
      "url": "https://bot.meuservidor.com",
      "typebot": "id-do-seu-typebot",
      "startSession": true
    }
  }'
```

### 2. Management panel

The evolution-manager provides a web interface for:
- Managing instances (create, delete, connect QR Code)
- Configuring webhooks
- Viewing status
- Managing licenses

### 3. Multi-provider

Supports simultaneously:
- **Baileys**: WhatsApp Web (free, like evolution-go)
- **Meta Cloud API**: Official WhatsApp Business API (requires Meta Business account)

## Next step

Back to [Applications Index](./README.md) or [Multi-channel Chatbot](../10-multichannel-chatbot/README.md).
