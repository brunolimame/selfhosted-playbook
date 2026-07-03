# 09-10 - pgAdmin

## O que e?

[pgAdmin](https://www.pgadmin.org) e a ferramenta de administracao open-source mais popular para PostgreSQL. Oferece interface web completa para gerenciar bancos de dados, executar queries SQL, monitorar atividades e configurar replicacao.

Funcionalidades principais:
- Interface web para gerenciar multiplos servidores PostgreSQL
- Editor SQL com syntax highlighting e autocomplete
- Visualizador de esquemas (tabelas, indices, views)
- Grafos de dependencias entre objetos
- Monitoramento de conexoes ativas e locks
- Backup e restore com interface grafica
- Importacao/exportacao de dados

## Por que pgAdmin neste projeto?

Quase todas as aplicacoes da VM usam PostgreSQL:

| App | Banco | Uso |
|-----|-------|-----|
| Coolify | `coolify` | Dados da plataforma |
| n8n | `n8n` | Workflows, credenciais |
| evolution-go | `evogo_auth`, `evogo_users` | Instancias, usuarios |
| Typebot | `typebot` | Fluxos, configuracoes |
| Chatwoot | `chatwoot` | Conversas, contatos, agentes |
| Dify | `dify` | Aplicacoes, documentos |
| Odysseus | `odysseus` | Dados do workspace |

Com pgAdmin, voce gerencia todos de um lugar.

## Arquitetura

```
pgadmin (container) -> Porta 5050 (interface web)
    |
    Conecta-se a qualquer PostgreSQL da rede
    (mesma VM ou externo)
```

## Pre-requisitos

- Docker instalado na VM
- Pelo menos um servidor PostgreSQL rodando (ja deve ter de outras aplicacoes)

## Instalacao

### 1. Acessar a VM

```bash
ssh ubuntu@192.168.1.100
```

### 2. Criar diretorio

```bash
mkdir -p ~/pgadmin
cd ~/pgadmin
```

### 3. Criar docker-compose.yml

```bash
nano docker-compose.yml
```

```yaml
services:
  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: pgadmin
    restart: unless-stopped
    ports:
      - "5050:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@meuservidor.com
      PGADMIN_DEFAULT_PASSWORD: pgadmin-senha-forte
      PGADMIN_DISABLE_POSTFIX: "true"
      PGADMIN_CONFIG_SERVER_MODE: "False"
      PGADRONFIG_MASTER_PASSWORD_REQUIRED: "False"
    volumes:
      - pgadmin_data:/var/lib/pgadmin
      - ./servers.json:/pgadmin4/servers.json:ro

volumes:
  pgadmin_data:
```

### 4. (Opcional) Auto-configurar servidores

Para que o pgAdmin ja venha com os servidores configurados, crie o arquivo `servers.json`:

```bash
nano servers.json
```

```json
{
  "Servers": {
    "1": {
      "Name": "Coolify DB",
      "Group": "Servidores VM",
      "Host": "192.168.1.100",
      "Port": 5432,
      "MaintenanceDB": "postgres",
      "Username": "postgres",
      "PassFile": "/pgpass",
      "SSLMode": "prefer"
    },
    "2": {
      "Name": "n8n DB",
      "Group": "Servidores VM",
      "Host": "192.168.1.100",
      "Port": 5432,
      "MaintenanceDB": "n8n",
      "Username": "n8n",
      "PassFile": "/pgpass",
      "SSLMode": "prefer"
    }
  }
}
```

E o arquivo de senhas:

```bash
nano pgpass
```

```
192.168.1.100:5432:*:postgres:senha-do-postgres
192.168.1.100:5432:*:n8n:senha-do-n8n
```

```bash
chmod 600 pgpass
```

Atualize o `docker-compose.yml` para incluir o pgpass:

```yaml
    volumes:
      - pgadmin_data:/var/lib/pgadmin
      - ./servers.json:/pgadmin4/servers.json:ro
      - ./pgpass:/pgpass:ro
```

### 5. Iniciar

```bash
docker compose up -d
```

### 6. Verificar

```bash
docker compose ps
docker compose logs -f
```

## Acessar

- **Local**: `http://192.168.1.100:5050`
- **Login**: `admin@meuservidor.com` / `pgadmin-senha-forte`

## Configurar dominio no Cloudflare Tunnel

```bash
nano ~/.cloudflared/config.yml
```

```yaml
  - hostname: dbadmin.meuservidor.com
    service: http://localhost:5050
```

```bash
sudo systemctl restart cloudflared
```

## Conectar manualmente a um banco

1. Acesse o pgAdmin
2. Clique com direito em **Servers > Register > Server**
3. Aba **General**: Nome = `evolution-go`
4. Aba **Connection**:
   - Host: `192.168.1.100`
   - Port: `5432`
   - Username: `postgres`
   - Password: senha configurada
5. Salve

## Comandos uteis no Query Tool

```sql
-- Listar todos os bancos
SELECT datname FROM pg_database;

-- Ver conexoes ativas
SELECT pid, usename, application_name, state FROM pg_stat_activity;

-- Tamanho dos bancos
SELECT datname, pg_size_pretty(pg_database_size(datname)) FROM pg_database;

-- Limpar conexoes ociosas (antes de restaurar backup)
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE state = 'idle';
```

## Manutencao

### Atualizar

```bash
cd ~/pgadmin
docker compose pull
docker compose up -d
```

### Backup das configuracoes do pgAdmin

```bash
docker run --rm -v pgadmin_data:/source -v ~/backups:/backup alpine tar czf /backup/pgadmin-config-$(date +%Y%m%d).tar.gz -C /source .
```

## Troubleshooting

| Erro | Causa | Solucao |
|------|-------|---------|
| `Connection refused` ao conectar | PostgreSQL nao acessivel | Verificar se o Postgres esta rodando e acessivel pelo IP da VM |
| `Password authentication failed` | Senha errada | Verificar o `pgpass` e as configuracoes do PostgreSQL |
| Tela de login loop | `PGADMIN_CONFIG_SERVER_MODE` = True | Mudar para `False` e reiniciar |
| `FATAL: Ident authentication failed` | `pg_hba.conf` restritivo | Alterar `peer` para `md5` no `pg_hba.conf` do Postgres |

## Proximo passo

[Uptime Kuma](./11-uptime-kuma.md) - Monitoramento de uptime das aplicacoes.
