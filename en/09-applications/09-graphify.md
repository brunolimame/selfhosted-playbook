# 09-09 - Graphify

## What is it?

[Graphify](https://graphifylabs.ai) is an open-source tool that transforms code repositories, documentation, PDFs and images into a **knowledge graph** queryable by AI assistants (Claude Code, Codex, OpenCode, Cursor, Gemini CLI, etc.).

Instead of grepping through files, you ask Graphify and it searches the project's knowledge graph.

Main features:
- Maps code, docs, PDFs, images and videos into a knowledge graph
- Works with Claude Code, Codex, OpenCode, Cursor, GitHub Copilot and more
- Supports 20 programming languages (Python, JS, Go, Rust, Java...)
- Local processing (no code sent to the cloud)
- MIT License, free

## Architecture

```
graphify (MCP HTTP server)
    Port: 8080
    |
    +-- /data/graph.json (graph persisted to disk)
    |
    +-- Uses configured AI assistant's LLM (does not send raw code)
```

## Difference from other tools

| Tool | Purpose |
|-----------|-----------|
| Graphify | Knowledge graph for AI assistants |
| Dify | AI app platform + RAG |
| Odysseus | AI workspace (chat, agents, email) |
| n8n | Workflow automation |

Graphify is specifically for **developers** who want their AI assistant to understand the full codebase.

## Prerequisites

- Docker installed on the VM
- Python 3.10+ (if using via pip)
- Project to analyze (local Git repository)

## Installation

### 1. Access the VM

```bash
ssh ubuntu@192.168.1.100
```

### 2. Option A: Install via Docker

```bash
mkdir -p ~/graphify
cd ~/graphify
```

Create the `docker-compose.yml`:

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

### 3. Option B: Install via pip

```bash
# Install the package
pip install graphifyy

# Install the skill in the AI assistant
graphify install
```

### 4. Build the graph

```bash
# Docker
docker compose exec graphify python -m graphify.serve /project --output /data/graph.json

# Or via CLI
graphify /caminho/para/seu/projeto
```

## How to use with AI assistants

### OpenCode / Claude Code / Codex

In the assistant's chat, type:

```
/graphify ./caminho/do/projeto
```

Graphify will:
1. Analyze all project files
2. Extract structure (classes, functions, dependencies)
3. Build the knowledge graph
4. Make it available for querying

### Query example

```
/graphify ./src

# Then ask:
"What function calculates the cart total?"
"How does the authentication flow work?"
"Which API endpoints use the rate limit middleware?"
"Show the dependency diagram between modules"
```

## Usage with other projects on the VM

Graphify can analyze the repositories of installed applications:

```bash
# Analyze Typebot
graphify ~/typebot

# Analyze Dify
graphify ~/dify

# Analyze n8n workflows (exported as JSON)
graphify ~/n8n
```

## Configure domain on Cloudflare Tunnel (optional)

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

## Next step

[pgAdmin](./10-pgadmin.md) - PostgreSQL administration via web.
