# 09-06 - Dify

## O que e?

[Dify](https://dify.ai) e uma plataforma open-source para criacao de aplicacoes com IA generativa. Permite construir chatbots inteligentes com base de conhecimento propria (RAG), workflows de IA e agentes autonomos.

Funcionalidades principais:
- **RAG (Retrieval-Augmented Generation)**: Conecte documentos (PDF, TXT, HTML, etc.) e faca perguntas sobre eles
- **Chatbot com contexto**: Upload de documentos como base de conhecimento
- **Workflow de IA**: Crie pipelines com nos visuais (LLM, ferramentas, logica)
- **Agentes**: Agentes autonomos com ferramentas (web search, API, calculo)
- **API completa**: Integre com qualquer sistema (evolution-go, n8n, Typebot)
- **Multi-modelo**: OpenAI, Anthropic, Ollama (local), modelos open-source

## Arquitetura

```
dify (multi-containers)
    |
    +-- api (backend Flask/Python)      -> Porta 5001
    +-- worker (processamento em fila)
    +-- web (frontend Next.js)          -> Porta 3000
    +-- nginx (proxy reverso interno)   -> Porta 80
    |
    +-- PostgreSQL (dados da aplicacao)
    +-- Redis (cache/filas)
    +-- Weaviate (vetor database para RAG)
```

## Por que Dify com Evolution Go?

O Dify da inteligencia para o atendimento:

```
Cliente pergunta: "Qual o horario de funcionamento?"
     |
evolution-go recebe a mensagem
     |
n8n encaminha para Dify
     |
Dify consulta base de conhecimento (PDF com FAQ)
     |
LLM responde com contexto dos documentos
     |
n8n retorna resposta para evolution-go
     |
evolution-go envia mensagem no WhatsApp
```

## Pre-requisitos

- Docker e Docker Compose 2.24+ instalados na VM
- No minimo 4 GB RAM livres (Dify + Weaviate consomem memoria)
- CPU 2+ cores
- Dominio configurado: `ia.meuservidor.com`
- (Opcional) API key de um provedor LLM (OpenAI, Anthropic)

## Instalacao

### 1. Acessar a VM e clonar o repositorio

```bash
ssh ubuntu@192.168.1.100
cd ~
git clone https://github.com/langgenius/dify.git
cd dify
```

### 2. Configurar variaveis de ambiente

```bash
cd docker
cp .env.example .env
nano .env
```

Variaveis essenciais:

```env
# Modo de deploy
DEPLOY_ENV=PRODUCTION

# Portas (evitar conflito com outras apps)
EXPOSE_NGINX_PORT=80
EXPOSE_NGINX_SSL_PORT=443

# Chave secreta (gerar com: openssl rand -hex 32)
SECRET_KEY=sua-chave-secreta-aqui

# Banco de dados
DB_USERNAME=postgres
DB_PASSWORD=dify-senha
DB_DATABASE=dify
DB_PORT=5432

# Redis
REDIS_PASSWORD=dify-redis-senha

# Weaviate (vetor store)
WEAVIATE_AUTHENTICATION_ANONYMOUS_ACCESS_ENABLED=false
WEAVIATE_AUTHENTICATION_APIKEY_ENABLED=true
WEAVIATE_AUTHENTICATION_APIKEY_ALLOWED_KEYS=dify-weaviate-key
WEAVIATE_AUTHENTICATION_APIKEY_USERS=admin

# Inicializacao do admin
INIT_PASSWORD=senha-admin-super-forte
INIT_EMAIL=admin@meuservidor.com
```

### 3. Ajustar docker-compose.yml (portas)

Se o Coolify ja estiver usando a porta 80, edite o `docker-compose.yml`:

```bash
nano docker-compose.yml
```

Altere as portas do nginx:

```yaml
  nginx:
    ports:
      - "8080:80"
```

Isso fara o Dify rodar em `http://localhost:8080`.

### 4. Iniciar

```bash
cd ~/dify/docker
docker compose up -d
```

A primeira vez pode demorar (download de varias imagens).

### 5. Verificar

```bash
docker compose ps
```

Devem estar rodando: `api`, `worker`, `web`, `nginx`, `db`, `redis`, `weaviate`, `ssrf_proxy`, `sandbox`, `plugin_daemon`.

### 6. Acessar e configurar admin

Acesse `http://192.168.1.100:8080/install` e siga o fluxo de configuracao:
1. Defina email e senha do admin (ou use `INIT_EMAIL`/`INIT_PASSWORD` do `.env`)
2. Selecione o provedor LLM padrao (OpenAI, Anthropic, ou configure depois)

## Configurar dominio no Cloudflare Tunnel

Edite o config.yml:

```bash
nano ~/.cloudflared/config.yml
```

Adicione:

```yaml
  - hostname: ia.meuservidor.com
    service: http://localhost:8080
```

Reinicie:

```bash
sudo systemctl restart cloudflared
```

## Configurar provedor LLM

Va em **Settings > Model Provider** e adicione pelo menos um provedor:

### OpenAI
```env
OPENAI_API_KEY=sk-proj-seu-token
```

### Ollama (modelos locais)
Se tiver Ollama instalado na VM:
```env
OLLAMA_BASE_URL=http://192.168.1.100:11434
```

Depois va em **Settings > Model Provider > Ollama** e configure.

## Criar uma base de conhecimento

Para usar o RAG com seus documentos:

1. Va em **Knowledge > Create Knowledge**
2. Selecione **Upload Files**
3. Envie PDFs, TXTs, ou outros documentos (ex: FAQ, manual, catalogo)
4. Escolha o metodo de indexacao (recomendado: **High Quality** com Weaviate)
5. Aguarde o processamento
6. Va em **Studio > Create App > Chatbot**
7. Em **Context**, selecione a base de conhecimento criada
8. Publique o app

## Integrar com n8n + Evolution Go

Para conectar Dify, n8n e evolution-go:

### No Dify

1. Crie um app do tipo **Chatbot** com base de conhecimento
2. Va em **API Access** e copie a **API Key** e o **API URL**
3. Copie o **endpoint** (ex: `https://ia.meuservidor.com/v1/chat-messages`)

### No n8n

Crie um workflow com:
1. **Webhook node**: recebe mensagem do evolution-go
2. **HTTP Request node**: chama Dify API
3. **HTTP Request node**: envia resposta para evolution-go

Exemplo de chamada a API do Dify:

```bash
curl -X POST https://ia.meuservidor.com/v1/chat-messages \
  -H "Authorization: Bearer SEU-API-KEY-DO-DIFY" \
  -H "Content-Type: application/json" \
  -d '{
    "inputs": {},
    "query": "Qual o horario de funcionamento?",
    "response_mode": "blocking",
    "user": "5511999999999"
  }'
```

### Com Typebot + Dify

O Typebot pode consumir o Dify via bloco **HTTP Request**:

1. Crie um Typebot com fluxo de FAQ
2. Adicione bloco **HTTP Request** configurado para chamar o Dify
3. Use variaveis do Typebot para passar a pergunta do cliente
4. Exiba a resposta do Dify no chat

## Configurar armazenamento S3/MinIO

Para salvar arquivos e imagens enviados pelos usuarios:

```env
STORAGE_TYPE=s3
S3_ENDPOINT=http://minio:9000
S3_REGION=us-east-1
S3_BUCKET_NAME=dify
S3_ACCESS_KEY=minioadmin
S3_SECRET_KEY=minioadmin
```

Veja o [guia do MinIO](./07-minio.md) para configurar o bucket.

## Manutencao

### Atualizar

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

# Backup do banco
docker compose exec -T db pg_dump -U postgres dify > $BACKUP_DIR/dify-db-$DATE.sql
gzip $BACKUP_DIR/dify-db-$DATE.sql

# Backup dos arquivos (uploads, base de conhecimento)
docker run --rm -v dify_storage:/source -v $BACKUP_DIR:/backup alpine tar czf /backup/dify-storage-$DATE.tar.gz -C /source .

echo "Backup concluido: $DATE"
```

## Troubleshooting

| Erro | Causa | Solucao |
|------|-------|---------|
| Weaviate connection refused | Weaviate nao iniciou | `docker compose logs weaviate` |
| LLM response timeout | API key invalida ou limite excedido | Verificar saldo do provedor |
| Base de conhecimento vazia | Documentos nao indexaram | Verificar logs do worker |
| 502 Bad Gateway | api nao respondeu | `docker compose logs api` |
| Pouca memoria | Weaviate + API + Worker consomem muito | Aumentar RAM da VM para 6GB+ |

## Proximo passo

[MinIO](./07-minio.md) - Armazenamento S3 para arquivos de midia.
