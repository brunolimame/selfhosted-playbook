# 09-10 - pgAdmin

## What is it?

[pgAdmin](https://www.pgadmin.org) is the most popular open-source administration tool for PostgreSQL. It offers a complete web interface for managing databases, running SQL queries, monitoring activity, and configuring replication.

Main features:
- Web interface to manage multiple PostgreSQL servers
- SQL editor with syntax highlighting and autocomplete
- Schema viewer (tables, indexes, views)
- Dependency graphs between objects
- Active connection and lock monitoring
- Backup and restore with graphical interface
- Data import/export

## Why pgAdmin in this project?

Almost all applications on the VM use PostgreSQL:

| App | Database | Usage |
|-----|-------|-----|
| Coolify | `coolify` | Platform data |
| n8n | `n8n` | Workflows, credentials |
| evolution-go | `evogo_auth`, `evogo_users` | Instances, users |
| Typebot | `typebot` | Flows, configurations |
| Chatwoot | `chatwoot` | Conversations, contacts, agents |
| Dify | `dify` | Applications, documents |
| Odysseus | `odysseus` | Workspace data |

With pgAdmin, you manage everything from one place.

## Architecture

```
pgadmin (container) -> Port 5050 (web interface)
    |
    Connects to any PostgreSQL on the network
    (same VM or external)
```

## Prerequisites

- Docker installed on the VM
- At least one PostgreSQL server running (should already have from other applications)

## Installation

### 1. Access the VM

```bash
ssh ubuntu@192.168.1.100
```

### 2. Create directory

```bash
mkdir -p ~/pgadmin
cd ~/pgadmin
```

### 3. Create docker-compose.yml

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

### 4. (Optional) Auto-configure servers

For pgAdmin to come with pre-configured servers, create the `servers.json` file:

```bash
nano servers.json
```

```json
{
  "Servers": {
    "1": {
      "Name": "Coolify DB",
      "Group": "VM Servers",
      "Host": "192.168.1.100",
      "Port": 5432,
      "MaintenanceDB": "postgres",
      "Username": "postgres",
      "PassFile": "/pgpass",
      "SSLMode": "prefer"
    },
    "2": {
      "Name": "n8n DB",
      "Group": "VM Servers",
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

And the password file:

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

Update the `docker-compose.yml` to include pgpass:

```yaml
    volumes:
      - pgadmin_data:/var/lib/pgadmin
      - ./servers.json:/pgadmin4/servers.json:ro
      - ./pgpass:/pgpass:ro
```

### 5. Start

```bash
docker compose up -d
```

### 6. Verify

```bash
docker compose ps
docker compose logs -f
```

## Access

- **Local**: `http://192.168.1.100:5050`
- **Login**: `admin@meuservidor.com` / `pgadmin-senha-forte`

## Configure domain on Cloudflare Tunnel

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

## Manually connect to a database

1. Access pgAdmin
2. Right-click on **Servers > Register > Server**
3. **General** tab: Name = `evolution-go`
4. **Connection** tab:
   - Host: `192.168.1.100`
   - Port: `5432`
   - Username: `postgres`
   - Password: configured password
5. Save

## Useful commands in Query Tool

```sql
-- List all databases
SELECT datname FROM pg_database;

-- View active connections
SELECT pid, usename, application_name, state FROM pg_stat_activity;

-- Database sizes
SELECT datname, pg_size_pretty(pg_database_size(datname)) FROM pg_database;

-- Terminate idle connections (before restoring backup)
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE state = 'idle';
```

## Maintenance

### Update

```bash
cd ~/pgadmin
docker compose pull
docker compose up -d
```

### Backup pgAdmin configurations

```bash
docker run --rm -v pgadmin_data:/source -v ~/backups:/backup alpine tar czf /backup/pgadmin-config-$(date +%Y%m%d).tar.gz -C /source .
```

## Troubleshooting

| Error | Cause | Solution |
|------|-------|---------|
| `Connection refused` when connecting | PostgreSQL not accessible | Check if Postgres is running and accessible via VM IP |
| `Password authentication failed` | Wrong password | Check `pgpass` and PostgreSQL settings |
| Login screen loop | `PGADMIN_CONFIG_SERVER_MODE` = True | Change to `False` and restart |
| `FATAL: Ident authentication failed` | Restrictive `pg_hba.conf` | Change `peer` to `md5` in Postgres `pg_hba.conf` |

## Next step

[Uptime Kuma](./11-uptime-kuma.md) - Application uptime monitoring.
