# 09-06 - Dify

## ¿Qué es?

[Dify](https://dify.ai) es una plataforma open-source para creación de aplicaciones con IA generativa. Permite construir chatbots inteligentes con base de conocimiento propia (RAG), workflows de IA y agentes autónomos.

Funcionalidades principales:
- **RAG (Retrieval-Augmented Generation)**: Conecta documentos (PDF, TXT, HTML, etc.) y haz preguntas sobre ellos
- **Chatbot con contexto**: Carga de documentos como base de conocimiento
- **Workflow de IA**: Crea pipelines con nodos visuales (LLM, herramientas, lógica)
- **Agentes**: Agentes autónomos con herramientas (web search, API, cálculo)
- **API completa**: Integra con cualquier sistema (evolution-go, n8n, Typebot)
- **Multimodelo**: OpenAI, Anthropic, Ollama (local), modelos open-source

## Arquitectura

```
dify (multi-contenedores)
    |
    +-- api (backend Flask/Python)      -> Puerto 5001
    +-- worker (procesamiento en cola)
    +-- web (frontend Next.js)          -> Puerto 3000
    +-- nginx (proxy reverso interno)   -> Puerto 80
    |
    +-- PostgreSQL (datos de la aplicación)
    +-- Redis (caché/colas)
    +-- Weaviate (base de datos vectorial para RAG)
```

## ¿Por qué Dify con Evolution Go?

Dify aporta inteligencia a la atención:

```
Cliente pregunta: "¿Cuál es el horario de atención?"
     |
evolution-go recibe el mensaje
     |
n8n reenvía a Dify
     |
Dify consulta base de conocimiento (PDF con FAQ)
     |
LLM responde con contexto de los documentos
     |
n8n devuelve respuesta a evolution-go
     |
evolution-go envía mensaje en WhatsApp
```

## Prerrequisitos

- Docker y Docker Compose 2.24+ instalados en la VM
- Al menos 4 GB RAM libres (Dify + Weaviate consumen memoria)
- CPU 2+ núcleos
- Dominio configurado: `ia.meuservidor.com`
- (Opcional) API key de un proveedor LLM (OpenAI, Anthropic)

## Instalación

### 1. Acceder a la VM y clonar el repositorio

```bash
ssh ubuntu@192.168.1.100
cd ~
git clone https://github.com/langgenius/dify.git
cd dify
```

### 2. Configurar variables de entorno

```bash
cd docker
cp .env.example .env
nano .env
```

Variables esenciales:

```env
# Modo de deploy
DEPLOY_ENV=PRODUCTION

# Puertos (evitar conflicto con otras apps)
EXPOSE_NGINX_PORT=80
EXPOSE_NGINX_SSL_PORT=443

# Clave secreta (generar con: openssl rand -hex 32)
SECRET_KEY=tu-clave-secreta-aqui

# Base de datos
DB_USERNAME=postgres
DB_PASSWORD=dify-senha
DB_DATABASE=dify
DB_PORT=5432

# Redis
REDIS_PASSWORD=dify-redis-senha

# Weaviate (vector store)
WEAVIATE_AUTHENTICATION_ANONYMOUS_ACCESS_ENABLED=false
WEAVIATE_AUTHENTICATION_APIKEY_ENABLED=true
WEAVIATE_AUTHENTICATION_APIKEY_ALLOWED_KEYS=dify-weaviate-key
WEAVIATE_AUTHENTICATION_APIKEY_USERS=admin

# Inicialización del admin
INIT_PASSWORD=senha-admin-super-forte
INIT_EMAIL=admin@meuservidor.com
```

### 3. Ajustar docker-compose.yml (puertos)

Si Coolify ya está usando el puerto 80, edita el `docker-compose.yml`:

```bash
nano docker-compose.yml
```

Cambia los puertos del nginx:

```yaml
  nginx:
    ports:
      - "8080:80"
```

Esto hará que Dify funcione en `http://localhost:8080`.

### 4. Iniciar

```bash
cd ~/dify/docker
docker compose up -d
```

La primera vez puede demorar (descarga de varias imágenes).

### 5. Verificar

```bash
docker compose ps
```

Deben estar funcionando: `api`, `worker`, `web`, `nginx`, `db`, `redis`, `weaviate`, `ssrf_proxy`, `sandbox`, `plugin_daemon`.

### 6. Acceder y configurar admin

Accede a `http://192.168.1.100:8080/install` y sigue el flujo de configuración:
1. Define email y contraseña del admin (o usa `INIT_EMAIL`/`INIT_PASSWORD` del `.env`)
2. Selecciona el proveedor LLM por defecto (OpenAI, Anthropic, o configúralo después)

## Configurar dominio en Cloudflare Tunnel

Edita el config.yml:

```bash
nano ~/.cloudflared/config.yml
```

Agrega:

```yaml
  - hostname: ia.meuservidor.com
    service: http://localhost:8080
```

Reinicia:

```bash
sudo systemctl restart cloudflared
```

## Configurar proveedor LLM

Ve a **Settings > Model Provider** y agrega al menos un proveedor:

### OpenAI
```env
OPENAI_API_KEY=sk-proj-tu-token
```

### Ollama (modelos locales)
Si tienes Ollama instalado en la VM:
```env
OLLAMA_BASE_URL=http://192.168.1.100:11434
```

Luego ve a **Settings > Model Provider > Ollama** y configúralo.

## Crear una base de conocimiento

Para usar RAG con tus documentos:

1. Ve a **Knowledge > Create Knowledge**
2. Selecciona **Upload Files**
3. Sube PDFs, TXTs u otros documentos (ej: FAQ, manual, catálogo)
4. Elige el método de indexación (recomendado: **High Quality** con Weaviate)
5. Espera el procesamiento
6. Ve a **Studio > Create App > Chatbot**
7. En **Context**, selecciona la base de conocimiento creada
8. Publica la app

## Integrar con n8n + Evolution Go

Para conectar Dify, n8n y evolution-go:

### En Dify

1. Crea una app de tipo **Chatbot** con base de conocimiento
2. Ve a **API Access** y copia la **API Key** y la **API URL**
3. Copia el **endpoint** (ej: `https://ia.meuservidor.com/v1/chat-messages`)

### En n8n

Crea un workflow con:
1. **Webhook node**: recibe mensaje de evolution-go
2. **HTTP Request node**: llama a Dify API
3. **HTTP Request node**: envía respuesta a evolution-go

Ejemplo de llamada a la API de Dify:

```bash
curl -X POST https://ia.meuservidor.com/v1/chat-messages \
  -H "Authorization: Bearer TU-API-KEY-DE-DIFY" \
  -H "Content-Type: application/json" \
  -d '{
    "inputs": {},
    "query": "¿Cuál es el horario de atención?",
    "response_mode": "blocking",
    "user": "5511999999999"
  }'
```

### Con Typebot + Dify

Typebot puede consumir Dify mediante un bloque **HTTP Request**:

1. Crea un Typebot con flujo de FAQ
2. Agrega un bloque **HTTP Request** configurado para llamar a Dify
3. Usa variables de Typebot para pasar la pregunta del cliente
4. Muestra la respuesta de Dify en el chat

## Configurar almacenamiento S3/MinIO

Para guardar archivos e imágenes enviados por los usuarios:

```env
STORAGE_TYPE=s3
S3_ENDPOINT=http://minio:9000
S3_REGION=us-east-1
S3_BUCKET_NAME=dify
S3_ACCESS_KEY=minioadmin
S3_SECRET_KEY=minioadmin
```

Consulta la [guía de MinIO](./07-minio.md) para configurar el bucket.

## Mantenimiento

### Actualizar

```bash
cd ~/dify
git pull
cd docker
docker compose down
docker compose up -d
```

### Logs

```bash
cd ~/dify/docker
docker compose logs -f api
docker compose logs -f web
```

### Backup

```bash
#!/bin/bash
# ~/dify/backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR=~/backups/dify
mkdir -p $BACKUP_DIR

# Backup de la base
docker compose exec -T db pg_dump -U postgres dify > $BACKUP_DIR/dify-db-$DATE.sql
gzip $BACKUP_DIR/dify-db-$DATE.sql

# Backup de archivos (uploads, base de conocimiento)
docker run --rm -v dify_storage:/source -v $BACKUP_DIR:/backup alpine tar czf /backup/dify-storage-$DATE.tar.gz -C /source .

echo "Backup completado: $DATE"
```

## Troubleshooting

| Error | Causa | Solución |
|-------|-------|----------|
| Weaviate connection refused | Weaviate no inició | `docker compose logs weaviate` |
| LLM response timeout | API key inválida o límite excedido | Verifica saldo del proveedor |
| Base de conocimiento vacía | Documentos no indexaron | Verifica logs del worker |
| 502 Bad Gateway | api no respondió | `docker compose logs api` |
| Poca memoria | Weaviate + API + Worker consumen mucho | Aumenta RAM de la VM a 6GB+ |

## Siguiente paso

[MinIO](./07-minio.md) - Almacenamiento S3 para archivos multimedia.
