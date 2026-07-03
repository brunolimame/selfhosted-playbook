# 09-09 - Graphify

## O que e?

[Graphify](https://graphifylabs.ai) e uma ferramenta open-source que transforma repositorios de codigo, documentacao, PDFs e imagens em um **grafo de conhecimento** consultavel por assistentes de IA (Claude Code, Codex, OpenCode, Cursor, Gemini CLI, etc.).

Em vez de fazer grep em arquivos, voce pergunta ao Graphify e ele busca no grafo de conhecimento do projeto.

Funcionalidades principais:
- Mapeia codigo, docs, PDFs, imagens e videos em um grafo de conhecimento
- Funciona com Claude Code, Codex, OpenCode, Cursor, GitHub Copilot e mais
- Suporte a 20 linguagens de programacao (Python, JS, Go, Rust, Java...)
- Processamento local (sem envio de codigo para nuvem)
- MIT License, gratuito

## Arquitetura

```
graphify (servidor MCP HTTP)
    Porta: 8080
    |
    +-- /data/graph.json (grafo persistido em disco)
    |
    +-- Usa LLM do assistente de IA configurado (nao envia codigo bruto)
```

## Diferenca para outras ferramentas

| Ferramenta | Proposito |
|-----------|-----------|
| Graphify | Grafo de conhecimento para assistentes de IA |
| Dify | Plataforma de apps com IA + RAG |
| Odysseus | Workspace de IA (chat, agentes, email) |
| n8n | Automacao de workflows |

Graphify e especifico para **desenvolvedores** que querem que seu assistente de IA entenda a base de codigo completa.

## Pre-requisitos

- Docker instalado na VM
- Python 3.10+ (se for usar via pip)
- Projeto para analisar (repositorio Git local)

## Instalacao

### 1. Acessar a VM

```bash
ssh ubuntu@192.168.1.100
```

### 2. Opcao A: Instalar via Docker

```bash
mkdir -p ~/graphify
cd ~/graphify
```

Crie o `docker-compose.yml`:

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

### 3. Opcao B: Instalar via pip

```bash
# Instalar o pacote
pip install graphifyy

# Instalar o skill no assistente de IA
graphify install
```

### 4. Construir o grafo

```bash
# Docker
docker compose exec graphify python -m graphify.serve /project --output /data/graph.json

# Ou via CLI
graphify /caminho/para/seu/projeto
```

## Como usar com assistentes de IA

### OpenCode / Claude Code / Codex

No chat do assistente, digite:

```
/graphify ./caminho/do/projeto
```

O Graphify vai:
1. Analisar todos os arquivos do projeto
2. Extrair estrutura (classes, funcoes, dependencias)
3. Construir o grafo de conhecimento
4. Disponibilizar para consulta

### Exemplo de consulta

```
/graphify ./src

# Depois pergunte:
"Qual a funcao que calcula o total do carrinho?"
"Como funciona o fluxo de autenticacao?"
"Quais endpoints da API usam o middleware de rate limit?"
"Mostre o diagrama de dependencias entre os modulos"
```

## Uso com outros projetos da VM

Graphify pode analisar os repositorios das aplicacoes instaladas:

```bash
# Analisar o Typebot
graphify ~/typebot

# Analisar o Dify
graphify ~/dify

# Analisar workflows do n8n (exportados como JSON)
graphify ~/n8n
```

## Configurar dominio no Cloudflare Tunnel (opcional)

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

## Proximo passo

[pgAdmin](./10-pgadmin.md) - Administracao de PostgreSQL via web.
