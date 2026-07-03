# 09-04 - Typebot

## What is it?

[Typebot](https://typebot.io) is an open-source platform for creating conversational chatbots with visual no-code flows. It is the self-hosted alternative to Landbot, ManyChat, and Chatfuel.

Main features:
- Create conversation flows with draggable blocks
- Support for text, buttons, images, videos, forms, and conditions
- Native integration with **Evolution Go** (WhatsApp)
- Webhooks to connect with n8n and other tools
- Scheduling and analytics
- File upload with S3/MinIO storage
- Multi-language

## Architecture

```
typebot-builder (container)  ->  Port 3001 (flow creation)
typebot-viewer (container)   ->  Port 3002 (bot execution)
     |
     +-- PostgreSQL (data, flows, users)
     +-- (Optional) MinIO/S3 (file upload)
```

## Why Typebot with Evolution Go?

Typebot has native integration with the Evolution ecosystem. The service flow looks like:

```
Client sends message on WhatsApp
     |
evolution-go receives the webhook
     |
Typebot executes the configured flow
     |
     +-- If automatic response -> Typebot replies via evolution-go API
     +-- If human needed -> triggers Chatwoot or n8n
```

## Prerequisites

- Docker and Docker Compose installed on the VM
- At least 1 GB free RAM
- A domain for the builder (e.g. `bot.meuservidor.com`)
- A domain for the viewer (e.g. `viewer.meuservidor.com`)

## Installation

### 1. Access the VM and create directory

```bash
ssh ubuntu@192.168.1.100
mkdir -p ~/typebot
cd ~/typebot
```

### 2. Download configuration files

```bash
wget https://raw.githubusercontent.com/baptisteArno/typebot.io/latest/docker-compose.yml
wget https://raw.githubusercontent.com/baptisteArno/typebot.io/latest/.env.example -O .env
```

### 3. Generate encryption key

```bash
openssl rand -base64 32 | tr -d '\n' ; echo
```

Copy the result to use in `.env`.

### 4. Configure environment variables

```bash
nano .env
```

```env
# Database
DATABASE_URL=postgresql://postgres:typebot@postgres:5432/typebot

# Encryption (use the value generated in the previous step)
ENCRYPTION_SECRET=place-your-32-character-key-here

# Builder URL (publicly accessible)
NEXTAUTH_URL=https://bot.meuservidor.com
NEXT_PUBLIC_VIEWER_URL=https://viewer.meuservidor.com

# Auth providers (at least one required)
# Email (magic link)
NEXT_PUBLIC_SMTP_FROM_EMAIL=noreply@meuservidor.com
SMTP_URL=smtp://user:pass@smtp.yourprovider.com:587
NEXT_PUBLIC_SMTP_NAME=Typebot

# Google OAuth (optional)
# GOOGLE_CLIENT_ID=...
# GOOGLE_CLIENT_SECRET=...

# GitHub OAuth (optional)
# GITHUB_CLIENT_ID=...
# GITHUB_CLIENT_SECRET=...

# Ports (adjust if there is a conflict)
BUILDER_PORT=3001
VIEWER_PORT=3002

# S3 / MinIO for file upload (optional)
# S3_ACCESS_KEY=minioadmin
# S3_SECRET_KEY=minioadmin
# S3_BUCKET=typebot
# S3_ENDPOINT=http://192.168.1.100:9000
# S3_SSL=false
```

### 5. Adjust docker-compose.yml

Edit the downloaded file to adjust the ports:

```bash
nano docker-compose.yml
```

Change the port sections to:

```yaml
  typebot-builder:
    ports:
      - '${BUILDER_PORT}:3000'
    # ...

  typebot-viewer:
    ports:
      - '${VIEWER_PORT}:3000'
    # ...
```

### 6. Start

```bash
docker compose up -d
```

### 7. Verify

```bash
docker compose ps
docker compose logs -f
```

## Configure domain in Cloudflare Tunnel

Edit config.yml:

```bash
nano ~/.cloudflared/config.yml
```

Add before the catch-all:

```yaml
  - hostname: bot.meuservidor.com
    service: http://localhost:3001

  - hostname: viewer.meuservidor.com
    service: http://localhost:3002
```

Restart:

```bash
sudo systemctl restart cloudflared
```

## Configure media files with MinIO (optional)

If you have MinIO installed ([guide](./07-minio.md)), create a bucket:

```bash
docker compose exec minio mc alias set local http://localhost:9000 minioadmin minioadmin
docker compose exec minio mc mb local/typebot
```

And in Typebot's `.env`:
```env
S3_ACCESS_KEY=minioadmin
S3_SECRET_KEY=minioadmin
S3_BUCKET=typebot
S3_ENDPOINT=http://minio:9000
S3_SSL=false
```

Restart Typebot after changing:
```bash
docker compose down && docker compose up -d
```

## Integrate with Evolution Go

To connect Typebot to evolution-go and reply to WhatsApp automatically:

### 1. Create the bot in Typebot

1. Access `https://bot.meuservidor.com`
2. Create your account
3. Create a new Typebot
4. Build the desired conversation flow
5. Publish the bot and copy the **public ID** (in the URL: `/typebots/[ID]/...`)

### 2. Configure webhook in evolution-go

In evolution-go, create the instance with Typebot integration:

```bash
curl -X POST http://192.168.1.100:4000/typebot/create/meu-whatsapp \
  -H "Content-Type: application/json" \
  -H "apiKey: YOUR_GLOBAL_API_KEY" \
  -d '{
    "typebot": {
      "url": "https://viewer.meuservidor.com",
      "name": "meu-bot-de-atendimento",
      "typebotId": "YOUR-TYPEBOT-ID",
      "startSession": true,
      "trigger": {
        "type": "keyword",
        "value": "!bot"
      }
    }
  }'
```

Now, when someone sends "!bot" on WhatsApp, Typebot will start the flow automatically.

### Pre-defined variables

evolution-go automatically sends to Typebot:
- `remoteJid` - WhatsApp contact ID
- `pushName` - Contact name
- `instanceName` - Instance name
- `serverUrl` - evolution-go server URL
- `apiKey` - API key

## Maintenance

### Update

```bash
cd ~/typebot
docker compose pull
docker compose down
docker compose up -d
```

### Logs

```bash
docker compose logs -f typebot-builder
docker compose logs -f typebot-viewer
```

### Backup

```bash
# Database backup
docker compose exec postgres pg_dump -U postgres typebot > ~/backups/typebot-$(date +%Y%m%d).sql
```

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| `ENCRYPTION_SECRET` invalid | Key too short | Must be exactly 32 characters |
| `NEXTAUTH_URL` not configured | Builder URL missing | Set `NEXTAUTH_URL` in `.env` |
| Viewer does not load | `NEXT_PUBLIC_VIEWER_URL` wrong | Check the viewer public URL |
| Error sending email | SMTP not configured | Configure SMTP or use OAuth |
| File upload fails | S3/MinIO not configured | Configure S3 or disable upload |

## Next step

[Chatwoot](./05-chatwoot.md) - Helpdesk for human support.
