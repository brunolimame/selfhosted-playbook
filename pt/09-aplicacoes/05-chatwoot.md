# 09-05 - Chatwoot

## O que e?

[Chatwoot](https://www.chatwoot.com) e uma plataforma open-source de atendimento ao cliente (helpdesk). E a alternativa self-hosted ao Intercom, Zendesk e Freshdesk.

Funcionalidades principais:
- Caixa de entrada unificada (WhatsApp, Email, Webchat, Telegram, etc.)
- Atendimento multi-agente com atribuicao de conversas
- Respostas pre-definidas (macros/canned responses)
- Notas internas e mencoes entre agentes
- Relatorios e analytics
- API e webhooks para integracoes
- Automacao de regras (auto-assign, labels, etc.)

## Arquitetura

```
chatwoot-web (container)    -> Porta 3000 (interface web + API)
chatwoot-worker (container) -> Sidekiq (jobs em background)
     |
     +-- PostgreSQL (conversas, contatos, agentes)
     +-- Redis (cache, filas de jobs)
```

## Por que Chatwoot com Evolution Go e Typebot?

O Chatwoot e o elo humano da automacao de atendimento. O fluxo completo:

```
Cliente envia mensagem no WhatsApp
     |
evolution-go recebe o webhook
     |
Typebot tenta resolver automaticamente
     |
     +-- Se cliente precisa de humano -> evolution-go enpara para Chatwoot
     |
Chatwoot exibe para o agente disponivel
     |
Agente responde -> Chatwoot envia para evolution-go -> evolution-go entrega no WhatsApp
```

## Pre-requisitos

- Docker e Docker Compose instalados na VM
- No minimo 2 GB RAM livres (Chatwoot e pesado)
- Dominio configurado: `atendimento.meuservidor.com`
- Conta SMTP para emails transacionais (opcional na instalacao inicial)

## Instalacao

### 1. Acessar a VM e criar diretorio

```bash
ssh ubuntu@192.168.1.100
mkdir -p ~/chatwoot
cd ~/chatwoot
```

### 2. Baixar o docker-compose de producao

```bash
wget -O docker-compose.yml https://raw.githubusercontent.com/chatwoot/chatwoot/develop/docker-compose.production.yaml
```

### 3. Criar arquivo .env

```bash
nano .env
```

```env
# === Geral ===
INSTALLATION_NAME=Meu Atendimento

# === Porta ===
PORT=3000

# === PostgreSQL ===
POSTGRES_USER=chatwoot
POSTGRES_PASSWORD=senha-forte-aqui
POSTGRES_DATABASE=chatwoot

# === Redis ===
REDIS_PASSWORD=redis-senha-aqui

# === URL base (dominio publico) ===
FRONTEND_URL=https://atendimento.meuservidor.com

# === Chave secreta (gerar com: openssl rand -hex 64) ===
SECRET_KEY_BASE=coloque-uma-chave-hex-de-64-caracteres-aqui

# === Chave de criptografia (gerar com: openssl rand -hex 32) ===
ENCRYPTION_PRIMARY_KEY=coloque-uma-chave-hex-de-32-caracteres-aqui

# === SMTP (para enviar emails de notificacao) ===
# SMTP_ADDRESS=smtp.gmail.com
# SMTP_PORT=587
# SMTP_USERNAME=seu-email@gmail.com
# SMTP_PASSWORD=sua-senha
# SMTP_AUTH_METHOD=plain
# SMTP_ENABLE_STARTTLS_AUTO=true
# SMTP_DOMAIN=gmail.com

# === Armazenamento (opcional - use MinIO) ===
# ACTIVE_STORAGE_SERVICE=local
# STORAGE_DIR=/app/storage

# === Idioma padrao ===
DEFAULT_LOCALE=pt_BR
```

### 4. Gerar as chaves

```bash
echo "SECRET_KEY_BASE=$(openssl rand -hex 64)"
echo "ENCRYPTION_PRIMARY_KEY=$(openssl rand -hex 32)"
```

Copie os valores para o `.env`.

### 5. Ajustar o docker-compose.yml

Edite para usar as portas corretas:

```bash
nano docker-compose.yml
```

Na secao do servico `chatwoot`, ajuste as portas:

```yaml
  chatwoot:
    ports:
      - "${PORT}:3000"
```

E no `depends_on`, altere para usar os nomes dos servicos corretos.

### 6. Preparar o banco de dados

```bash
docker compose run --rm chatwoot bundle exec rails db:chatwoot_prepare
```

Esse comando:
- Cria as tabelas no PostgreSQL
- Executa as migrations
- Semeia dados iniciais

### 7. Iniciar

```bash
docker compose up -d
```

### 8. Verificar

```bash
docker compose ps
docker compose logs -f chatwoot
```

Aguardar ate ver: `Listening on http://0.0.0.0:3000`

### 9. Criar conta admin

Acesse `http://192.168.1.100:3000` e crie a conta de administrador.

## Configurar dominio no Cloudflare Tunnel

Edite o config.yml:

```bash
nano ~/.cloudflared/config.yml
```

Adicione:

```yaml
  - hostname: atendimento.meuservidor.com
    service: http://localhost:3000
```

Reinicie:

```bash
sudo systemctl restart cloudflared
```

## Integrar com Evolution Go

### 1. No Chatwoot

1. Va em **Settings > Inboxes > Add Inbox**
2. Selecione **Evolution API**
3. Configure:
   - **Name**: WhatsApp Comercial
   - **Evolution API URL**: `http://192.168.1.100:4000`
   - **Global API Key**: (sua chave do evolution-go)
   - **Instance Name**: (nome da instancia no evolution-go)
4. Clique em **Create Inbox**

### 2. No evolution-go

Configure o webhook para enviar mensagens ao Chatwoot:

```bash
curl -X POST http://192.168.1.100:4000/webhook/create/meu-whatsapp \
  -H "Content-Type: application/json" \
  -H "apiKey: SUA_GLOBAL_API_KEY" \
  -d '{
    "webhook": {
      "url": "https://atendimento.meuservidor.com/webhooks/evolution/SEU-WEBHOOK-TOKEN",
      "events": ["message.upsert", "message.update", "connection.update"]
    }
  }'
```

O token do webhook e gerado automaticamente ao criar o inbox no Chatwoot.

## Fluxo completo (Typebot + Chatwoot)

Para configurar o roteamento automatico onde o Typebot tenta resolver e o Chatwoot assume quando necessario:

1. **Typebot**: cria um fluxo com bloco "Set Variable" definindo `transferir=false`
2. **Typebot**: se cliente pedir "atendente" ou "humano", muda `transferir=true`
3. **Typebot**: bloco "Webhook" envia para o n8n com a flag `transferir`
4. **n8n**: se `transferir=true`, chama a API do Chatwoot para criar conversa e notificar agente
5. **n8n**: envia mensagem no WhatsApp via evolution-go: "Voce sera atendido em instantes"

Exemplo de webhook n8n para criar conversa no Chatwoot:

```bash
curl -X POST https://atendimento.meuservidor.com/api/v1/accounts/1/conversations \
  -H "Content-Type: application/json" \
  -H "api_access_token: SEU-TOKEN-DO-CHATWOOT" \
  -d '{
    "source_id": "5511999999999@s.whatsapp.net",
    "inbox_id": 1,
    "contact_id": 1,
    "status": "pending"
  }'
```

## Manutencao

### Atualizar

```bash
cd ~/chatwoot
docker compose pull
docker compose down
docker compose up -d
```

Apos atualizar, rode as migrations se necessario:

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

# Backup do banco
docker compose exec -T postgres pg_dump -U chatwoot chatwoot > $BACKUP_DIR/chatwoot-db-$DATE.sql
gzip $BACKUP_DIR/chatwoot-db-$DATE.sql

# Backup de arquivos
docker run --rm -v chatwoot_data:/source -v $BACKUP_DIR:/backup alpine tar czf /backup/chatwoot-storage-$DATE.tar.gz -C /source .

echo "Backup concluido: $DATE"
```

## Troubleshooting

| Erro | Causa | Solucao |
|------|-------|---------|
| `500 Internal Server Error` | SECRET_KEY_BASE invalido | Gerar nova chave com `openssl rand -hex 64` |
| PostgreSQL connection refused | Postgres nao iniciou | `docker compose logs postgres` |
| Emails nao saem | SMTP nao configurado | Configurar SMTP no `.env` e reiniciar |
| Evolution inbox nao conecta | URL ou API Key errada | Verificar endpoint do evolution-go |
| Tela branca no login | `FRONTEND_URL` errada | Usar URL exata com https |

## Proximo passo

[Dify](./06-dify.md) - IA com base de conhecimento (RAG).
