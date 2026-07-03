# 09-09 - Graphify

## ¿Qué es?

[Graphify](https://graphifylabs.ai) es una herramienta open-source que transforma repositorios de código, documentación, PDFs e imágenes en un **grafo de conocimiento** consultable por asistentes de IA (Claude Code, Codex, OpenCode, Cursor, Gemini CLI, etc.).

En lugar de hacer grep en archivos, usted pregunta a Graphify y él busca en el grafo de conocimiento del proyecto.

Funcionalidades principales:
- Mapea código, docs, PDFs, imágenes y videos en un grafo de conocimiento
- Funciona con Claude Code, Codex, OpenCode, Cursor, GitHub Copilot y más
- Soporte a 20 lenguajes de programación (Python, JS, Go, Rust, Java...)
- Procesamiento local (sin envío de código a la nube)
- MIT License, gratuito

## Arquitectura

```
graphify (servidor MCP HTTP)
    Puerto: 8080
    |
    +-- /data/graph.json (grafo persistido en disco)
    |
    +-- Usa LLM del asistente de IA configurado (no envía código bruto)
```

## Diferencia respecto a otras herramientas

| Herramienta | Propósito |
|-----------|-----------|
| Graphify | Grafo de conocimiento para asistentes de IA |
| Dify | Plataforma de apps con IA + RAG |
| Odysseus | Workspace de IA (chat, agentes, email) |
| n8n | Automatización de workflows |

Graphify es específico para **desarrolladores** que quieren que su asistente de IA entienda la base de código completa.

## Prerrequisitos

- Docker instalado en la VM
- Python 3.10+ (si se usa via pip)
- Proyecto para analizar (repositorio Git local)

## Instalación

### 1. Acceder a la VM

```bash
ssh ubuntu@192.168.1.100
```

### 2. Opción A: Instalar via Docker

```bash
mkdir -p ~/graphify
cd ~/graphify
```

Cree el `docker-compose.yml`:

```yaml
services:
  graphify:
    build: https://github.com/safishamsi/graphify.git#v8
    container_name: graphify
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - graphify_data:/data
      - /caminho/para/seu/projeto:/project:ro
    command: ["/data/graph.json", "--transport", "http", "--host", "0.0.0.0", "--port", "8080", "--api-key", "sua-chave-aqui"]

volumes:
  graphify_data:
```

### 3. Opción B: Instalar via pip

```bash
# Instalar el paquete
pip install graphifyy

# Instalar el skill en el asistente de IA
graphify install
```

### 4. Construir el grafo

```bash
# Docker
docker compose exec graphify python -m graphify.serve /project --output /data/graph.json

# O via CLI
graphify /caminho/para/seu/projeto
```

## Cómo usar con asistentes de IA

### OpenCode / Claude Code / Codex

En el chat del asistente, escriba:

```
/graphify ./caminho/do/projeto
```

Graphify va a:
1. Analizar todos los archivos del proyecto
2. Extraer estructura (clases, funciones, dependencias)
3. Construir el grafo de conocimiento
4. Disponibilizar para consulta

### Ejemplo de consulta

```
/graphify ./src

# Después pregunte:
"¿Cuál es la función que calcula el total del carrito?"
"¿Cómo funciona el flujo de autenticación?"
"¿Qué endpoints de la API usan el middleware de rate limit?"
"Muestra el diagrama de dependencias entre los módulos"
```

## Uso con otros proyectos de la VM

Graphify puede analizar los repositorios de las aplicaciones instaladas:

```bash
# Analizar Typebot
graphify ~/typebot

# Analizar Dify
graphify ~/dify

# Analizar workflows de n8n (exportados como JSON)
graphify ~/n8n
```

## Configurar dominio en Cloudflare Tunnel (opcional)

```bash
nano ~/.cloudflared/config.yml
```

```yaml
  - hostname: graphify.meuservidor.com
    service: http://localhost:8080
```

```bash
sudo systemctl restart cloudflared
```

## Próximo paso

[pgAdmin](./10-pgadmin.md) - Administración de PostgreSQL via web.
