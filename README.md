# selfhosted-playbook

Complete playbook to create a home server with public domain access, using VirtualBox, Ubuntu Server, Coolify, Cloudflare Tunnel and dozens of self-hosted tools.

## Architecture

```
Internet -> vm.doc.local -> Cloudflare DNS -> Cloudflare Edge
                                                  |
      VM (VirtualBox) <- cloudflared (tunnel) <-----+
           |
      Coolify (Dashboard :8000)
           |
      Docker (applications)
```

## Documentation

| Language | Link | Root README |
|----------|------|-------------|
| English | [en/README.md](./en/README.md) | [README.md](./README.md) |
| Portugues | [pt/README.md](./pt/README.md) | [README.pt.md](./README.pt.md) |
| Espanol | [es/README.md](./es/README.md) | [README.es.md](./README.es.md) |

### Quick start (English)

1. [Introduction](./en/01-introduction.md) - Architecture overview
2. [Prerequisites](./en/02-prerequisites.md) - What you need
3. [VM](./en/03-vm/README.md) - VirtualBox + Ubuntu Server
4. [Domain](./en/04-domain.md) - Cloudflare DNS
5. [Public Exposure](./en/05-public-exposure/README.md) - Cloudflare Tunnel
6. [Coolify](./en/06-coolify/README.md) - App management
7. [Applications](./en/09-applications/README.md) - n8n, Typebot, evolution-go, Chatwoot, Dify, MinIO, Uptime Kuma, Netdata and more
8. [Security](./en/07-security.md) - Firewall, SSH, WAF

## What this playbook covers

### Base infrastructure
- VirtualBox installation on Windows, Linux and macOS
- Ubuntu Server LTS setup
- Static IP network configuration
- Domain registration and Cloudflare DNS

### Public exposure (4 methods)
- **Cloudflare Tunnel** (recommended)
- DDNS + Port Forwarding
- Tailscale Funnel
- ngrok

### Applications (12 documented)

| App | Purpose | Port |
|-----|---------|------|
| [Coolify](en/06-coolify/README.md) | Application management | 8000 |
| [n8n](en/09-applications/02-n8n.md) | Workflow automation | 5678 |
| [Typebot](en/09-applications/04-typebot.md) | Visual chatbot (WhatsApp, Telegram, Web) | 3001-3002 |
| [evolution-go](en/09-applications/03-evolution-go.md) | WhatsApp API | 4000 |
| [Chatwoot](en/09-applications/05-chatwoot.md) | Multi-agent helpdesk | 3000 |
| [Dify](en/09-applications/06-dify.md) | AI with RAG and knowledge base | 8080 |
| [MinIO](en/09-applications/07-minio.md) | S3 storage | 9000-9001 |
| [Uptime Kuma](en/09-applications/11-uptime-kuma.md) | Uptime monitoring | 3001 |
| [Netdata](en/09-applications/12-netdata.md) | Real-time metrics | 19999 |
| [pgAdmin](en/09-applications/10-pgadmin.md) | PostgreSQL administration | 5050 |
| [Graphify](en/09-applications/09-graphify.md) | Knowledge graph for AI | 8080 |
| [Odysseus](en/09-applications/01-odysseus.md) | AI workspace | 7000 |

### Multichannel chatbot
Complete guide to build a unified chatbot serving:
- **WhatsApp** (evolution-go + Typebot)
- **Telegram** (Bot API + n8n)
- **Facebook / Instagram** (Meta API + n8n)
- **Web** (Typebot embed)
- **Email** (n8n IMAP/SMTP)

### Security
- UFW Firewall
- SSH key authentication
- Fail2ban
- Cloudflare WAF
- Automated backups

## For LLMs: expand the documentation

Use the file [`en/ADD_APPLICATION.md`](./en/ADD_APPLICATION.md) as a template to document new applications. It contains instructions, checklist and a full markdown template for LLMs.

## License

[CC BY 4.0](./LICENSE) - Bruno Lima

| Language | File |
|----------|------|
| English | [LICENSE](./LICENSE) |
| Portugues (BR) | [LICENSE.pt-BR.md](./LICENSE.pt-BR.md) |
| Espanol | [LICENSE.es.md](./LICENSE.es.md) |
