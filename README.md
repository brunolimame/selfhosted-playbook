# selfhosted-playbook

Playbook completo para criar um servidor caseiro com acesso publico via dominio, utilizando VirtualBox, Ubuntu Server, Coolify, Cloudflare Tunnel e dezenas de ferramentas auto-hospedadas.

## Arquitetura

```
Internet -> vm.doc.local -> Cloudflare DNS -> Cloudflare Edge
                                                  |
     VM (VirtualBox) <- cloudflared (tunnel) <-----+
          |
     Coolify (Painel :8000)
          |
     Docker (aplicacoes)
```

## Documentacao

| Idioma | Link |
|--------|------|
| Portugues | [pt/README.md](./pt/README.md) |
| English | *em breve* |
| Espanol | *em breve* |

### Inicio rapido (Portugues)

1. [Introducao](./pt/01-introducao.md) - Visao geral da arquitetura
2. [Pre-requisitos](./pt/02-pre-requisitos.md) - O que voce precisa
3. [VM](./pt/03-vm/README.md) - VirtualBox + Ubuntu Server
4. [Dominio](./pt/04-dominio.md) - Cloudflare DNS
5. [Exposicao Publica](./pt/05-exposicao-publica/README.md) - Cloudflare Tunnel
6. [Coolify](./pt/06-coolify/README.md) - Gerenciamento de apps
7. [Aplicacoes](./pt/09-aplicacoes/README.md) - n8n, Typebot, evolution-go, Chatwoot, Dify, MinIO, Uptime Kuma, Netdata e mais
8. [Seguranca](./pt/07-seguranca.md) - Firewall, SSH, WAF

## O que este playbook cobre

### Infraestrutura base
- Instalacao do VirtualBox em Windows, Linux e macOS
- Configuracao do Ubuntu Server LTS
- Rede com IP fixo
- Registro de dominio e Cloudflare DNS

### Exposicao publica (4 metodos)
- **Cloudflare Tunnel** (recomendado)
- DDNS + Port Forwarding
- Tailscale Funnel
- ngrok

### Aplicacoes (12 documentadas)

| App | Finalidade | Porta |
|-----|-----------|-------|
| [Coolify](pt/06-coolify/README.md) | Gerenciamento de aplicacoes | 8000 |
| [n8n](pt/09-aplicacoes/02-n8n.md) | Automacao de workflows | 5678 |
| [Typebot](pt/09-aplicacoes/04-typebot.md) | Chatbot visual (WhatsApp, Telegram, Web) | 3001-3002 |
| [evolution-go](pt/09-aplicacoes/03-evolution-go.md) | API do WhatsApp | 4000 |
| [Chatwoot](pt/09-aplicacoes/05-chatwoot.md) | Helpdesk multi-agente | 3000 |
| [Dify](pt/09-aplicacoes/06-dify.md) | IA com RAG e base de conhecimento | 8080 |
| [MinIO](pt/09-aplicacoes/07-minio.md) | Armazenamento S3 | 9000-9001 |
| [Uptime Kuma](pt/09-aplicacoes/11-uptime-kuma.md) | Monitoramento de uptime | 3001 |
| [Netdata](pt/09-aplicacoes/12-netdata.md) | Metricas em tempo real | 19999 |
| [pgAdmin](pt/09-aplicacoes/10-pgadmin.md) | Administracao PostgreSQL | 5050 |
| [Graphify](pt/09-aplicacoes/09-graphify.md) | Grafo de conhecimento para IA | 8080 |
| [Odysseus](pt/09-aplicacoes/01-odysseus.md) | Workspace de IA | 7000 |

### Chatbot multicanal
Guia completo para criar um chatbot unificado que atende em:
- **WhatsApp** (evolution-go + Typebot)
- **Telegram** (Bot API + n8n)
- **Facebook / Instagram** (Meta API + n8n)
- **Web** (Typebot embed)
- **Email** (n8n IMAP/SMTP)

### Seguranca
- Firewall (UFW)
- SSH com chave
- Fail2ban
- Cloudflare WAF
- Backup automatizado

## Para LLMs: expandir a documentacao

Use o arquivo [`pt/ADICIONAR_APLICACAO.md`](./pt/ADICIONAR_APLICACAO.md) como template para documentar novas aplicacoes. Ele contem instrucoes, checklist e template markdown para LLMs.

## Licenca

MIT
