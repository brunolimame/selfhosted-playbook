# 09-07 - MinIO

## O que e?

[MinIO](https://min.io) e um servidor de armazenamento de objetos de alta performance, compativel com a API S3 da Amazon. E a alternativa self-hosted ao Amazon S3.

Funcionalidades principais:
- API 100% compativel com S3 (use SDKs e ferramentas S3 existentes)
- Console web para gerenciamento de buckets e arquivos
- Alto desempenho (escrito em Go, ~50 MB por container)
- Suporte a erasure coding para protecao de dados
- Multi-tenant com buckets e politicas de acesso
- Compativel com evolution-go, Typebot, Dify e Chatwoot para upload de midia

## Arquitetura

```
minio (container unico - single node)
    Porta 9000 (API S3)
    Porta 9001 (Console web)
    |
    +-- /data (volume persistente)
```

## Por que MinIO no ecossistema de automacao?

Varias aplicacoes precisam de armazenamento S3 para arquivos de midia:

| Aplicacao | Uso do MinIO |
|-----------|-------------|
| **Evolution Go** | Fotos, audios, videos e documentos enviados/recebidos no WhatsApp |
| **Typebot** | Upload de imagens e arquivos nos fluxos do chatbot |
| **Dify** | Documentos da base de conhecimento e arquivos enviados pelo usuario |
| **Chatwoot** | Anexos nas conversas de atendimento |
| **n8n** | Armazenamento intermediario de arquivos em workflows |

## Pre-requisitos

- Docker e Docker Compose instalados na VM
- Disco com espaco suficiente para os arquivos de midia
- Dominio para console: `minio.meuservidor.com` (opcional)

## Instalacao

### 1. Acessar a VM e criar diretorio

```bash
ssh ubuntu@192.168.1.100
mkdir -p ~/minio
cd ~/minio
```

### 2. Criar docker-compose.yml

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

### 5. Acessar o console

- **API S3**: `http://192.168.1.100:9000`
- **Console web**: `http://192.168.1.100:9001`
- **Usuario**: `minioadmin`
- **Senha**: `minio-senha-forte-aqui`

## Configurar dominio no Cloudflare Tunnel

Edite o config.yml:

```bash
nano ~/.cloudflared/config.yml
```

Adicione:

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

## Criar buckets e chaves de acesso

### Pelo console web

1. Acesse `http://192.168.1.100:9001`
2. Login com `minioadmin` / `minio-senha-forte-aqui`
3. **Buckets > Create Bucket**:
   - `evolution-media` (midia do WhatsApp)
   - `typebot` (uploads do Typebot)
   - `dify` (documentos da base de conhecimento)
   - `chatwoot` (anexos do Chatwoot)
4. **Access Keys > Create Access Key**:
   - Anote o `Access Key` e `Secret Key`

### Pela linha de comando (mc client)

```bash
# Instalar mc client
docker compose exec minio mc alias set local http://localhost:9000 minioadmin minio-senha-forte-aqui

# Criar buckets
docker compose exec minio mc mb local/evolution-media
docker compose exec minio mc mb local/typebot
docker compose exec minio mc mb local/dify
docker compose exec minio mc mb local/chatwoot

# Listar buckets
docker compose exec minio mc ls local
```

## Configurar politicas de acesso (opcional)

Para buckets que precisam ser publicos (ex: imagens de perfil do WhatsApp):

```bash
docker compose exec minio mc anonymous set download local/evolution-media
```

## Integrar com as aplicacoes

### Evolution Go

No `.env` do evolution-go:

```env
MINIO_ENABLED=true
MINIO_ENDPOINT=minio:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minio-senha-forte-aqui
MINIO_BUCKET=evolution-media
MINIO_SSL=false
```

### Typebot

No `.env` do Typebot:

```env
S3_ACCESS_KEY=minioadmin
S3_SECRET_KEY=minio-senha-forte-aqui
S3_BUCKET=typebot
S3_ENDPOINT=http://minio:9000
S3_SSL=false
```

### Dify

No `.env` do Dify (em `~/dify/docker/.env`):

```env
STORAGE_TYPE=s3
S3_ENDPOINT=http://minio:9000
S3_REGION=us-east-1
S3_BUCKET_NAME=dify
S3_ACCESS_KEY=minioadmin
S3_SECRET_KEY=minio-senha-forte-aqui
```

### Chatwoot

No `.env` do Chatwoot:

```env
ACTIVE_STORAGE_SERVICE=amazon
STORAGE_S3_ENDPOINT=http://minio:9000
STORAGE_S3_BUCKET=chatwoot
AWS_ACCESS_KEY_ID=minioadmin
AWS_SECRET_ACCESS_KEY=minio-senha-forte-aqui
AWS_REGION=us-east-1
```

**Importante**: Para que os containers se comuniquem entre si, use o nome do servico (`minio`) em vez de `localhost` ou `192.168.1.100` no endpoint, desde que estejam na mesma rede Docker.

Se as aplicacoes estao em redes Docker separadas, use o IP da VM (`192.168.1.100`) no endpoint.

## Manutencao

### Atualizar

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

# Backup dos dados
docker run --rm -v minio_data:/source -v $BACKUP_DIR:/backup alpine tar czf /backup/minio-data-$DATE.tar.gz -C /source .

# Backup da configuracao (buckets, politicas)
docker compose exec minio mc alias set local http://localhost:9000 minioadmin minio-senha-forte-aqui
docker compose exec minio mc admin config get local > $BACKUP_DIR/minio-config-$DATE.txt

echo "Backup concluido: $DATE"
```

### Restaurar

```bash
docker run --rm -v minio_data:/dest -v $BACKUP_DIR:/backup alpine tar xzf /backup/minio-data-20260101.tar.gz -C /dest
docker compose restart
```

### Limpeza de arquivos antigos (opcional)

```bash
# Script para apagar midias com mais de 90 dias
docker compose exec minio mc find local/evolution-media --older-than 90d --exec "mc rm {}"
```

## Troubleshooting

| Erro | Causa | Solucao |
|------|-------|---------|
| `Connection refused` | Container nao iniciou | `docker compose logs minio` |
| `Access Denied` | Credenciais erradas | Verificar `MINIO_ROOT_USER`/`MINIO_ROOT_PASSWORD` |
| Bucket ja existe | Nome duplicado | Usar outro nome |
| S3 incompativel | Endpoint errado | Usar `http://minio:9000` (Docker) ou `http://192.168.1.100:9000` (host) |
| Disco cheio | Muitas midias armazenadas | Configurar politica de retencao/limpeza |

## Proximo passo

[Seguranca](../07-seguranca.md) - Proteja seu servidor.
