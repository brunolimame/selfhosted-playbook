# 09-03 - Evolution Go

## O que e?

[Evolution Go](https://github.com/evolution-foundation/evolution-go) e uma API de alta performance para integracao com WhatsApp, escrita em Go. Faz parte do ecossistema Evolution Foundation.

Funcionalidades principais:
- Enviar e receber mensagens de texto, midia e documentos
- Gerenciar multiplas instancias (numeros) simultaneamente
- Webhooks para eventos em tempo real
- Suporte a filas RabbitMQ, NATS e WebSocket
- Armazenamento de midia em MinIO/S3
- Documentacao Swagger integrada
- QR Code para pareamento de dispositivos

## Arquitetura

```
evolution-go (container)
    Porta: 4000
    |
    +-- PostgreSQL (autenticacao, usuarios, mensagens)
    +-- (Opcional) RabbitMQ para filas de eventos
    +-- (Opcional) MinIO para armazenamento de midia
```

## Licenciamento

O Evolution Go requer uma licenca para funcionar. No primeiro inicio:
1. Acesse `http://localhost:4000/manager/`
2. Informe sua `GLOBAL_API_KEY`
3. Complete o fluxo de ativacao

Para obter uma licenca, consulte: https://evolutionfoundation.com.br

## Pre-requisitos

- Docker e Docker Compose instalados na VM
- No minimo 512 MB RAM livres
- PostgreSQL 15+
- Uma licenca valida do Evolution API

## Instalacao

### 1. Acessar a VM

```bash
ssh ubuntu@192.168.1.100
```

### 2. Criar diretorio do projeto

```bash
mkdir -p ~/evolution-go
cd ~/evolution-go
```

### 3. Criar arquivo .env

```bash
nano .env
```

```env
# Porta do servidor
SERVER_PORT=4000

# Identificador do cliente
CLIENT_NAME=evolution

# Chave global da API (troque por um UUID forte!)
GLOBAL_API_KEY=seu-uuid-unico-aqui

# PostgreSQL
POSTGRES_AUTH_DB=postgresql://postgres:senha@postgres:5432/evogo_auth?sslmode=disable
POSTGRES_USERS_DB=postgresql://postgres:senha@postgres:5432/evogo_users?sslmode=disable

# Armazenamento de mensagens (desligado por seguranca ate configurar)
DATABASE_SAVE_MESSAGES=false

# Logs
WADEBUG=INFO
LOGTYPE=console

# Reconectar instancias ao iniciar
CONNECT_ON_STARTUP=false

# Webhooks (configurar depois)
# WEBHOOK_URL=https://n8n.meuservidor.com/webhook/whatsapp
```

### 4. Criar docker-compose.yml

```bash
nano docker-compose.yml
```

```yaml
services:
  postgres:
    image: postgres:15-alpine
    container_name: evogo-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: senha
    ports:
      - "5432:5432"
    volumes:
      - evogo_postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  evolution-go:
    image: evoapicloud/evolution-go:latest
    container_name: evolution-go
    restart: unless-stopped
    ports:
      - "4000:4000"
    environment:
      SERVER_PORT: ${SERVER_PORT}
      CLIENT_NAME: ${CLIENT_NAME}
      GLOBAL_API_KEY: ${GLOBAL_API_KEY}
      POSTGRES_AUTH_DB: ${POSTGRES_AUTH_DB}
      POSTGRES_USERS_DB: ${POSTGRES_USERS_DB}
      DATABASE_SAVE_MESSAGES: ${DATABASE_SAVE_MESSAGES}
      WADEBUG: ${WADEBUG}
      LOGTYPE: ${LOGTYPE}
      CONNECT_ON_STARTUP: ${CONNECT_ON_STARTUP}
    volumes:
      - evogo_data:/app/dbdata
      - evogo_logs:/app/logs
    depends_on:
      postgres:
        condition: service_healthy

volumes:
  evogo_postgres_data:
  evogo_data:
  evogo_logs:
```

### 5. Inicializar os bancos de dados

O Evolution Go espera que os bancos `evogo_auth` e `evogo_users` existam. Crie-os no PostgreSQL antes de iniciar:

```bash
# Executar dentro do container PostgreSQL
docker compose exec postgres psql -U postgres -c "CREATE DATABASE evogo_auth;"
docker compose exec postgres psql -U postgres -c "CREATE DATABASE evogo_users;"
```

### 6. Iniciar

```bash
cd ~/evolution-go
docker compose up -d
```

### 7. Verificar

```bash
docker compose ps
docker compose logs -f evolution-go
```

## Configurar dominio no Cloudflare Tunnel

Edite o config.yml:

```bash
nano ~/.cloudflared/config.yml
```

Adicione:

```yaml
  - hostname: whatsapp.meuservidor.com
    service: http://localhost:4000
```

Reinicie:

```bash
sudo systemctl restart cloudflared
```

## Primeiro uso

### 1. Ativar licenca

Acesse `http://192.168.1.100:4000/manager/` e siga o fluxo de ativacao com sua `GLOBAL_API_KEY`.

### 2. Criar uma instancia

```bash
curl -X POST http://192.168.1.100:4000/instance/create \
  -H "Content-Type: application/json" \
  -H "apiKey: SUA_GLOBAL_API_KEY" \
  -d '{
    "instanceName": "meu-whatsapp",
    "qrcode": true
  }'
```

### 3. Obter QR Code

```bash
curl -X GET http://192.168.1.100:4000/instance/qrcode \
  -H "apiKey: SUA_GLOBAL_API_KEY" \
  -d '{"instanceName": "meu-whatsapp"}'
```

Acesse a URL retornada ou abra a imagem do QR Code no navegador.
Escaneie com o WhatsApp no seu celular: **Aparelhos Conectados > Conectar um dispositivo**.

### 4. Enviar uma mensagem

```bash
curl -X POST http://192.168.1.100:4000/message/sendText \
  -H "Content-Type: application/json" \
  -H "apiKey: SUA_GLOBAL_API_KEY" \
  -d '{
    "number": "5511999999999",
    "options": {
      "delay": 1000,
      "presence": "composing"
    },
    "textMessage": {
      "text": "Ola do Evolution Go!"
    }
  }'
```

## Configuracao avancada

### Com RabbitMQ (para filas de eventos)

```yaml
  rabbitmq:
    image: rabbitmq:3.13-alpine
    container_name: evogo-rabbitmq
    restart: unless-stopped
    volumes:
      - evogo_rabbitmq_data:/var/lib/rabbitmq
```

Adicione ao `.env`:
```env
AMQP_ENABLED=true
AMQP_URL=amqp://guest:guest@rabbitmq:5672
AMQP_GLOBAL_EXCHANGE=true
AMQP_GLOBAL_EVENTS=presence.update,message.upsert,message.update,connection.update
```

### Com MinIO (para armazenamento de midia)

```yaml
  minio:
    image: minio/minio:latest
    container_name: evogo-minio
    restart: unless-stopped
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    volumes:
      - evogo_minio_data:/data
```

Adicione ao `.env`:
```env
MINIO_ENABLED=true
MINIO_ENDPOINT=minio:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_BUCKET=evolution-media
```

## Integracao com n8n

O n8n pode consumir eventos do Evolution Go via webhook:

1. No Evolution Go, configure o webhook no `.env`:
```env
WEBHOOK_URL=https://n8n.meuservidor.com/webhook/whatsapp
```

2. No n8n, crie um workflow com no **Webhook** configurado para `/webhook/whatsapp`

3. Use o no **HTTP Request** no n8n para enviar mensagens via Evolution Go.

## Manutencao

### Atualizar

```bash
cd ~/evolution-go
docker compose pull
docker compose up -d
```

### Logs

```bash
docker compose logs -f evolution-go
```

### Backup

```bash
#!/bin/bash
# ~/evolution-go/backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR=~/backups/evolution-go
mkdir -p $BACKUP_DIR

# Backup dos bancos
docker compose exec -T postgres pg_dump -U postgres evogo_auth > $BACKUP_DIR/evogo-auth-$DATE.sql
docker compose exec -T postgres pg_dump -U postgres evogo_users > $BACKUP_DIR/evogo-users-$DATE.sql

gzip $BACKUP_DIR/*.sql

echo "Backup concluido: $DATE"
```

### Listar instancias

```bash
curl -X GET http://192.168.1.100:4000/instance/fetchInstances \
  -H "apiKey: SUA_GLOBAL_API_KEY"
```

## Troubleshooting

| Erro | Causa | Solucao |
|------|-------|---------|
| `License not activated` | Licenca nao ativada | Acesse `/manager/` e ative |
| `QR Code expired` | Tempo excedido | Gere novo QR Code |
| `Connection refused` | Numero banido temporariamente | Aguarde 24h ou troque de numero |
| `ECONNREFUSED postgres` | PostgreSQL nao iniciou | `docker compose logs postgres` |
| `401 Unauthorized` | API Key invalida | Verificar `GLOBAL_API_KEY` no `.env` |

## Proximo passo

[Seguranca](../07-seguranca.md) - Proteja seu servidor.
