# 09-05 - Chatwoot

## What is it?

[Chatwoot](https://www.chatwoot.com) is an open-source customer support platform (helpdesk). It is the self-hosted alternative to Intercom, Zendesk, and Freshdesk.

Main features:
- Unified inbox (WhatsApp, Email, Webchat, Telegram, etc.)
- Multi-agent support with conversation assignment
- Pre-defined responses (macros/canned responses)
- Internal notes and mentions between agents
- Reports and analytics
- API and webhooks for integrations
- Rule automation (auto-assign, labels, etc.)

## Architecture

```
chatwoot-web (container)    -> Port 3000 (web interface + API)
chatwoot-worker (container) -> Sidekiq (background jobs)
     |
     +-- PostgreSQL (conversations, contacts, agents)
     +-- Redis (cache, job queues)
```

## Why Chatwoot with Evolution Go and Typebot?

Chatwoot is the human link in the customer service automation chain. The complete flow:

```
Client sends message on WhatsApp
     |
evolution-go receives the webhook
     |
Typebot tries to resolve automatically
     |
     +-- If client needs human -> evolution-go forwards to Chatwoot
     |
Chatwoot displays to the available agent
     |
Agent replies -> Chatwoot sends to evolution-go -> evolution-go delivers on WhatsApp
```

## Prerequisites

- Docker and Docker Compose installed on the VM
- At least 2 GB free RAM (Chatwoot is heavy)
- Configured domain: `atendimento.meuservidor.com`
- SMTP account for transactional emails (optional for initial setup)

## Installation

### 1. Access the VM and create directory

```bash
ssh ubuntu@192.168.1.100
mkdir -p ~/chatwoot
cd ~/chatwoot
```

### 2. Download the production docker-compose

```bash
wget -O docker-compose.yml https://raw.githubusercontent.com/chatwoot/chatwoot/develop/docker-compose.production.yaml
```

### 3. Create .env file

```bash
nano .env
```

```env
# === General ===
INSTALLATION_NAME=My Support

# === Port ===
PORT=3000

# === PostgreSQL ===
POSTGRES_USER=chatwoot
POSTGRES_PASSWORD=strong-password-here
POSTGRES_DATABASE=chatwoot

# === Redis ===
REDIS_PASSWORD=redis-password-here

# === Base URL (public domain) ===
FRONTEND_URL=https://atendimento.meuservidor.com

# === Secret key (generate with: openssl rand -hex 64) ===
SECRET_KEY_BASE=place-a-64-character-hex-key-here

# === Encryption key (generate with: openssl rand -hex 32) ===
ENCRYPTION_PRIMARY_KEY=place-a-32-character-hex-key-here

# === SMTP (for sending notification emails) ===
# SMTP_ADDRESS=smtp.gmail.com
# SMTP_PORT=587
# SMTP_USERNAME=seu-email@gmail.com
# SMTP_PASSWORD=sua-senha
# SMTP_AUTH_METHOD=plain
# SMTP_ENABLE_STARTTLS_AUTO=true
# SMTP_DOMAIN=gmail.com

# === Storage (optional - use MinIO) ===
# ACTIVE_STORAGE_SERVICE=local
# STORAGE_DIR=/app/storage

# === Default language ===
DEFAULT_LOCALE=en
```

### 4. Generate the keys

```bash
echo "SECRET_KEY_BASE=$(openssl rand -hex 64)"
echo "ENCRYPTION_PRIMARY_KEY=$(openssl rand -hex 32)"
```

Copy the values into `.env`.

### 5. Adjust docker-compose.yml

Edit to use the correct ports:

```bash
nano docker-compose.yml
```

In the `chatwoot` service section, adjust the ports:

```yaml
  chatwoot:
    ports:
      - "${PORT}:3000"
```

And in `depends_on`, change to use the correct service names.

### 6. Prepare the database

```bash
docker compose run --rm chatwoot bundle exec rails db:chatwoot_prepare
```

This command:
- Creates the tables in PostgreSQL
- Runs migrations
- Seeds initial data

### 7. Start

```bash
docker compose up -d
```

### 8. Verify

```bash
docker compose ps
docker compose logs -f chatwoot
```

Wait until you see: `Listening on http://0.0.0.0:3000`

### 9. Create admin account

Access `http://192.168.1.100:3000` and create the administrator account.

## Configure domain in Cloudflare Tunnel

Edit config.yml:

```bash
nano ~/.cloudflared/config.yml
```

Add:

```yaml
  - hostname: atendimento.meuservidor.com
    service: http://localhost:3000
```

Restart:

```bash
sudo systemctl restart cloudflared
```

## Integrate with Evolution Go

### 1. In Chatwoot

1. Go to **Settings > Inboxes > Add Inbox**
2. Select **Evolution API**
3. Configure:
   - **Name**: Business WhatsApp
   - **Evolution API URL**: `http://192.168.1.100:4000`
   - **Global API Key**: (your evolution-go key)
   - **Instance Name**: (instance name in evolution-go)
4. Click **Create Inbox**

### 2. In evolution-go

Configure the webhook to send messages to Chatwoot:

```bash
curl -X POST http://192.168.1.100:4000/webhook/create/meu-whatsapp \
  -H "Content-Type: application/json" \
  -H "apiKey: YOUR_GLOBAL_API_KEY" \
  -d '{
    "webhook": {
      "url": "https://atendimento.meuservidor.com/webhooks/evolution/YOUR-WEBHOOK-TOKEN",
      "events": ["message.upsert", "message.update", "connection.update"]
    }
  }'
```

The webhook token is automatically generated when creating the inbox in Chatwoot.

## Complete flow (Typebot + Chatwoot)

To configure automatic routing where Typebot tries to resolve and Chatwoot takes over when necessary:

1. **Typebot**: create a flow with a "Set Variable" block setting `transferir=false`
2. **Typebot**: if the client asks for "agent" or "human", change `transferir=true`
3. **Typebot**: "Webhook" block sends to n8n with the `transferir` flag
4. **n8n**: if `transferir=true`, calls the Chatwoot API to create a conversation and notify the agent
5. **n8n**: sends a message on WhatsApp via evolution-go: "You will be assisted shortly"

Example n8n webhook to create a conversation in Chatwoot:

```bash
curl -X POST https://atendimento.meuservidor.com/api/v1/accounts/1/conversations \
  -H "Content-Type: application/json" \
  -H "api_access_token: YOUR-CHATWOOT-TOKEN" \
  -d '{
    "source_id": "5511999999999@s.whatsapp.net",
    "inbox_id": 1,
    "contact_id": 1,
    "status": "pending"
  }'
```

## Maintenance

### Update

```bash
cd ~/chatwoot
docker compose pull
docker compose down
docker compose up -d
```

After updating, run migrations if needed:

```bash
docker compose run --rm chatwoot bundle exec rails db:migrate
```

### Logs

```bash
docker compose logs -f chatwoot
docker compose logs -f chatwoot-worker
```

### Backup

```bash
#!/bin/bash
# ~/chatwoot/backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR=~/backups/chatwoot
mkdir -p $BACKUP_DIR

# Database backup
docker compose exec -T postgres pg_dump -U chatwoot chatwoot > $BACKUP_DIR/chatwoot-db-$DATE.sql
gzip $BACKUP_DIR/chatwoot-db-$DATE.sql

# File backup
docker run --rm -v chatwoot_data:/source -v $BACKUP_DIR:/backup alpine tar czf /backup/chatwoot-storage-$DATE.tar.gz -C /source .

echo "Backup completed: $DATE"
```

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| `500 Internal Server Error` | Invalid SECRET_KEY_BASE | Generate new key with `openssl rand -hex 64` |
| PostgreSQL connection refused | Postgres did not start | `docker compose logs postgres` |
| Emails not sending | SMTP not configured | Set up SMTP in `.env` and restart |
| Evolution inbox not connecting | Wrong URL or API Key | Check evolution-go endpoint |
| White screen on login | Wrong `FRONTEND_URL` | Use exact URL with https |

## Next step

[Dify](./06-dify.md) - AI with knowledge base (RAG).
