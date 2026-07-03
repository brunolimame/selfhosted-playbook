# 09 - Advanced Applications

Detailed installation guide for advanced applications that run on Coolify (or directly on the VM).

## Index

| Page | Description | Port |
|------|-------------|------|
| [01 - Odysseus](./01-odysseus.md) | Self-hosted AI workspace (chat, agents, email, documents) | 7000 |
| [02 - n8n](./02-n8n.md) | Workflow automation (Zapier alternative) | 5678 |
| [03 - Evolution Go](./03-evolution-go.md) | WhatsApp API in Go (evolution-api) | 4000 |
| [04 - Typebot](./04-typebot.md) | Visual chatbot with no-code flows (native evolution integration) | 3001 / 3002 |
| [05 - Chatwoot](./05-chatwoot.md) | Multi-agent helpdesk/CRM | 3000 |
| [06 - Dify](./06-dify.md) | AI with RAG and knowledge base (LLM + documents) | 8080 |
| [07 - MinIO](./07-minio.md) | S3 storage for media (compatible with all apps) | 9000 / 9001 |
| [08 - Evolution API (Node.js)](./08-evolution-api.md) | Full Node.js API (multi-provider, web panel, native integrations) | 8080 |
| [09 - Graphify](./09-graphify.md) | Knowledge graph for AI assistants (codebase -> query) | 8080 |
| [10 - pgAdmin](./10-pgadmin.md) | Web administration for PostgreSQL | 5050 |
| [11 - Uptime Kuma](./11-uptime-kuma.md) | Uptime monitoring with status page | 3001 |
| [12 - Netdata](./12-netdata.md) | Real-time monitoring (CPU, RAM, disk, Docker) | 19999 |

## Installation Method

The applications will be installed via Docker Compose directly on the VM (outside Coolify for now), since they have complex dependencies (PostgreSQL, Redis, etc.) that Coolify may not manage properly.

The general workflow is:
1. SSH into the VM
2. Create a directory for the application
3. Configure `docker-compose.yml` and `.env`
4. Start with `docker compose up -d`
5. Configure domain in Cloudflare Tunnel
6. Test access

## Template for New Applications

To add documentation for a new application, use the template at [`ADD_APPLICATION.md`](../ADD_APPLICATION.md). It contains specific instructions for LLMs, including structure, checklist, and the complete markdown template.

## Recommended Architecture for Customer Service Automation

```
Client's WhatsApp
     |
evolution-go (WhatsApp connection)
     |
     +-- Typebot (automatic chatbot - FAQ, data capture)
     |       |
     |       +-- Dify (AI with knowledge base - intelligent responses)
     |
     +-- Chatwoot (human support - agents, queue, history)
     |
n8n (orchestration between all systems)
     |
MinIO (media storage - photos, audio, documents)
```

## Recommended Monitoring Architecture

```
Uptime Kuma (know IF it went down)
    |
Netdata (know WHY it went down)
    |
pgAdmin (investigate database)
```

## Next step

[Odysseus](./01-odysseus.md) - Advanced AI workspace, or go directly to:

- [Typebot](./04-typebot.md) - Chatbot for automatic customer service
- [Chatwoot](./05-chatwoot.md) - Helpdesk for human support
- [Dify](./06-dify.md) - AI with knowledge base
- [MinIO](./07-minio.md) - S3 storage for all apps
- [Uptime Kuma](./11-uptime-kuma.md) - Uptime monitoring
- [Netdata](./12-netdata.md) - Real-time monitoring
