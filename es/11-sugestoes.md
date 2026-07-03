# 11 - Sugerencias de Proximas Aplicaciones

Con base en el ecosistema ya documentado, aqui estan sugerencias de que anadir a continuacion, organizadas por categoria.

## Monitoreo y Observabilidad

### Uptime Kuma
Monitor de uptime con dashboard bonito, notificaciones y status page publica.

```
docker compose:
  - port: 3001
  - image: louislam/uptime-kuma
  - depende de: nada
```

Por que anadirlo: Monitorea todas las aplicaciones de la VM (Coolify, Typebot, n8n, evolution-go) y envia alerta si algo cae.

### Netdata
Monitoreo en tiempo real de CPU, RAM, disco, red de toda la VM.

```
docker compose:
  - port: 19999
  - image: netdata/netdata
  - depende de: nada
```

Por que anadirlo: Visibilidad completa de los recursos de la VM, esencial para saber si falta RAM o CPU.

### Sentry (self-hosted)
Rastreo de errores en aplicaciones. Capture excepciones de Typebot, n8n, Dify.

```
docker compose:
  - port: 9000
  - image: getsentry/sentry
  - depende de: postgres, redis
```

Por que anadirlo: Saber cuando una aplicacion se rompió y por que, con stack trace completo.

## Base de Datos

### pgAdmin
Interfaz web para administrar PostgreSQL. Util para todas las apps que usan Postgres.

```
docker compose:
  - port: 5050
  - image: dpage/pgadmin4
```

### Redis Commander
Interfaz web para administrar Redis (usado por Dify, Chatwoot, evolution-api).

```
docker compose:
  - port: 8081
  - image: rediscommander/redis-commander
```

## CI/CD y Git

### Gitea / Forgejo
Servidor Git auto-hospedado (como GitHub). Permite hospedar repositorios privados e integra con Coolify.

```
docker compose:
  - port: 3000
  - image: gitea/gitea
  - depende de: postgres
```

Por que anadirlo: Hospedar los repositorios de las aplicaciones localmente, integrando con Coolify para deploy automatico.

### Woodpecker CI
Pipeline CI/CD ligero que integra con Gitea. Ejecuta tests y deploy automaticamente.

```
docker compose:
  - port: 8000
  - image: woodpeckerci/woodpecker-server
  - depende de: postgres, gitea
```

## Comunicacion

### Mattermost
Chat de equipo auto-hospedado (alternativa a Slack). Integra con n8n para notificaciones.

```
docker compose:
  - port: 8065
  - image: mattermost/mattermost
  - depende de: postgres
```

## DNS y Proxy

### AdGuard Home
Bloqueo de anuncios y rastreadores a nivel de DNS para toda la red.

```
docker compose:
  - port: 80/3000
  - image: adguard/adguardhome
```

### Nginx Proxy Manager
Interfaz web para gestionar proxies inversos y certificados SSL. Alternativa mas simple a Cloudflare Tunnel.

```
docker compose:
  - port: 80/81/443
  - image: jc21/nginx-proxy-manager
  - depende de: nada
```

## Backup

### Duplicati
Backup automatico con cifrado para la nube (Google Drive, S3, etc.).

```
docker compose:
  - port: 8200
  - image: linuxserver/duplicati
```

### BorgBackup + Borgmatic
Backup eficiente con deduplicacion y compresion.

```bash
sudo apt install borgmatic
```

## Resumen por prioridad

| Prioridad | App | Motivo |
|-----------|-----|--------|
| Alta | **Uptime Kuma** | Saber si los servicios estan en linea |
| Alta | **Netdata** | Monitorear recursos de la VM |
| Media | **Gitea** | Hospedar repositorios localmente |
| Media | **pgAdmin** | Gestionar bases PostgreSQL |
| Baja | **AdGuard Home** | Bloquear anuncios en la red |
| Baja | **Duplicati** | Backup a la nube |

## Proximo paso

[Herramientas de Desarrollo](./herramientas-dev.md) - Herramientas y servicios utiles para desarrollo.
