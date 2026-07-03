# 09-06 - Dify

## What is it?

[Dify](https://dify.ai) is an open-source platform for building generative AI applications. It allows you to create intelligent chatbots with your own knowledge base (RAG), AI workflows, and autonomous agents.

Main features:
- **RAG (Retrieval-Augmented Generation)**: Connect documents (PDF, TXT, HTML, etc.) and ask questions about them
- **Chatbot with context**: Upload documents as a knowledge base
- **AI Workflow**: Create pipelines with visual nodes (LLM, tools, logic)
- **Agents**: Autonomous agents with tools (web search, API, calculator)
- **Complete API**: Integrate with any system (evolution-go, n8n, Typebot)
- **Multi-model**: OpenAI, Anthropic, Ollama (local), open-source models

## Architecture

```
dify (multi-containers)
    |
    +-- api (Flask/Python backend)      -> Port 5001
    +-- worker (queue processing)
    +-- web (Next.js frontend)          -> Port 3000
    +-- nginx (internal reverse proxy)  -> Port 80
    |
    +-- PostgreSQL (application data)
    +-- Redis (cache/queues)
    +-- Weaviate (vector database for RAG)
```

## Why Dify with Evolution Go?

Dify provides intelligence for customer service:

```
Client asks: "What are your business hours?"
     |
evolution-go receives the message
     |
n8n forwards to Dify
     |
Dify queries the knowledge base (PDF with FAQ)
     |
LLM responds with context from the documents
     |
n8n returns the response to evolution-go
     |
evolution-go sends the message on WhatsApp
```

## Prerequisites

- Docker and Docker Compose 2.24+ installed on the VM
- At least 4 GB free RAM (Dify + Weaviate consume memory)
- CPU 2+ cores
- Configured domain: `ia.meuservidor.com`
- (Optional) API key from an LLM provider (OpenAI, Anthropic)

## Installation

### 1. Access the VM and clone the repository

```bash
ssh ubuntu@192.168.1.100
cd ~
git clone https://github.com/langgenius/dify.git
cd dify
```

### 2. Configure environment variables

```bash
cd docker
cp .env.example .env
nano .env
```

Essential variables:

```env
# Deployment mode
DEPLOY_ENV=PRODUCTION

# Ports (avoid conflict with other apps)
EXPOSE_NGINX_PORT=80
EXPOSE_NGINX_SSL_PORT=443

# Secret key (generate with: openssl rand -hex 32)
SECRET_KEY=your-secret-key-here

# Database
DB_USERNAME=postgres
DB_PASSWORD=dify-password
DB_DATABASE=dify
DB_PORT=5432

# Redis
REDIS_PASSWORD=dify-redis-password

# Weaviate (vector store)
WEAVIATE_AUTHENTICATION_ANONYMOUS_ACCESS_ENABLED=false
WEAVIATE_AUTHENTICATION_APIKEY_ENABLED=true
WEAVIATE_AUTHENTICATION_APIKEY_ALLOWED_KEYS=dify-weaviate-key
WEAVIATE_AUTHENTICATION_APIKEY_USERS=admin

# Admin initialization
INIT_PASSWORD=super-strong-admin-password
INIT_EMAIL=admin@meuservidor.com
```

### 3. Adjust docker-compose.yml (ports)

If Coolify is already using port 80, edit the `docker-compose.yml`:

```bash
nano docker-compose.yml
```

Change the nginx ports:

```yaml
  nginx:
    ports:
      - "8080:80"
```

This will make Dify run on `http://localhost:8080`.

### 4. Start

```bash
cd ~/dify/docker
docker compose up -d
```

The first time may take a while (downloading multiple images).

### 5. Verify

```bash
docker compose ps
```

The following should be running: `api`, `worker`, `web`, `nginx`, `db`, `redis`, `weaviate`, `ssrf_proxy`, `sandbox`, `plugin_daemon`.

### 6. Access and configure admin

Access `http://192.168.1.100:8080/install` and follow the setup flow:
1. Set admin email and password (or use `INIT_EMAIL`/`INIT_PASSWORD` from `.env`)
2. Select the default LLM provider (OpenAI, Anthropic, or configure later)

## Configure domain in Cloudflare Tunnel

Edit config.yml:

```bash
nano ~/.cloudflared/config.yml
```

Add:

```yaml
  - hostname: ia.meuservidor.com
    service: http://localhost:8080
```

Restart:

```bash
sudo systemctl restart cloudflared
```

## Configure LLM provider

Go to **Settings > Model Provider** and add at least one provider:

### OpenAI
```env
OPENAI_API_KEY=sk-proj-your-token
```

### Ollama (local models)
If you have Ollama installed on the VM:
```env
OLLAMA_BASE_URL=http://192.168.1.100:11434
```

Then go to **Settings > Model Provider > Ollama** and configure.

## Create a knowledge base

To use RAG with your documents:

1. Go to **Knowledge > Create Knowledge**
2. Select **Upload Files**
3. Upload PDFs, TXTs, or other documents (e.g. FAQ, manual, catalog)
4. Choose the indexing method (recommended: **High Quality** with Weaviate)
5. Wait for processing
6. Go to **Studio > Create App > Chatbot**
7. In **Context**, select the created knowledge base
8. Publish the app

## Integrate with n8n + Evolution Go

To connect Dify, n8n, and evolution-go:

### In Dify

1. Create a **Chatbot** type app with a knowledge base
2. Go to **API Access** and copy the **API Key** and **API URL**
3. Copy the **endpoint** (e.g. `https://ia.meuservidor.com/v1/chat-messages`)

### In n8n

Create a workflow with:
1. **Webhook node**: receives message from evolution-go
2. **HTTP Request node**: calls Dify API
3. **HTTP Request node**: sends response to evolution-go

Example Dify API call:

```bash
curl -X POST https://ia.meuservidor.com/v1/chat-messages \
  -H "Authorization: Bearer YOUR-DIFY-API-KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "inputs": {},
    "query": "What are your business hours?",
    "response_mode": "blocking",
    "user": "5511999999999"
  }'
```

### With Typebot + Dify

Typebot can consume Dify via the **HTTP Request** block:

1. Create a Typebot with an FAQ flow
2. Add an **HTTP Request** block configured to call Dify
3. Use Typebot variables to pass the client's question
4. Display the Dify response in the chat

## Configure S3/MinIO storage

To save files and images uploaded by users:

```env
STORAGE_TYPE=s3
S3_ENDPOINT=http://minio:9000
S3_REGION=us-east-1
S3_BUCKET_NAME=dify
S3_ACCESS_KEY=minioadmin
S3_SECRET_KEY=minioadmin
```

See the [MinIO guide](./07-minio.md) to configure the bucket.

## Maintenance

### Update

```bash
cd ~/dify
git pull
cd docker
docker compose down
docker compose up -d
```

### Logs

```bash
cd ~/dify/docker
docker compose logs -f api
docker compose logs -f web
```

### Backup

```bash
#!/bin/bash
# ~/dify/backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR=~/backups/dify
mkdir -p $BACKUP_DIR

# Database backup
docker compose exec -T db pg_dump -U postgres dify > $BACKUP_DIR/dify-db-$DATE.sql
gzip $BACKUP_DIR/dify-db-$DATE.sql

# File backup (uploads, knowledge base)
docker run --rm -v dify_storage:/source -v $BACKUP_DIR:/backup alpine tar czf /backup/dify-storage-$DATE.tar.gz -C /source .

echo "Backup completed: $DATE"
```

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| Weaviate connection refused | Weaviate did not start | `docker compose logs weaviate` |
| LLM response timeout | Invalid API key or exceeded limit | Check provider balance |
| Empty knowledge base | Documents not indexed | Check worker logs |
| 502 Bad Gateway | api did not respond | `docker compose logs api` |
| Low memory | Weaviate + API + Worker consume a lot | Increase VM RAM to 6GB+ |

## Next step

[MinIO](./07-minio.md) - S3 storage for media files.
