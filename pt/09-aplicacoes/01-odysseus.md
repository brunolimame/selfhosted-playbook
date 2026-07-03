# 09-01 - Odysseus

## O que e?

[Odysseus](https://github.com/pewdiepie-archdaemon/odysseus) e um workspace de IA auto-hospedado criado por PewDiePie. Ele reune em uma unica interface:

- Chat com IA (modelos locais ou via API)
- Agentes autonomos com ferramentas (bash, arquivos, web, MCP)
- Pesquisa profunda (deep research) com geracao de relatorios
- Editor de documentos com edicao por IA
- Cliente de email (IMAP/SMTP) com triagem e resumo por IA
- Notas, tarefas e calendario (CalDAV)
- Suporte a modelos locais via Ollama

## Arquitetura

```
odysseus (container)
    Porta: 7000
    |
    +-- PostgreSQL (dados persistentes)
    +-- Redis (cache/sessoes)
```

## Pre-requisitos

- Docker e Docker Compose instalados na VM
- No minimo 2 GB de RAM livres
- Git instalado: `sudo apt install -y git`

## Instalacao

### 1. Acessar a VM

```bash
ssh ubuntu@192.168.1.100
```

### 2. Clonar o repositorio

```bash
cd ~
git clone https://github.com/pewdiepie-archdaemon/odysseus.git
cd odysseus
```

### 3. Configurar variaveis de ambiente

```bash
cp .env.example .env
nano .env
```

Variaveis essenciais:

```env
# Porta do servidor
ODYSSEUS_PORT=7000

# Chave secreta para sessoes (gere uma forte)
SECRET_KEY=gere-uma-chave-aleatoria-aqui

# Banco de dados
DATABASE_URL=postgresql://odysseus:senha@postgres:5432/odysseus
REDIS_URL=redis://redis:6379

# Primeiro usuario admin (criado automaticamente no primeiro inicio)
FIRST_ADMIN_EMAIL=admin@meuservidor.com
FIRST_ADMIN_PASSWORD=senha-super-forte-aqui

# URL publica (para links em emails)
PUBLIC_URL=https://odysseus.meuservidor.com

# Modelo padrao (opcional - API key)
# OPENAI_API_KEY=sua-chave
# ANTHROPIC_API_KEY=sua-chave
# OLLAMA_BASE_URL=http://host.docker.internal:11434
```

### 4. Iniciar com Docker Compose

```bash
docker compose up -d --build
```

A primeira vez pode demorar alguns minutos (build da imagem).

### 5. Verificar se esta rodando

```bash
docker compose ps
docker compose logs -f
```

### 6. Obter senha do admin

Para instalacoes que geram senha automatica, veja os logs:

```bash
docker compose logs | grep -i password
```

Se configurou `FIRST_ADMIN_PASSWORD` no `.env`, use essa senha.

## Configurar dominio no Cloudflare Tunnel

Se estiver usando Cloudflare Tunnel, edite o config.yml:

```bash
nano ~/.cloudflared/config.yml
```

Adicione o hostname antes do catch-all (404):

```yaml
  - hostname: odysseus.meuservidor.com
    service: http://localhost:7000
```

Reinicie o tunnel:

```bash
sudo systemctl restart cloudflared
```

## Adicionar modelos de IA

### OpenAI
```env
OPENAI_API_KEY=sk-proj-...
```

### Modelos locais (Ollama)
Se tiver Ollama rodando no host ou em outro servidor:

```env
OLLAMA_BASE_URL=http://192.168.1.200:11434
```

Para instalar Ollama na propria VM:

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull llama3.2:3b
```

### Provedores compatíveis
- OpenAI / OpenAI-compatible
- Anthropic Claude
- Ollama (modelos locais)
- Google Gemini
- Qualquer API compatível com OpenAI

## Acessar

- Local: `http://192.168.1.100:7000`
- Publico: `https://odysseus.meuservidor.com`

## Manutencao

### Atualizar

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
# Backup do banco de dados
docker compose exec postgres pg_dump -U odysseus odysseus > ~/backup-odysseus-$(date +%Y%m%d).sql
```

## Proximo passo

[n8n](./02-n8n.md) - Automacao de workflows.
