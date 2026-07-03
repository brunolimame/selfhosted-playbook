# 09-04 - Typebot

## O que e?

[Typebot](https://typebot.io) e uma plataforma open-source para criar chatbots conversacionais com fluxos visuais (no-code). E a alternativa self-hosted ao Landbot, ManyChat e Chatfuel.

Funcionalidades principais:
- Criar fluxos de conversa com blocos arrastaveis
- Suporte a texto, botoes, imagens, videos, formularios e condicoes
- Integracao nativa com **Evolution Go** (WhatsApp)
- Webhooks para conectar com n8n e outras ferramentas
- Agendamento e analytics
- Upload de arquivos com armazenamento S3/MinIO
- Multi-idioma

## Arquitetura

```
typebot-builder (container)  ->  Porta 3001 (criacao de fluxos)
typebot-viewer (container)   ->  Porta 3002 (execucao dos bots)
     |
     +-- PostgreSQL (dados, fluxos, usuarios)
     +-- (Opcional) MinIO/S3 (upload de arquivos)
```

## Por que Typebot com Evolution Go?

O Typebot tem integracao nativa com o ecossistema Evolution. O fluxo de atendimento fica:

```
Cliente envia mensagem no WhatsApp
     |
evolution-go recebe o webhook
     |
Typebot executa o fluxo configurado
     |
     +-- Se resposta automatica -> Typebot responde via API do evolution-go
     +-- Se precisa de humano -> aciona Chatwoot ou n8n
```

## Pre-requisitos

- Docker e Docker Compose instalados na VM
- No minimo 1 GB RAM livre
- Um dominio para o builder (ex: `bot.meuservidor.com`)
- Um dominio para o viewer (ex: `viewer.meuservidor.com`)

## Instalacao

### 1. Acessar a VM e criar diretorio

```bash
ssh ubuntu@192.168.1.100
mkdir -p ~/typebot
cd ~/typebot
```

### 2. Baixar arquivos de configuracao

```bash
wget https://raw.githubusercontent.com/baptisteArno/typebot.io/latest/docker-compose.yml
wget https://raw.githubusercontent.com/baptisteArno/typebot.io/latest/.env.example -O .env
```

### 3. Gerar chave de criptografia

```bash
openssl rand -base64 32 | tr -d '\n' ; echo
```

Copie o resultado para usar no `.env`.

### 4. Configurar variaveis de ambiente

```bash
nano .env
```

```env
# Database
DATABASE_URL=postgresql://postgres:typebot@postgres:5432/typebot

# Criptografia (use o valor gerado no passo anterior)
ENCRYPTION_SECRET=coloque-a-chave-de-32-caracteres-aqui

# URL do Builder (acessivel publicamente)
NEXTAUTH_URL=https://bot.meuservidor.com
NEXT_PUBLIC_VIEWER_URL=https://viewer.meuservidor.com

# Auth providers (pelo menos um obrigatorio)
# Email (magic link)
NEXT_PUBLIC_SMTP_FROM_EMAIL=noreply@meuservidor.com
SMTP_URL=smtp://user:pass@smtp.seuprovedor.com:587
NEXT_PUBLIC_SMTP_NAME=Typebot

# Google OAuth (opcional)
# GOOGLE_CLIENT_ID=...
# GOOGLE_CLIENT_SECRET=...

# GitHub OAuth (opcional)
# GITHUB_CLIENT_ID=...
# GITHUB_CLIENT_SECRET=...

# Portas (ajuste se houver conflito)
BUILDER_PORT=3001
VIEWER_PORT=3002

# S3 / MinIO para upload de arquivos (opcional)
# S3_ACCESS_KEY=minioadmin
# S3_SECRET_KEY=minioadmin
# S3_BUCKET=typebot
# S3_ENDPOINT=http://192.168.1.100:9000
# S3_SSL=false
```

### 5. Ajustar docker-compose.yml

Edite o arquivo baixado para ajustar as portas:

```bash
nano docker-compose.yml
```

Altere as secoes de porta para:

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

### 6. Iniciar

```bash
docker compose up -d
```

### 7. Verificar

```bash
docker compose ps
docker compose logs -f
```

## Configurar dominio no Cloudflare Tunnel

Edite o config.yml:

```bash
nano ~/.cloudflared/config.yml
```

Adicione antes do catch-all:

```yaml
  - hostname: bot.meuservidor.com
    service: http://localhost:3001

  - hostname: viewer.meuservidor.com
    service: http://localhost:3002
```

Reinicie:

```bash
sudo systemctl restart cloudflared
```

## Configurar arquivos de midia com MinIO (opcional)

Se tiver o MinIO instalado ([guia](./07-minio.md)), crie um bucket:

```bash
docker compose exec minio mc alias set local http://localhost:9000 minioadmin minioadmin
docker compose exec minio mc mb local/typebot
```

E no `.env` do Typebot:
```env
S3_ACCESS_KEY=minioadmin
S3_SECRET_KEY=minioadmin
S3_BUCKET=typebot
S3_ENDPOINT=http://minio:9000
S3_SSL=false
```

Reinicie o Typebot apos alterar:
```bash
docker compose down && docker compose up -d
```

## Integrar com Evolution Go

Para conectar o Typebot ao evolution-go e responder WhatsApp automaticamente:

### 1. Criar o bot no Typebot

1. Acesse `https://bot.meuservidor.com`
2. Crie sua conta
3. Crie um novo Typebot
4. Monte o fluxo de conversa desejado
5. Publique o bot e copie o **ID publico** (na URL: `/typebots/[ID]/...`)

### 2. Configurar webhook no evolution-go

No evolution-go, crie a instancia com Typebot integrado:

```bash
curl -X POST http://192.168.1.100:4000/typebot/create/meu-whatsapp \
  -H "Content-Type: application/json" \
  -H "apiKey: SUA_GLOBAL_API_KEY" \
  -d '{
    "typebot": {
      "url": "https://viewer.meuservidor.com",
      "name": "meu-bot-de-atendimento",
      "typebotId": "ID-DO-SEU-TYPEBOT",
      "startSession": true,
      "trigger": {
        "type": "keyword",
        "value": "!bot"
      }
    }
  }'
```

Agora, quando alguem enviar "!bot" no WhatsApp, o Typebot iniciara o fluxo automaticamente.

### Variaveis pre-definidas

O evolution-go envia automaticamente para o Typebot:
- `remoteJid` - ID do contato no WhatsApp
- `pushName` - Nome do contato
- `instanceName` - Nome da instancia
- `serverUrl` - URL do servidor evolution-go
- `apiKey` - Chave da API

## Manutencao

### Atualizar

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
# Backup do banco
docker compose exec postgres pg_dump -U postgres typebot > ~/backups/typebot-$(date +%Y%m%d).sql
```

## Troubleshooting

| Erro | Causa | Solucao |
|------|-------|---------|
| `ENCRYPTION_SECRET` invalido | Chave muito curta | Deve ter exatamente 32 caracteres |
| `NEXTAUTH_URL` nao configurado | URL do builder ausente | Configure `NEXTAUTH_URL` no `.env` |
| Viewer nao carrega | `NEXT_PUBLIC_VIEWER_URL` errado | Verificar URL publica do viewer |
| Erro ao enviar email | SMTP nao configurado | Configure SMTP ou use OAuth |
| Upload de arquivos falha | S3/MinIO nao configurado | Configure S3 ou desabilite upload |

## Proximo passo

[Chatwoot](./05-chatwoot.md) - Helpdesk para atendimento humano.
