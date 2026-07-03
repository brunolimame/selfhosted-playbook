# 09-01 - Odysseus

## What is it?

[Odysseus](https://github.com/pewdiepie-archdaemon/odysseus) is a self-hosted AI workspace created by PewDiePie. It brings together in a single interface:

- AI Chat (local models or via API)
- Autonomous agents with tools (bash, files, web, MCP)
- Deep research with report generation
- Document editor with AI-powered editing
- Email client (IMAP/SMTP) with AI sorting and summarization
- Notes, tasks, and calendar (CalDAV)
- Support for local models via Ollama

## Architecture

```
odysseus (container)
    Port: 7000
    |
    +-- PostgreSQL (persistent data)
    +-- Redis (cache/sessions)
```

## Prerequisites

- Docker and Docker Compose installed on the VM
- At least 2 GB of free RAM
- Git installed: `sudo apt install -y git`

## Installation

### 1. Access the VM

```bash
ssh ubuntu@192.168.1.100
```

### 2. Clone the repository

```bash
cd ~
git clone https://github.com/pewdiepie-archdaemon/odysseus.git
cd odysseus
```

### 3. Configure environment variables

```bash
cp .env.example .env
nano .env
```

Essential variables:

```env
# Server port
ODYSSEUS_PORT=7000

# Secret key for sessions (generate a strong one)
SECRET_KEY=generate-a-random-key-here

# Database
DATABASE_URL=postgresql://odysseus:password@postgres:5432/odysseus
REDIS_URL=redis://redis:6379

# First admin user (created automatically on first start)
FIRST_ADMIN_EMAIL=admin@meuservidor.com
FIRST_ADMIN_PASSWORD=super-strong-password-here

# Public URL (for links in emails)
PUBLIC_URL=https://odysseus.meuservidor.com

# Default model (optional - API key)
# OPENAI_API_KEY=your-key
# ANTHROPIC_API_KEY=your-key
# OLLAMA_BASE_URL=http://host.docker.internal:11434
```

### 4. Start with Docker Compose

```bash
docker compose up -d --build
```

The first time may take a few minutes (image build).

### 5. Verify it is running

```bash
docker compose ps
docker compose logs -f
```

### 6. Get admin password

For installations that generate an automatic password, check the logs:

```bash
docker compose logs | grep -i password
```

If you set `FIRST_ADMIN_PASSWORD` in `.env`, use that password.

## Configure domain in Cloudflare Tunnel

If using Cloudflare Tunnel, edit the config.yml:

```bash
nano ~/.cloudflared/config.yml
```

Add the hostname before the catch-all (404):

```yaml
  - hostname: odysseus.meuservidor.com
    service: http://localhost:7000
```

Restart the tunnel:

```bash
sudo systemctl restart cloudflared
```

## Add AI models

### OpenAI
```env
OPENAI_API_KEY=sk-proj-...
```

### Local models (Ollama)
If you have Ollama running on the host or another server:

```env
OLLAMA_BASE_URL=http://192.168.1.200:11434
```

To install Ollama on the VM itself:

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull llama3.2:3b
```

### Supported providers
- OpenAI / OpenAI-compatible
- Anthropic Claude
- Ollama (local models)
- Google Gemini
- Any OpenAI-compatible API

## Access

- Local: `http://192.168.1.100:7000`
- Public: `https://odysseus.meuservidor.com`

## Maintenance

### Update

```bash
cd ~/odysseus
git pull
docker compose down
docker compose up -d --build
```

### Logs

```bash
docker compose logs -f
```

### Backup

```bash
# Database backup
docker compose exec postgres pg_dump -U odysseus odysseus > ~/backup-odysseus-$(date +%Y%m%d).sql
```

## Next step

[n8n](./02-n8n.md) - Workflow automation.
