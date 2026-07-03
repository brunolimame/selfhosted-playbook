# 09-11 - Uptime Kuma

## ¿Qué es?

[Uptime Kuma](https://github.com/louislam/uptime-kuma) es una herramienta de monitoreo de uptime auto-hospedada, bonita y fácil de usar. Es la alternativa open-source a Uptime Robot, Pingdom y StatusCake.

Funcionalidades principales:
- Monitoreo HTTP(s), TCP, Ping, DNS, WebSocket, Docker Containers
- Notificaciones via Telegram, Discord, Email, WhatsApp y 90+ servicios
- Status page pública (comparta con clientes)
- Intervalo de 20 segundos
- Multi-idioma (incluyendo español)
- Gráficos de uptime, latencia y certificado SSL
- API para consulta de estado
- Backup y restore con un clic

## ¿Por qué Uptime Kuma en este proyecto?

Con decenas de servicios funcionando, necesita saber cuándo algo se cae:

| Servicio | Monitorear | URL de ejemplo |
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

## Arquitectura

```
uptime-kuma (contenedor) -> Puerto 3001
    |
    +-- SQLite (base de datos interna)
    +-- Notificaciones (Telegram, Discord, Email...)
    +-- Status page (opcional, pública)
```

## Prerrequisitos

- Docker instalado en la VM
- Mínimo 256 MB RAM libres

## Instalación

### 1. Acceder a la VM

```bash
ssh ubuntu@192.168.1.100
```

### 2. Crear directorio

```bash
mkdir -p ~/uptime-kuma
cd ~/uptime-kuma
```

### 3. Crear docker-compose.yml

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

## Configuración inicial

1. Acceda a `http://192.168.1.100:3001`
2. Cree el usuario admin (nombre, email, contraseña)
3. Seleccione **SQLite** como base de datos

## Configurar dominio en Cloudflare Tunnel

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

## Añadir monitores

### Monitorear servicios HTTP

Haga clic en **Add Monitor** y configure:

```
Monitor Type: HTTP(s)
Name: Typebot - Builder
URL: https://bot.meuservidor.com
Interval: 30s
Resend Notification: 3 veces
Notification: Telegram (configurar)
```

### Monitorear servicios TCP (base de datos)

```
Monitor Type: TCP Port
Name: PostgreSQL
Hostname: 192.168.1.100
Port: 5432
Interval: 60s
```

### Monitorear ping (internet)

```
Monitor Type: Ping
Name: Internet - Google DNS
Hostname: 8.8.8.8
Interval: 60s
```

### Monitorear certificado SSL

```
Monitor Type: HTTP(s)
Name: SSL - bot.meuservidor.com
URL: https://bot.meuservidor.com
Resend Notification: 1x por día (solo si cambia)
```

## Configurar notificaciones

### Telegram

1. Haga clic en **Settings > Notifications > Add Notification**
2. Tipo: **Telegram**
3. Bot Token: (token de @BotFather)
4. Chat ID: (obtener con `@userinfobot` o enviar `/start` al bot y luego `https://api.telegram.org/botTOKEN/getUpdates`)
5. Pruebe la notificación

### Discord

1. Tipo: **Discord**
2. Webhook URL: (crear en Discord: Configuración del Canal > Integraciones > Webhooks)
3. Pruebe

### Email (SMTP)

1. Tipo: **SMTP**
2. Host, Port, User, Pass según su proveedor de email
3. Pruebe

## Crear Status Page pública

1. Haga clic en **Status Page > Add Status Page**
2. Slug: `status` (quedará en `https://status.meuservidor.com/status`)
3. Title: `Estado de los Servicios`
4. Seleccione los monitores a mostrar
5. Active **Publish**
6. Comparta el enlace con clientes: `https://status.meuservidor.com/status`

## Mantenimiento

### Actualizar

```bash
cd ~/uptime-kuma
docker compose pull
docker compose up -d
```

### Backup

```bash
# Backup completo de SQLite
docker run --rm -v uptime_kuma_data:/source -v ~/backups:/backup alpine tar czf /backup/uptime-kuma-$(date +%Y%m%d).tar.gz -C /source .
```

### Exportar/Importar monitores

En el panel: **Settings > Backup > Create Backup** (archivo JSON con toda la configuración).

## Troubleshooting

| Error | Causa | Solución |
|------|-------|---------|
| Monitor muestra "down" | Servicio realmente fuera | Verificar que el servicio esté funcionando |
| Notificación no llega | Token/configuración incorrecta | Probar configuración en la pantalla de notificaciones |
| SSL certificate expired | Certificado expiró | Verificar Cloudflare o certbot |
| Timeout en la petición | Servicio lento o firewall | Aumentar timeout en el monitor a 30s |
| Backup corrupto | SQLite corrupto | Parar el contenedor, copiar `kuma.db`, ejecutar `sqlite3 kuma.db .dump` |

## Próximo paso

[Netdata](./12-netdata.md) - Monitoreo en tiempo real de la VM.
