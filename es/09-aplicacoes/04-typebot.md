# 09-04 - Typebot

## ¿Qué es?

[Typebot](https://typebot.io) es una plataforma open-source para crear chatbots conversacionales con flujos visuales (no-code). Es la alternativa self-hosted a Landbot, ManyChat y Chatfuel.

Funcionalidades principales:
- Crear flujos de conversación con bloques arrastrables
- Soporte para texto, botones, imágenes, videos, formularios y condiciones
- Integración nativa con **Evolution Go** (WhatsApp)
- Webhooks para conectar con n8n y otras herramientas
- Programación y analytics
- Carga de archivos con almacenamiento S3/MinIO
- Multi-idioma

## Arquitectura

```
typebot-builder (contenedor)  ->  Puerto 3001 (creación de flujos)
typebot-viewer (contenedor)   ->  Puerto 3002 (ejecución de los bots)
     |
     +-- PostgreSQL (datos, flujos, usuarios)
     +-- (Opcional) MinIO/S3 (carga de archivos)
```

## ¿Por qué Typebot con Evolution Go?

Typebot tiene integración nativa con el ecosistema Evolution. El flujo de atención queda:

```
Cliente envía mensaje en WhatsApp
     |
evolution-go recibe el webhook
     |
Typebot ejecuta el flujo configurado
     |
     +-- Si respuesta automática -> Typebot responde vía API de evolution-go
     +-- Si necesita humano -> activa Chatwoot o n8n
```

## Prerrequisitos

- Docker y Docker Compose instalados en la VM
- Al menos 1 GB RAM libre
- Un dominio para el builder (ej: `bot.meuservidor.com`)
- Un dominio para el viewer (ej: `viewer.meuservidor.com`)

## Instalación

### 1. Acceder a la VM y crear directorio

```bash
ssh ubuntu@192.168.1.100
mkdir -p ~/typebot
cd ~/typebot
```

### 2. Descargar archivos de configuración

```bash
wget https://raw.githubusercontent.com/baptisteArno/typebot.io/latest/docker-compose.yml
wget https://raw.githubusercontent.com/baptisteArno/typebot.io/latest/.env.example -O .env
```

### 3. Generar clave de cifrado

```bash
openssl rand -base64 32 | tr -d '\n' ; echo
```

Copia el resultado para usarlo en el `.env`.

### 4. Configurar variables de entorno

```bash
nano .env
```

```env
# Database
DATABASE_URL=postgresql://postgres:typebot@postgres:5432/typebot

# Cifrado (usa el valor generado en el paso anterior)
ENCRYPTION_SECRET=coloca-la-clave-de-32-caracteres-aqui

# URL del Builder (accesible públicamente)
NEXTAUTH_URL=https://bot.meuservidor.com
NEXT_PUBLIC_VIEWER_URL=https://viewer.meuservidor.com

# Auth providers (al menos uno obligatorio)
# Email (magic link)
NEXT_PUBLIC_SMTP_FROM_EMAIL=noreply@meuservidor.com
SMTP_URL=smtp://user:pass@smtp.tuproveedor.com:587
NEXT_PUBLIC_SMTP_NAME=Typebot

# Google OAuth (opcional)
# GOOGLE_CLIENT_ID=...
# GOOGLE_CLIENT_SECRET=...

# GitHub OAuth (opcional)
# GITHUB_CLIENT_ID=...
# GITHUB_CLIENT_SECRET=...

# Puertos (ajusta si hay conflicto)
BUILDER_PORT=3001
VIEWER_PORT=3002

# S3 / MinIO para carga de archivos (opcional)
# S3_ACCESS_KEY=minioadmin
# S3_SECRET_KEY=minioadmin
# S3_BUCKET=typebot
# S3_ENDPOINT=http://192.168.1.100:9000
# S3_SSL=false
```

### 5. Ajustar docker-compose.yml

Edita el archivo descargado para ajustar los puertos:

```bash
nano docker-compose.yml
```

Cambia las secciones de puerto a:

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

## Configurar dominio en Cloudflare Tunnel

Edita el config.yml:

```bash
nano ~/.cloudflared/config.yml
```

Agrega antes del catch-all:

```yaml
  - hostname: bot.meuservidor.com
    service: http://localhost:3001

  - hostname: viewer.meuservidor.com
    service: http://localhost:3002
```

Reinicia:

```bash
sudo systemctl restart cloudflared
```

## Configurar archivos multimedia con MinIO (opcional)

Si tienes MinIO instalado ([guía](./07-minio.md)), crea un bucket:

```bash
docker compose exec minio mc alias set local http://localhost:9000 minioadmin minioadmin
docker compose exec minio mc mb local/typebot
```

Y en el `.env` de Typebot:
```env
S3_ACCESS_KEY=minioadmin
S3_SECRET_KEY=minioadmin
S3_BUCKET=typebot
S3_ENDPOINT=http://minio:9000
S3_SSL=false
```

Reinicia Typebot después de cambiar:
```bash
docker compose down && docker compose up -d
```

## Integrar con Evolution Go

Para conectar Typebot a evolution-go y responder WhatsApp automáticamente:

### 1. Crear el bot en Typebot

1. Accede a `https://bot.meuservidor.com`
2. Crea tu cuenta
3. Crea un nuevo Typebot
4. Arma el flujo de conversación deseado
5. Publica el bot y copia el **ID público** (en la URL: `/typebots/[ID]/...`)

### 2. Configurar webhook en evolution-go

En evolution-go, crea la instancia con Typebot integrado:

```bash
curl -X POST http://192.168.1.100:4000/typebot/create/mi-whatsapp \
  -H "Content-Type: application/json" \
  -H "apiKey: TU_GLOBAL_API_KEY" \
  -d '{
    "typebot": {
      "url": "https://viewer.meuservidor.com",
      "name": "mi-bot-de-atencion",
      "typebotId": "ID-DE-TU-TYPEBOT",
      "startSession": true,
      "trigger": {
        "type": "keyword",
        "value": "!bot"
      }
    }
  }'
```

Ahora, cuando alguien envíe "!bot" en WhatsApp, Typebot iniciará el flujo automáticamente.

### Variables predefinidas

evolution-go envía automáticamente a Typebot:
- `remoteJid` - ID del contacto en WhatsApp
- `pushName` - Nombre del contacto
- `instanceName` - Nombre de la instancia
- `serverUrl` - URL del servidor evolution-go
- `apiKey` - Clave de la API

## Mantenimiento

### Actualizar

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
# Backup de la base
docker compose exec postgres pg_dump -U postgres typebot > ~/backups/typebot-$(date +%Y%m%d).sql
```

## Troubleshooting

| Error | Causa | Solución |
|-------|-------|----------|
| `ENCRYPTION_SECRET` inválido | Clave demasiado corta | Debe tener exactamente 32 caracteres |
| `NEXTAUTH_URL` no configurado | URL del builder ausente | Configura `NEXTAUTH_URL` en el `.env` |
| Viewer no carga | `NEXT_PUBLIC_VIEWER_URL` incorrecto | Verifica la URL pública del viewer |
| Error al enviar email | SMTP no configurado | Configura SMTP o usa OAuth |
| Carga de archivos falla | S3/MinIO no configurado | Configura S3 o deshabilita la carga |

## Siguiente paso

[Chatwoot](./05-chatwoot.md) - Helpdesk para atención humana.
