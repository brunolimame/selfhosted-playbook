# 09-11 - Uptime Kuma

## O que e?

[Uptime Kuma](https://github.com/louislam/uptime-kuma) e uma ferramenta de monitoramento de uptime auto-hospedada, bonita e facil de usar. E a alternativa open-source ao Uptime Robot, Pingdom e StatusCake.

Funcionalidades principais:
- Monitoramento HTTP(s), TCP, Ping, DNS, WebSocket, Docker Containers
- Notificacoes via Telegram, Discord, Email, WhatsApp e 90+ servicos
- Status page publica (compartilhe com clientes)
- Intervalo de 20 segundos
- Multi-idioma (inclusive portugues)
- Graficos de uptime, latencia e certificado SSL
- API para consulta de status
- Backup e restore com um clique

## Por que Uptime Kuma neste projeto?

Com dezenas de servicos rodando, voce precisa saber quando algo cai:

| Servico | Monitorar | URL de exemplo |
|---------|-----------|----------------|
| Typebot | HTTP | `https://bot.meuservidor.com` |
| n8n | HTTP | `https://n8n.meuservidor.com` |
| evolution-go | HTTP | `http://192.168.1.100:4000/manager/` |
| Dify | HTTP | `https://ia.meuservidor.com` |
| Chatwoot | HTTP | `https://atendimento.meuservidor.com` |
| Coolify | HTTP | `http://192.168.1.100:8000` |
| MinIO | HTTP | `http://192.168.1.100:9000/minio/health/live` |
| PostgreSQL | TCP | `192.168.1.100:5432` |
| Internet | Ping | `8.8.8.8` |

## Arquitetura

```
uptime-kuma (container) -> Porta 3001
    |
    +-- SQLite (banco de dados interno)
    +-- Notificacoes (Telegram, Discord, Email...)
    +-- Status page (opcional, publica)
```

## Pre-requisitos

- Docker instalado na VM
- No minimo 256 MB RAM livres

## Instalacao

### 1. Acessar a VM

```bash
ssh ubuntu@192.168.1.100
```

### 2. Criar diretorio

```bash
mkdir -p ~/uptime-kuma
cd ~/uptime-kuma
```

### 3. Criar docker-compose.yml

```bash
nano docker-compose.yml
```

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:2
    container_name: uptime-kuma
    restart: unless-stopped
    ports:
      - "3001:3001"
    volumes:
      - uptime_kuma_data:/app/data

volumes:
  uptime_kuma_data:
```

### 4. Iniciar

```bash
docker compose up -d
```

### 5. Verificar

```bash
docker compose ps
docker compose logs -f
```

## Configuracao inicial

1. Acesse `http://192.168.1.100:3001`
2. Crie o usuario admin (nome, email, senha)
3. Selecione **SQLite** como banco de dados

## Configurar dominio no Cloudflare Tunnel

```bash
nano ~/.cloudflared/config.yml
```

```yaml
  - hostname: status.meuservidor.com
    service: http://localhost:3001
```

```bash
sudo systemctl restart cloudflared
```

## Adicionar monitores

### Monitorar servicos HTTP

Clique em **Add Monitor** e configure:

```
Monitor Type: HTTP(s)
Name: Typebot - Builder
URL: https://bot.meuservidor.com
Interval: 30s
Resend Notification: 3 vezes
Notification: Telegram (configurar)
```

### Monitorar servicos TCP (banco de dados)

```
Monitor Type: TCP Port
Name: PostgreSQL
Hostname: 192.168.1.100
Port: 5432
Interval: 60s
```

### Monitorar ping (internet)

```
Monitor Type: Ping
Name: Internet - Google DNS
Hostname: 8.8.8.8
Interval: 60s
```

### Monitorar certificado SSL

```
Monitor Type: HTTP(s)
Name: SSL - bot.meuservidor.com
URL: https://bot.meuservidor.com
Resend Notification: 1x por dia (apenas se mudar)
```

## Configurar notificacoes

### Telegram

1. Clique em **Settings > Notifications > Add Notification**
2. Tipo: **Telegram**
3. Bot Token: (token do @BotFather)
4. Chat ID: (obter com `@userinfobot` ou enviar `/start` para o bot e depois `https://api.telegram.org/botTOKEN/getUpdates`)
5. Teste a notificacao

### Discord

1. Tipo: **Discord**
2. Webhook URL: (criar no Discord: Configuracoes do Canal > Integracoes > Webhooks)
3. Teste

### Email (SMTP)

1. Tipo: **SMTP**
2. Host, Port, User, Pass conforme seu provedor de email
3. Teste

## Criar Status Page publica

1. Clique em **Status Page > Add Status Page**
2. Slug: `status` (ficara em `https://status.meuservidor.com/status`)
3. Title: `Status dos Servicos`
4. Selecione os monitores para exibir
5. Ative **Publish**
6. Compartilhe o link com clientes: `https://status.meuservidor.com/status`

## Manutencao

### Atualizar

```bash
cd ~/uptime-kuma
docker compose pull
docker compose up -d
```

### Backup

```bash
# Backup completo do SQLite
docker run --rm -v uptime_kuma_data:/source -v ~/backups:/backup alpine tar czf /backup/uptime-kuma-$(date +%Y%m%d).tar.gz -C /source .
```

### Exportar/Importar monitores

No painel: **Settings > Backup > Create Backup** (arquivo JSON com toda configuracao).

## Troubleshooting

| Erro | Causa | Solucao |
|------|-------|---------|
| Monitor mostra "down" | Servico realmente fora | Verificar se o servico esta rodando |
| Notificacao nao chega | Token/configuracao errada | Testar configuracao na tela de notificacoes |
| SSL certificate expired | Certificado expirou | Verificar Cloudflare ou certbot |
| Timeout na requisicao | Servico lento ou firewall | Aumentar timeout no monitor para 30s |
| Backup corrompido | SQLite corrompido | Parar o container, copiar `kuma.db`, rodar `sqlite3 kuma.db .dump` |

## Proximo passo

[Netdata](./12-netdata.md) - Monitoramento em tempo real da VM.
