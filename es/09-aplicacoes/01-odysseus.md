# 09-01 - Odysseus

## ¿Qué es?

[Odysseus](https://github.com/pewdiepie-archdaemon/odysseus) es un workspace de IA autoalojado creado por PewDiePie. Reúne en una sola interfaz:

- Chat con IA (modelos locales o vía API)
- Agentes autónomos con herramientas (bash, archivos, web, MCP)
- Investigación profunda (deep research) con generación de informes
- Editor de documentos con edición por IA
- Cliente de email (IMAP/SMTP) con clasificación y resumen por IA
- Notas, tareas y calendario (CalDAV)
- Soporte para modelos locales vía Ollama

## Arquitectura

```
odysseus (contenedor)
    Puerto: 7000
    |
    +-- PostgreSQL (datos persistentes)
    +-- Redis (caché/sesiones)
```

## Prerrequisitos

- Docker y Docker Compose instalados en la VM
- Al menos 2 GB de RAM libres
- Git instalado: `sudo apt install -y git`

## Instalación

### 1. Acceder a la VM

```bash
ssh ubuntu@192.168.1.100
```

### 2. Clonar el repositorio

```bash
cd ~
git clone https://github.com/pewdiepie-archdaemon/odysseus.git
cd odysseus
```

### 3. Configurar variables de entorno

```bash
cp .env.example .env
nano .env
```

Variables esenciales:

```env
# Puerto del servidor
ODYSSEUS_PORT=7000

# Clave secreta para sesiones (genera una fuerte)
SECRET_KEY=genera-una-clave-aleatoria-aqui

# Base de datos
DATABASE_URL=postgresql://odysseus:senha@postgres:5432/odysseus
REDIS_URL=redis://redis:6379

# Primer usuario admin (creado automáticamente en el primer inicio)
FIRST_ADMIN_EMAIL=admin@meuservidor.com
FIRST_ADMIN_PASSWORD=senha-super-forte-aqui

# URL pública (para enlaces en correos)
PUBLIC_URL=https://odysseus.meuservidor.com

# Modelo por defecto (opcional - API key)
# OPENAI_API_KEY=su-chave
# ANTHROPIC_API_KEY=su-chave
# OLLAMA_BASE_URL=http://host.docker.internal:11434
```

### 4. Iniciar con Docker Compose

```bash
docker compose up -d --build
```

La primera vez puede demorar algunos minutos (build de la imagen).

### 5. Verificar que está funcionando

```bash
docker compose ps
docker compose logs -f
```

### 6. Obtener contraseña del admin

Para instalaciones que generan contraseña automática, revisa los logs:

```bash
docker compose logs | grep -i password
```

Si configuraste `FIRST_ADMIN_PASSWORD` en el `.env`, usa esa contraseña.

## Configurar dominio en Cloudflare Tunnel

Si estás usando Cloudflare Tunnel, edita el config.yml:

```bash
nano ~/.cloudflared/config.yml
```

Agrega el hostname antes del catch-all (404):

```yaml
  - hostname: odysseus.meuservidor.com
    service: http://localhost:7000
```

Reinicia el tunnel:

```bash
sudo systemctl restart cloudflared
```

## Agregar modelos de IA

### OpenAI
```env
OPENAI_API_KEY=sk-proj-...
```

### Modelos locales (Ollama)
Si tienes Ollama corriendo en el host o en otro servidor:

```env
OLLAMA_BASE_URL=http://192.168.1.200:11434
```

Para instalar Ollama en la propia VM:

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull llama3.2:3b
```

### Proveedores compatibles
- OpenAI / OpenAI-compatible
- Anthropic Claude
- Ollama (modelos locales)
- Google Gemini
- Cualquier API compatible con OpenAI

## Acceder

- Local: `http://192.168.1.100:7000`
- Público: `https://odysseus.meuservidor.com`

## Mantenimiento

### Actualizar

```bash
cd ~/odysseus
git pull
docker compose down
docker compose up -d --build
```

### Logs

```bash
docker compose logs -f
```

### Backup

```bash
# Backup de la base de datos
docker compose exec postgres pg_dump -U odysseus odysseus > ~/backup-odysseus-$(date +%Y%m%d).sql
```

## Siguiente paso

[n8n](./02-n8n.md) - Automatización de workflows.
