# 09-07 - MinIO

## O que es?

[MinIO](https://min.io) es un servidor de almacenamiento de objetos de alto rendimiento, compatible con la API S3 de Amazon. Es la alternativa self-hosted a Amazon S3.

Funcionalidades principales:
- API 100% compatible con S3 (use SDKs y herramientas S3 existentes)
- Consola web para gestión de buckets y archivos
- Alto rendimiento (escrito en Go, ~50 MB por contenedor)
- Soporte a erasure coding para protección de datos
- Multi-tenant con buckets y políticas de acceso
- Compatible con evolution-go, Typebot, Dify y Chatwoot para subida de medios

## Arquitectura

```
minio (contenedor único - single node)
    Puerto 9000 (API S3)
    Puerto 9001 (Consola web)
    |
    +-- /data (volumen persistente)
```

## ¿Por qué MinIO en el ecosistema de automatización?

Varias aplicaciones necesitan almacenamiento S3 para archivos multimedia:

| Aplicación | Uso de MinIO |
|-----------|-------------|
| **Evolution Go** | Fotos, audios, videos y documentos enviados/recibidos en WhatsApp |
| **Typebot** | Subida de imágenes y archivos en los flujos del chatbot |
| **Dify** | Documentos de la base de conocimiento y archivos enviados por el usuario |
| **Chatwoot** | Archivos adjuntos en las conversaciones de atención |
| **n8n** | Almacenamiento intermedio de archivos en workflows |

## Prerrequisitos

- Docker y Docker Compose instalados en la VM
- Disco con espacio suficiente para los archivos multimedia
- Dominio para consola: `minio.meuservidor.com` (opcional)

## Instalación

### 1. Acceder a la VM y crear directorio

```bash
ssh ubuntu@192.168.1.100
mkdir -p ~/minio
cd ~/minio
```

### 2. Crear docker-compose.yml

```bash
nano docker-compose.yml
```

```yaml
services:
  minio:
    image: quay.io/minio/minio:latest
    container_name: minio
    restart: unless-stopped
    command: server /data --console-address ":9001"
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - minio_data:/data
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minio-senha-forte-aqui
      MINIO_BROWSER_REDIRECT_URL: https://minio.meuservidor.com
      MINIO_SERVER_URL: https://s3.meuservidor.com
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  minio_data:
```

### 3. Iniciar

```bash
docker compose up -d
```

### 4. Verificar

```bash
docker compose ps
docker compose logs -f
```

### 5. Acceder a la consola

- **API S3**: `http://192.168.1.100:9000`
- **Consola web**: `http://192.168.1.100:9001`
- **Usuario**: `minioadmin`
- **Contraseña**: `minio-senha-forte-aqui`

## Configurar dominio en Cloudflare Tunnel

Edite el config.yml:

```bash
nano ~/.cloudflared/config.yml
```

Añada:

```yaml
  - hostname: minio.meuservidor.com
    service: http://localhost:9001

  - hostname: s3.meuservidor.com
    service: http://localhost:9000
```

Reinicie:

```bash
sudo systemctl restart cloudflared
```

## Crear buckets y claves de acceso

### Desde la consola web

1. Acceda a `http://192.168.1.100:9001`
2. Inicie sesión con `minioadmin` / `minio-senha-forte-aqui`
3. **Buckets > Create Bucket**:
   - `evolution-media` (medios de WhatsApp)
   - `typebot` (subidas de Typebot)
   - `dify` (documentos de la base de conocimiento)
   - `chatwoot` (archivos adjuntos de Chatwoot)
4. **Access Keys > Create Access Key**:
   - Anote el `Access Key` y `Secret Key`

### Desde la línea de comandos (mc client)

```bash
# Instalar mc client
docker compose exec minio mc alias set local http://localhost:9000 minioadmin minio-senha-forte-aqui

# Crear buckets
docker compose exec minio mc mb local/evolution-media
docker compose exec minio mc mb local/typebot
docker compose exec minio mc mb local/dify
docker compose exec minio mc mb local/chatwoot

# Listar buckets
docker compose exec minio mc ls local
```

## Configurar políticas de acceso (opcional)

Para buckets que necesitan ser públicos (ej: imágenes de perfil de WhatsApp):

```bash
docker compose exec minio mc anonymous set download local/evolution-media
```

## Integrar con las aplicaciones

### Evolution Go

En el `.env` de evolution-go:

```env
MINIO_ENABLED=true
MINIO_ENDPOINT=minio:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minio-senha-forte-aqui
MINIO_BUCKET=evolution-media
MINIO_SSL=false
```

### Typebot

En el `.env` de Typebot:

```env
S3_ACCESS_KEY=minioadmin
S3_SECRET_KEY=minio-senha-forte-aqui
S3_BUCKET=typebot
S3_ENDPOINT=http://minio:9000
S3_SSL=false
```

### Dify

En el `.env` de Dify (en `~/dify/docker/.env`):

```env
STORAGE_TYPE=s3
S3_ENDPOINT=http://minio:9000
S3_REGION=us-east-1
S3_BUCKET_NAME=dify
S3_ACCESS_KEY=minioadmin
S3_SECRET_KEY=minio-senha-forte-aqui
```

### Chatwoot

En el `.env` de Chatwoot:

```env
ACTIVE_STORAGE_SERVICE=amazon
STORAGE_S3_ENDPOINT=http://minio:9000
STORAGE_S3_BUCKET=chatwoot
AWS_ACCESS_KEY_ID=minioadmin
AWS_SECRET_ACCESS_KEY=minio-senha-forte-aqui
AWS_REGION=us-east-1
```

**Importante**: Para que los contenedores se comuniquen entre sí, use el nombre del servicio (`minio`) en lugar de `localhost` o `192.168.1.100` en el endpoint, siempre que estén en la misma red Docker.

Si las aplicaciones están en redes Docker separadas, use la IP de la VM (`192.168.1.100`) en el endpoint.

## Mantenimiento

### Actualizar

```bash
cd ~/minio
docker compose pull
docker compose up -d
```

### Logs

```bash
docker compose logs -f
```

### Backup

```bash
#!/bin/bash
# ~/minio/backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR=~/backups/minio
mkdir -p $BACKUP_DIR

# Backup de los datos
docker run --rm -v minio_data:/source -v $BACKUP_DIR:/backup alpine tar czf /backup/minio-data-$DATE.tar.gz -C /source .

# Backup de la configuración (buckets, políticas)
docker compose exec minio mc alias set local http://localhost:9000 minioadmin minio-senha-forte-aqui
docker compose exec minio mc admin config get local > $BACKUP_DIR/minio-config-$DATE.txt

echo "Backup completado: $DATE"
```

### Restaurar

```bash
docker run --rm -v minio_data:/dest -v $BACKUP_DIR:/backup alpine tar xzf /backup/minio-data-20260101.tar.gz -C /dest
docker compose restart
```

### Limpieza de archivos antiguos (opcional)

```bash
# Script para eliminar medios con más de 90 días
docker compose exec minio mc find local/evolution-media --older-than 90d --exec "mc rm {}"
```

## Troubleshooting

| Error | Causa | Solución |
|------|-------|---------|
| `Connection refused` | Contenedor no inició | `docker compose logs minio` |
| `Access Denied` | Credenciales incorrectas | Verificar `MINIO_ROOT_USER`/`MINIO_ROOT_PASSWORD` |
| Bucket ya existe | Nombre duplicado | Usar otro nombre |
| S3 incompatible | Endpoint incorrecto | Usar `http://minio:9000` (Docker) o `http://192.168.1.100:9000` (host) |
| Disco lleno | Muchos medios almacenados | Configurar política de retención/limpieza |

## Próximo paso

[Seguridad](../07-seguranca.md) - Proteja su servidor.
