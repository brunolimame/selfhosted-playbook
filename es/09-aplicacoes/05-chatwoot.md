# 09-05 - Chatwoot

## ¿Qué es?

[Chatwoot](https://www.chatwoot.com) es una plataforma open-source de atención al cliente (helpdesk). Es la alternativa self-hosted a Intercom, Zendesk y Freshdesk.

Funcionalidades principales:
- Bandeja de entrada unificada (WhatsApp, Email, Webchat, Telegram, etc.)
- Atención multiagente con asignación de conversaciones
- Respuestas predefinidas (macros/canned responses)
- Notas internas y menciones entre agentes
- Reportes y analytics
- API y webhooks para integraciones
- Automatización de reglas (auto-assign, labels, etc.)

## Arquitectura

```
chatwoot-web (contenedor)    -> Puerto 3000 (interfaz web + API)
chatwoot-worker (contenedor) -> Sidekiq (jobs en background)
     |
     +-- PostgreSQL (conversaciones, contactos, agentes)
     +-- Redis (caché, colas de jobs)
```

## ¿Por qué Chatwoot con Evolution Go y Typebot?

Chatwoot es el eslabón humano de la automatización de atención. El flujo completo:

```
Cliente envía mensaje en WhatsApp
     |
evolution-go recibe el webhook
     |
Typebot intenta resolver automáticamente
     |
     +-- Si cliente necesita humano -> evolution-go deriva a Chatwoot
     |
Chatwoot muestra al agente disponible
     |
Agente responde -> Chatwoot envía a evolution-go -> evolution-go entrega en WhatsApp
```

## Prerrequisitos

- Docker y Docker Compose instalados en la VM
- Al menos 2 GB RAM libres (Chatwoot es pesado)
- Dominio configurado: `atendimento.meuservidor.com`
- Cuenta SMTP para correos transaccionales (opcional en la instalación inicial)

## Instalación

### 1. Acceder a la VM y crear directorio

```bash
ssh ubuntu@192.168.1.100
mkdir -p ~/chatwoot
cd ~/chatwoot
```

### 2. Descargar el docker-compose de producción

```bash
wget -O docker-compose.yml https://raw.githubusercontent.com/chatwoot/chatwoot/develop/docker-compose.production.yaml
```

### 3. Crear archivo .env

```bash
nano .env
```

```env
# === General ===
INSTALLATION_NAME=Mi Atención

# === Puerto ===
PORT=3000

# === PostgreSQL ===
POSTGRES_USER=chatwoot
POSTGRES_PASSWORD=senha-forte-aqui
POSTGRES_DATABASE=chatwoot

# === Redis ===
REDIS_PASSWORD=redis-senha-aqui

# === URL base (dominio público) ===
FRONTEND_URL=https://atendimento.meuservidor.com

# === Clave secreta (generar con: openssl rand -hex 64) ===
SECRET_KEY_BASE=coloca-una-clave-hex-de-64-caracteres-aqui

# === Clave de cifrado (generar con: openssl rand -hex 32) ===
ENCRYPTION_PRIMARY_KEY=coloca-una-clave-hex-de-32-caracteres-aqui

# === SMTP (para enviar correos de notificación) ===
# SMTP_ADDRESS=smtp.gmail.com
# SMTP_PORT=587
# SMTP_USERNAME=tu-email@gmail.com
# SMTP_PASSWORD=tu-contraseña
# SMTP_AUTH_METHOD=plain
# SMTP_ENABLE_STARTTLS_AUTO=true
# SMTP_DOMAIN=gmail.com

# === Almacenamiento (opcional - usa MinIO) ===
# ACTIVE_STORAGE_SERVICE=local
# STORAGE_DIR=/app/storage

# === Idioma por defecto ===
DEFAULT_LOCALE=pt_BR
```

### 4. Generar las claves

```bash
echo "SECRET_KEY_BASE=$(openssl rand -hex 64)"
echo "ENCRYPTION_PRIMARY_KEY=$(openssl rand -hex 32)"
```

Copia los valores en el `.env`.

### 5. Ajustar el docker-compose.yml

Edita para usar los puertos correctos:

```bash
nano docker-compose.yml
```

En la sección del servicio `chatwoot`, ajusta los puertos:

```yaml
  chatwoot:
    ports:
      - "${PORT}:3000"
```

Y en el `depends_on`, cambia para usar los nombres de los servicios correctos.

### 6. Preparar la base de datos

```bash
docker compose run --rm chatwoot bundle exec rails db:chatwoot_prepare
```

Este comando:
- Crea las tablas en PostgreSQL
- Ejecuta las migrations
- Siembra datos iniciales

### 7. Iniciar

```bash
docker compose up -d
```

### 8. Verificar

```bash
docker compose ps
docker compose logs -f chatwoot
```

Espera hasta ver: `Listening on http://0.0.0.0:3000`

### 9. Crear cuenta admin

Accede a `http://192.168.1.100:3000` y crea la cuenta de administrador.

## Configurar dominio en Cloudflare Tunnel

Edita el config.yml:

```bash
nano ~/.cloudflared/config.yml
```

Agrega:

```yaml
  - hostname: atendimento.meuservidor.com
    service: http://localhost:3000
```

Reinicia:

```bash
sudo systemctl restart cloudflared
```

## Integrar con Evolution Go

### 1. En Chatwoot

1. Ve a **Settings > Inboxes > Add Inbox**
2. Selecciona **Evolution API**
3. Configura:
   - **Name**: WhatsApp Comercial
   - **Evolution API URL**: `http://192.168.1.100:4000`
   - **Global API Key**: (tu clave de evolution-go)
   - **Instance Name**: (nombre de la instancia en evolution-go)
4. Haz clic en **Create Inbox**

### 2. En evolution-go

Configura el webhook para enviar mensajes a Chatwoot:

```bash
curl -X POST http://192.168.1.100:4000/webhook/create/mi-whatsapp \
  -H "Content-Type: application/json" \
  -H "apiKey: TU_GLOBAL_API_KEY" \
  -d '{
    "webhook": {
      "url": "https://atendimento.meuservidor.com/webhooks/evolution/TU-WEBHOOK-TOKEN",
      "events": ["message.upsert", "message.update", "connection.update"]
    }
  }'
```

El token del webhook se genera automáticamente al crear el inbox en Chatwoot.

## Flujo completo (Typebot + Chatwoot)

Para configurar el enrutamiento automático donde Typebot intenta resolver y Chatwoot asume cuando es necesario:

1. **Typebot**: crea un flujo con bloque "Set Variable" definiendo `transferir=false`
2. **Typebot**: si el cliente pide "agente" o "humano", cambia `transferir=true`
3. **Typebot**: bloque "Webhook" envía a n8n con la flag `transferir`
4. **n8n**: si `transferir=true`, llama a la API de Chatwoot para crear conversación y notificar al agente
5. **n8n**: envía mensaje en WhatsApp vía evolution-go: "Serás atendido en instantes"

Ejemplo de webhook n8n para crear conversación en Chatwoot:

```bash
curl -X POST https://atendimento.meuservidor.com/api/v1/accounts/1/conversations \
  -H "Content-Type: application/json" \
  -H "api_access_token: TU-TOKEN-DE-CHATWOOT" \
  -d '{
    "source_id": "5511999999999@s.whatsapp.net",
    "inbox_id": 1,
    "contact_id": 1,
    "status": "pending"
  }'
```

## Mantenimiento

### Actualizar

```bash
cd ~/chatwoot
docker compose pull
docker compose down
docker compose up -d
```

Después de actualizar, ejecuta las migrations si es necesario:

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

# Backup de la base
docker compose exec -T postgres pg_dump -U chatwoot chatwoot > $BACKUP_DIR/chatwoot-db-$DATE.sql
gzip $BACKUP_DIR/chatwoot-db-$DATE.sql

# Backup de archivos
docker run --rm -v chatwoot_data:/source -v $BACKUP_DIR:/backup alpine tar czf /backup/chatwoot-storage-$DATE.tar.gz -C /source .

echo "Backup completado: $DATE"
```

## Troubleshooting

| Error | Causa | Solución |
|-------|-------|----------|
| `500 Internal Server Error` | SECRET_KEY_BASE inválido | Genera nueva clave con `openssl rand -hex 64` |
| PostgreSQL connection refused | Postgres no inició | `docker compose logs postgres` |
| Correos no salen | SMTP no configurado | Configura SMTP en el `.env` y reinicia |
| Evolution inbox no conecta | URL o API Key incorrecta | Verifica el endpoint de evolution-go |
| Pantalla blanca en login | `FRONTEND_URL` incorrecta | Usa la URL exacta con https |

## Siguiente paso

[Dify](./06-dify.md) - IA con base de conocimiento (RAG).
