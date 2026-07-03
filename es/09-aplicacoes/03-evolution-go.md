# 09-03 - Evolution Go

## ¿Qué es?

[Evolution Go](https://github.com/evolution-foundation/evolution-go) es una API de alto rendimiento para integración con WhatsApp, escrita en Go. Forma parte del ecosistema Evolution Foundation.

Funcionalidades principales:
- Enviar y recibir mensajes de texto, multimedia y documentos
- Gestionar múltiples instancias (números) simultáneamente
- Webhooks para eventos en tiempo real
- Soporte para colas RabbitMQ, NATS y WebSocket
- Almacenamiento de archivos multimedia en MinIO/S3
- Documentación Swagger integrada
- Código QR para emparejamiento de dispositivos

## Arquitectura

```
evolution-go (contenedor)
    Puerto: 4000
    |
    +-- PostgreSQL (autenticación, usuarios, mensajes)
    +-- (Opcional) RabbitMQ para colas de eventos
    +-- (Opcional) MinIO para almacenamiento de archivos multimedia
```

## Licenciamiento

Evolution Go requiere una licencia para funcionar. En el primer inicio:
1. Accede a `http://localhost:4000/manager/`
2. Ingresa tu `GLOBAL_API_KEY`
3. Completa el flujo de activación

Para obtener una licencia, consulta: https://evolutionfoundation.com.br

## Prerrequisitos

- Docker y Docker Compose instalados en la VM
- Al menos 512 MB RAM libres
- PostgreSQL 15+
- Una licencia válida de Evolution API

## Instalación

### 1. Acceder a la VM

```bash
ssh ubuntu@192.168.1.100
```

### 2. Crear directorio del proyecto

```bash
mkdir -p ~/evolution-go
cd ~/evolution-go
```

### 3. Crear archivo .env

```bash
nano .env
```

```env
# Puerto del servidor
SERVER_PORT=4000

# Identificador del cliente
CLIENT_NAME=evolution

# Clave global de la API (cámbiala por un UUID fuerte)
GLOBAL_API_KEY=tu-uuid-unico-aqui

# PostgreSQL
POSTGRES_AUTH_DB=postgresql://postgres:senha@postgres:5432/evogo_auth?sslmode=disable
POSTGRES_USERS_DB=postgresql://postgres:senha@postgres:5432/evogo_users?sslmode=disable

# Almacenamiento de mensajes (desactivado por seguridad hasta configurar)
DATABASE_SAVE_MESSAGES=false

# Logs
WADEBUG=INFO
LOGTYPE=console

# Reconectar instancias al iniciar
CONNECT_ON_STARTUP=false

# Webhooks (configurar después)
# WEBHOOK_URL=https://n8n.meuservidor.com/webhook/whatsapp
```

### 4. Crear docker-compose.yml

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

### 5. Inicializar las bases de datos

Evolution Go espera que las bases `evogo_auth` y `evogo_users` existan. Créalas en PostgreSQL antes de iniciar:

```bash
# Ejecutar dentro del contenedor PostgreSQL
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

## Configurar dominio en Cloudflare Tunnel

Edita el config.yml:

```bash
nano ~/.cloudflared/config.yml
```

Agrega:

```yaml
  - hostname: whatsapp.meuservidor.com
    service: http://localhost:4000
```

Reinicia:

```bash
sudo systemctl restart cloudflared
```

## Primer uso

### 1. Activar licencia

Accede a `http://192.168.1.100:4000/manager/` y sigue el flujo de activación con tu `GLOBAL_API_KEY`.

### 2. Crear una instancia

```bash
curl -X POST http://192.168.1.100:4000/instance/create \
  -H "Content-Type: application/json" \
  -H "apiKey: TU_GLOBAL_API_KEY" \
  -d '{
    "instanceName": "mi-whatsapp",
    "qrcode": true
  }'
```

### 3. Obtener Código QR

```bash
curl -X GET http://192.168.1.100:4000/instance/qrcode \
  -H "apiKey: TU_GLOBAL_API_KEY" \
  -d '{"instanceName": "mi-whatsapp"}'
```

Accede a la URL devuelta o abre la imagen del Código QR en el navegador.
Escanéalo con WhatsApp en tu celular: **Dispositivos Vinculados > Vincular un dispositivo**.

### 4. Enviar un mensaje

```bash
curl -X POST http://192.168.1.100:4000/message/sendText \
  -H "Content-Type: application/json" \
  -H "apiKey: TU_GLOBAL_API_KEY" \
  -d '{
    "number": "5511999999999",
    "options": {
      "delay": 1000,
      "presence": "composing"
    },
    "textMessage": {
      "text": "¡Hola desde Evolution Go!"
    }
  }'
```

## Configuración avanzada

### Con RabbitMQ (para colas de eventos)

```yaml
  rabbitmq:
    image: rabbitmq:3.13-alpine
    container_name: evogo-rabbitmq
    restart: unless-stopped
    volumes:
      - evogo_rabbitmq_data:/var/lib/rabbitmq
```

Agrega al `.env`:
```env
AMQP_ENABLED=true
AMQP_URL=amqp://guest:guest@rabbitmq:5672
AMQP_GLOBAL_EXCHANGE=true
AMQP_GLOBAL_EVENTS=presence.update,message.upsert,message.update,connection.update
```

### Con MinIO (para almacenamiento de archivos multimedia)

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

Agrega al `.env`:
```env
MINIO_ENABLED=true
MINIO_ENDPOINT=minio:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_BUCKET=evolution-media
```

## Integración con n8n

n8n puede consumir eventos de Evolution Go vía webhook:

1. En Evolution Go, configura el webhook en el `.env`:
```env
WEBHOOK_URL=https://n8n.meuservidor.com/webhook/whatsapp
```

2. En n8n, crea un workflow con un nodo **Webhook** configurado para `/webhook/whatsapp`

3. Usa el nodo **HTTP Request** en n8n para enviar mensajes vía Evolution Go.

## Mantenimiento

### Actualizar

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

# Backup de las bases
docker compose exec -T postgres pg_dump -U postgres evogo_auth > $BACKUP_DIR/evogo-auth-$DATE.sql
docker compose exec -T postgres pg_dump -U postgres evogo_users > $BACKUP_DIR/evogo-users-$DATE.sql

gzip $BACKUP_DIR/*.sql

echo "Backup completado: $DATE"
```

### Listar instancias

```bash
curl -X GET http://192.168.1.100:4000/instance/fetchInstances \
  -H "apiKey: TU_GLOBAL_API_KEY"
```

## Troubleshooting

| Error | Causa | Solución |
|-------|-------|----------|
| `License not activated` | Licencia no activada | Accede a `/manager/` y actívala |
| `QR Code expired` | Tiempo excedido | Genera un nuevo Código QR |
| `Connection refused` | Número baneado temporalmente | Espera 24h o cambia de número |
| `ECONNREFUSED postgres` | PostgreSQL no inició | `docker compose logs postgres` |
| `401 Unauthorized` | API Key inválida | Verifica `GLOBAL_API_KEY` en el `.env` |

## Siguiente paso

[Seguridad](../07-seguranca.md) - Protege tu servidor.
