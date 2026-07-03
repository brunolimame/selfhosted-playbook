# Template para Adicionar Nova Aplicacao

Este arquivo e um template/instruct para LLMs (como eu) que precisam documentar a instalacao de uma nova aplicacao neste projeto.

## Instrucoes para o LLM

Ao receber a solicitacao de adicionar uma nova aplicacao, siga estas etapas:

1.  **Pesquise** a aplicacao para entender:
    - O que ela faz (descricao curta)
    - Arquitetura (portas, dependencias, containers)
    - Forma de instalacao (Docker Compose, script, manual)
    - Requisitos de hardware e software
    - Dependencias externas (banco de dados, filas, armazenamento)

2.  **Crie o arquivo** em `docs/09-aplicacoes/XX-nome-da-app.md` seguindo o template abaixo.

3.  **Atualize** os indices de navegacao:
    - `docs/09-aplicacoes/README.md` - adicione a tabela de conteudo
    - `docs/README.md` - adicione na secao 09 se aplicavel

## Template da documentacao

Copie o conteudo abaixo e preencha os campos marcados com `[...]`:

```markdown
# 09-XX - Nome da Aplicacao

## O que e?

[Nome da aplicacao](URL-do-repositorio) e uma breve descricao do que ela faz.

Funcionalidades principais:
- Funcionalidade 1
- Funcionalidade 2
- Funcionalidade 3

## Arquitetura

```
[nome-app] (container)
    Porta: [PORTA]
    |
    +-- [Dependencia 1] (ex: PostgreSQL, Redis, MySQL)
    +-- [Dependencia 2] (se houver)
```

## Pre-requisitos

- Docker e Docker Compose instalados na VM
- [Outros requisitos especificos: RAM, CPU, bibliotecas]
- [Contas necessarias: API keys, servicos externos]

## Instalacao

### 1. Acessar a VM

\`\`\`bash
ssh ubuntu@192.168.1.100
\`\`\`

### 2. Criar diretorio do projeto

\`\`\`bash
mkdir -p ~/[nome-app]
cd ~/[nome-app]
\`\`\`

### 3. Configurar variaveis de ambiente

\`\`\`bash
nano .env
\`\`\`

\`\`\`env
# Porta do servidor
PORT=[PORTA]

# Configuracoes principais
# [adicione aqui as variaveis essenciais]
API_KEY=sua-chave-aqui

# Banco de dados
DATABASE_URL=postgresql://usuario:senha@postgres:5432/[dbname]
\`\`\`

### 4. Criar docker-compose.yml

\`\`\`bash
nano docker-compose.yml
\`\`\`

\`\`\`yaml
services:
  postgres:
    image: postgres:[versao]-alpine
    container_name: [app]-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: usuario
      POSTGRES_PASSWORD: senha
      POSTGRES_DB: [dbname]
    volumes:
      - [app]_postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U usuario"]
      interval: 10s
      timeout: 5s
      retries: 5

  [app]:
    image: [imagem-oficial]:[tag]
    container_name: [app]
    restart: unless-stopped
    ports:
      - "[PORTA]:[PORTA]"
    environment:
      - PORT=\${PORT}
      - DATABASE_URL=\${DATABASE_URL}
      # [mais variaveis do .env]
    volumes:
      - [app]_data:/app/data
    depends_on:
      postgres:
        condition: service_healthy

volumes:
  [app]_postgres_data:
  [app]_data:
\`\`\`

### 5. Iniciar

\`\`\`bash
cd ~/[nome-app]
docker compose up -d
\`\`\`

### 6. Verificar

\`\`\`bash
docker compose ps
docker compose logs -f
\`\`\`

### 7. [Passo adicional se necessario: criar banco, rodar migrations, etc.]

\`\`\`bash
# Exemplo: criar banco de dados
docker compose exec postgres psql -U usuario -c "CREATE DATABASE [dbname];"
\`\`\`

## Configurar dominio no Cloudflare Tunnel

Edite o config.yml:

\`\`\`bash
nano ~/.cloudflared/config.yml
\`\`\`

Adicione antes do catch-all (404):

\`\`\`yaml
  - hostname: [subdominio].meuservidor.com
    service: http://localhost:[PORTA]
\`\`\`

Reinicie:

\`\`\`bash
sudo systemctl restart cloudflared
\`\`\`

## Acessar

- Local: \`http://192.168.1.100:[PORTA]\`
- Publico: \`https://[subdominio].meuservidor.com\`

## Integracoes uteis

- [Integracao 1: descreva como esta app se conecta com n8n, Evolution Go, etc.]
- [Integracao 2]

## Manutencao

### Atualizar

\`\`\`bash
cd ~/[nome-app]
docker compose pull
docker compose up -d
\`\`\`

### Logs

\`\`\`bash
docker compose logs -f
\`\`\`

### Backup

\`\`\`bash
#!/bin/bash
# ~/[nome-app]/backup.sh

DATE=\$(date +%Y%m%d_%H%M%S)
BACKUP_DIR=~/backups/[nome-app]
mkdir -p \$BACKUP_DIR

# [Comandos de backup especificos]
docker compose exec -T postgres pg_dump -U usuario [dbname] > \$BACKUP_DIR/[app]-db-\$DATE.sql
gzip \$BACKUP_DIR/*.sql

echo "Backup concluido: \$DATE"
\`\`\`

## Troubleshooting

| Erro | Causa | Solucao |
|------|-------|---------|
| [Erro comum 1] | [Causa] | [Solucao] |
| [Erro comum 2] | [Causa] | [Solucao] |
| [Erro comum 3] | [Causa] | [Solucao] |
```

## Checklist para o LLM antes de finalizar

- [ ] Pesquisei a aplicacao oficialmente (site, GitHub, docs)
- [ ] Verifiquei a porta padrao da aplicacao
- [ ] Inclui as variaveis de ambiente minimas para funcionar
- [ ] Inclui configuracao de dominio (Cloudflare Tunnel)
- [ ] Inclui secao de manutencao (backup, logs, atualizacao)
- [ ] Inclui troubleshooting com erros comuns
- [ ] Atualizei o `docs/09-aplicacoes/README.md` com a entrada na tabela
- [ ] Atualizei o `docs/README.md` se necessario
- [ ] Usei `192.168.1.100` como IP de exemplo da VM
- [ ] Usei `meuservidor.com` como dominio de exemplo
- [ ] Numeracao do arquivo segue a sequencia (01, 02, 03...)
