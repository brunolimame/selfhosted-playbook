# selfhosted-playbook

Playbook completo para crear un servidor casero con acceso publico via dominio, utilizando VirtualBox, Ubuntu Server, Coolify, Cloudflare Tunnel y decenas de herramientas auto-hospedadas.

## Arquitectura

```
Internet -> vm.doc.local -> Cloudflare DNS -> Cloudflare Edge
                                                  |
      VM (VirtualBox) <- cloudflared (tunnel) <-----+
           |
      Coolify (Panel :8000)
           |
      Docker (aplicaciones)
```

## Documentacion

| Idioma | Enlace |
|--------|--------|
| Espanol | [es/README.md](./es/README.md) |
| English | [en/README.md](./en/README.md) |
| Portugues | [pt/README.md](./pt/README.md) |

### Inicio rapido (Espanol)

1. [Introduccion](./es/01-introducao.md) - Vision general de la arquitectura
2. [Pre-requisitos](./es/02-pre-requisitos.md) - Lo que necesitas
3. [VM](./es/03-vm/README.md) - VirtualBox + Ubuntu Server
4. [Dominio](./es/04-dominio.md) - Cloudflare DNS
5. [Exposicion Publica](./es/05-exposicao-publica/README.md) - Cloudflare Tunnel
6. [Coolify](./es/06-coolify/README.md) - Gestion de aplicaciones
7. [Aplicaciones](./es/09-aplicacoes/README.md) - n8n, Typebot, evolution-go, Chatwoot, Dify, MinIO, Uptime Kuma, Netdata y mas
8. [Seguridad](./es/07-seguranca.md) - Firewall, SSH, WAF

## Lo que cubre este playbook

### Infraestructura base
- Instalacion de VirtualBox en Windows, Linux y macOS
- Configuracion de Ubuntu Server LTS
- Red con IP fija
- Registro de dominio y Cloudflare DNS

### Exposicion publica (4 metodos)
- **Cloudflare Tunnel** (recomendado)
- DDNS + Port Forwarding
- Tailscale Funnel
- ngrok

### Aplicaciones (12 documentadas)

| App | Proposito | Puerto |
|-----|-----------|--------|
| [Coolify](es/06-coolify/README.md) | Gestion de aplicaciones | 8000 |
| [n8n](es/09-aplicacoes/02-n8n.md) | Automatizacion de workflows | 5678 |
| [Typebot](es/09-aplicacoes/04-typebot.md) | Chatbot visual (WhatsApp, Telegram, Web) | 3001-3002 |
| [evolution-go](es/09-aplicacoes/03-evolution-go.md) | API de WhatsApp | 4000 |
| [Chatwoot](es/09-aplicacoes/05-chatwoot.md) | Helpdesk multi-agente | 3000 |
| [Dify](es/09-aplicacoes/06-dify.md) | IA con RAG y base de conocimiento | 8080 |
| [MinIO](es/09-aplicacoes/07-minio.md) | Almacenamiento S3 | 9000-9001 |
| [Uptime Kuma](es/09-aplicacoes/11-uptime-kuma.md) | Monitoreo de uptime | 3001 |
| [Netdata](es/09-aplicacoes/12-netdata.md) | Metricas en tiempo real | 19999 |
| [pgAdmin](es/09-aplicacoes/10-pgadmin.md) | Administracion PostgreSQL | 5050 |
| [Graphify](es/09-aplicacoes/09-graphify.md) | Grafo de conocimiento para IA | 8080 |
| [Odysseus](es/09-aplicacoes/01-odysseus.md) | Workspace de IA | 7000 |

### Chatbot multicanal
Guia completa para crear un chatbot unificado que atiende en:
- **WhatsApp** (evolution-go + Typebot)
- **Telegram** (Bot API + n8n)
- **Facebook / Instagram** (Meta API + n8n)
- **Web** (Typebot embed)
- **Email** (n8n IMAP/SMTP)

### Seguridad
- Firewall (UFW)
- SSH con clave
- Fail2ban
- Cloudflare WAF
- Backup automatizado

## Para LLMs: expandir la documentacion

Usa el archivo [`es/ADICIONAR_APLICACAO.md`](./es/ADICIONAR_APLICACAO.md) como plantilla para documentar nuevas aplicaciones. Contiene instrucciones, checklist y plantilla markdown para LLMs.

## Licencia

[CC BY 4.0](./LICENSE) - Bruno Lima

| Idioma | Archivo |
|--------|---------|
| English | [LICENSE](./LICENSE) |
| Portugues (BR) | [LICENSE.pt-BR.md](./LICENSE.pt-BR.md) |
| Espanol | [LICENSE.es.md](./LICENSE.es.md) |
