# 09-08 - Evolution API (Node.js)

## ¿Qué es?

La [Evolution API](https://github.com/evolution-foundation/evolution-api) es la versión Node.js/TypeScript del ecosistema Evolution. Diferente de [evolution-go](./03-evolution-go.md) (ligero y enfocado en WhatsApp via whatsmeow), esta versión ofrece:

- **Multi-proveedor**: Baileys (WhatsApp Web), Meta Cloud API (WhatsApp Business)
- **Soporte futuro**: Instagram y Messenger (en desarrollo)
- **Gestión vía Web**: Panel administrativo incluido (evolution-manager)
- **Integraciones nativas**: Typebot, Chatwoot, Dify integradas en la API
- **Multi-base**: PostgreSQL y MySQL
- **Más eventos**: WebSocket, Socket.io, RabbitMQ, Kafka, SQS, NATS, Pusher

## ¿Cuándo usar evolution-api vs. evolution-go?

| Característica | evolution-go | evolution-api (Node.js) |
|---------------|-------------|------------------------|
| Rendimiento | Alto (Go) | Medio (Node.js) |
| Tamaño | ~50 MB | ~200 MB |
| Panel admin | No (vía API) | Sí (evolution-manager) |
| Integraciones nativas | Webhooks manuales | Typebot, Chatwoot, Dify built-in |
| Instagram/Messenger | No | Futuro |
| Base de datos | PostgreSQL | PostgreSQL o MySQL |
| Caché | No | Redis |
| Licensing | Sí | Sí |

## Arquitectura

```
evolution-api (contenedor)      -> Puerto 8080 (API)
evolution-manager (contenedor)  -> Puerto 3000 (Panel web)
     |
     +-- PostgreSQL (datos)
     +-- Redis (caché/colas)
```

## Prerrequisitos

- Docker y Docker Compose
- Mínimo 2 GB RAM
- PostgreSQL 15+
- Redis 7+
- Licencia Evolution API (consulte https://evolutionfoundation.com.br)

## Instalación

### 1. Acceder a la VM y crear directorio

```bash
ssh ubuntu@192.168.1.100
mkdir -p ~/evolution-api
cd ~/evolution-api
```

### 2. Crear docker-compose.yml

```bash
nano docker-compose.yml
```

```yaml
services:
  redis:
    image: redis:7-alpine
    container_name: evo-redis
    restart: unless-stopped
    volumes:
      - evo_redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  postgres:
    image: postgres:15-alpine
    container_name: evo-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: evolution
      POSTGRES_PASSWORD: senha-forte-aqui
      POSTGRES_DB: evolution
    volumes:
      - evo_postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U evolution"]
      interval: 10s
      timeout: 5s
      retries: 5

  api:
    image: evoapicloud/evolution-api:latest
    container_name: evolution-api
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - evo_instances:/evolution/instances
    env_file: .env
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy

  manager:
    image: evoapicloud/evolution-manager:latest
    container_name: evolution-manager
    restart: unless-stopped
    ports:
      - "3000:80"
    depends_on:
      - api

volumes:
  evo_redis_data:
  evo_postgres_data:
  evo_instances:
```

### 3. Crear .env

```bash
nano .env
```

```env
# === Database ===
DATABASE_PROVIDER=postgresql
DATABASE_URL=postgresql://evolution:senha-forte-aqui@postgres:5432/evolution

# === Redis ===
REDIS_URI=redis://redis:6379
CACHE_TTL=86400

# === Autenticación ===
AUTHENTICATION_TYPE=apikey
GLOBAL_API_KEY=seu-uuid-unico-aqui

# === Puerto ===
PORT=8080

# === Proveedores habilitados ===
PROVIDER_BAILEYS_ENABLED=true
PROVIDER_META_ENABLED=false

# === Webhooks ===
WEBHOOK_GLOBAL_URL=
WEBHOOK_GLOBAL_ENABLED=false

# === Integraciones (opcionales) ===
# TYPEBOT_URL=https://bot.meuservidor.com
# CHATWOOT_URL=https://atendimento.meuservidor.com
# DIFY_URL=https://ia.meuservidor.com
```

### 4. Iniciar

```bash
docker compose up -d
```

### 5. Verificar

```bash
docker compose ps
docker compose logs -f api
```

Aguardar: `Server is running on port 8080`

### 6. Acceder al panel

- **API**: `http://192.168.1.100:8080`
- **Manager**: `http://192.168.1.100:3000`

## Configurar dominio en Cloudflare Tunnel

```bash
nano ~/.cloudflared/config.yml
```

```yaml
  - hostname: api.evolution.meuservidor.com
    service: http://localhost:8080
  - hostname: manager.evolution.meuservidor.com
    service: http://localhost:3000
```

```bash
sudo systemctl restart cloudflared
```

## Diferencias principales respecto a evolution-go

### 1. Integración nativa con Typebot

En evolution-api, Typebot se configura directamente en los endpoints, sin necesidad de n8n:

```bash
curl -X POST http://localhost:8080/typebot/create/meu-whatsapp \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer SUA_API_KEY" \
  -d '{
    "typebot": {
      "url": "https://bot.meuservidor.com",
      "typebot": "id-do-seu-typebot",
      "startSession": true
    }
  }'
```

### 2. Panel de gestión

Evolution-manager ofrece interfaz web para:
- Gestionar instancias (crear, eliminar, conectar QR Code)
- Configurar webhooks
- Visualizar Estado
- Gestionar licencias

### 3. Multi-proveedor

Soporta simultáneamente:
- **Baileys**: WhatsApp Web (gratuito, como evolution-go)
- **Meta Cloud API**: WhatsApp Business API oficial (requiere cuenta Meta Business)

## Próximo paso

Volver a [Índice de Aplicaciones](./README.md) o [Chatbot Multicanal](../10-chatbot-multicanal/README.md).
