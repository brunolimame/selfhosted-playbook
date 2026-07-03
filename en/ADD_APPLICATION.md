# Template for Adding a New Application

This file is a template/instruct for LLMs (like me) that need to document the installation of a new application in this project.

## Instructions for the LLM

When receiving a request to add a new application, follow these steps:

1.  **Research** the application to understand:
    - What it does (short description)
    - Architecture (ports, dependencies, containers)
    - Installation method (Docker Compose, script, manual)
    - Hardware and software requirements
    - External dependencies (database, queues, storage)

2.  **Create the file** at `docs/09-applications/XX-app-name.md` following the template below.

3.  **Update** the navigation indexes:
    - `docs/09-applications/README.md` - add to the table of contents
    - `docs/README.md` - add to section 09 if applicable

## Documentation template

Copy the content below and fill in the fields marked with `[...]`:

```markdown
# 09-XX - Application Name

## What is it?

[Application name](repository-URL) is a brief description of what it does.

Main features:
- Feature 1
- Feature 2
- Feature 3

## Architecture

```
[app-name] (container)
    Port: [PORT]
    |
    +-- [Dependency 1] (ex: PostgreSQL, Redis, MySQL)
    +-- [Dependency 2] (if any)
```

## Prerequisites

- Docker and Docker Compose installed on the VM
- [Other specific requirements: RAM, CPU, libraries]
- [Required accounts: API keys, external services]

## Installation

### 1. Access the VM

\`\`\`bash
ssh ubuntu@192.168.1.100
\`\`\`

### 2. Create project directory

\`\`\`bash
mkdir -p ~/[app-name]
cd ~/[app-name]
\`\`\`

### 3. Configure environment variables

\`\`\`bash
nano .env
\`\`\`

\`\`\`env
# Server port
PORT=[PORT]

# Main settings
# [add essential variables here]
API_KEY=your-key-here

# Database
DATABASE_URL=postgresql://user:password@postgres:5432/[dbname]
\`\`\`

### 4. Create docker-compose.yml

\`\`\`bash
nano docker-compose.yml
\`\`\`

\`\`\`yaml
services:
  postgres:
    image: postgres:[version]-alpine
    container_name: [app]-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: [dbname]
    volumes:
      - [app]_postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5

  [app]:
    image: [official-image]:[tag]
    container_name: [app]
    restart: unless-stopped
    ports:
      - "[PORT]:[PORT]"
    environment:
      - PORT=\${PORT}
      - DATABASE_URL=\${DATABASE_URL}
      # [more .env variables]
    volumes:
      - [app]_data:/app/data
    depends_on:
      postgres:
        condition: service_healthy

volumes:
  [app]_postgres_data:
  [app]_data:
\`\`\`

### 5. Start

\`\`\`bash
cd ~/[app-name]
docker compose up -d
\`\`\`

### 6. Verify

\`\`\`bash
docker compose ps
docker compose logs -f
\`\`\`

### 7. [Additional step if needed: create database, run migrations, etc.]

\`\`\`bash
# Example: create database
docker compose exec postgres psql -U user -c "CREATE DATABASE [dbname];"
\`\`\`

## Configure domain in Cloudflare Tunnel

Edit config.yml:

\`\`\`bash
nano ~/.cloudflared/config.yml
\`\`\`

Add before the catch-all (404):

\`\`\`yaml
  - hostname: [subdomain].myserver.com
    service: http://localhost:[PORT]
\`\`\`

Restart:

\`\`\`bash
sudo systemctl restart cloudflared
\`\`\`

## Access

- Local: \`http://192.168.1.100:[PORT]\`
- Public: \`https://[subdomain].myserver.com\`

## Useful integrations

- [Integration 1: describe how this app connects with n8n, Evolution Go, etc.]
- [Integration 2]

## Maintenance

### Update

\`\`\`bash
cd ~/[app-name]
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
# ~/[app-name]/backup.sh

DATE=\$(date +%Y%m%d_%H%M%S)
BACKUP_DIR=~/backups/[app-name]
mkdir -p \$BACKUP_DIR

# [Specific backup commands]
docker compose exec -T postgres pg_dump -U user [dbname] > \$BACKUP_DIR/[app]-db-\$DATE.sql
gzip \$BACKUP_DIR/*.sql

echo "Backup completed: \$DATE"
\`\`\`

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| [Common error 1] | [Cause] | [Solution] |
| [Common error 2] | [Cause] | [Solution] |
| [Common error 3] | [Cause] | [Solution] |
```

## Checklist for the LLM before finalizing

- [ ] I researched the application officially (website, GitHub, docs)
- [ ] I verified the default application port
- [ ] I included the minimum environment variables to run
- [ ] I included domain configuration (Cloudflare Tunnel)
- [ ] I included a maintenance section (backup, logs, update)
- [ ] I included troubleshooting with common errors
- [ ] I updated `docs/09-applications/README.md` with the table entry
- [ ] I updated `docs/README.md` if necessary
- [ ] I used `192.168.1.100` as example VM IP
- [ ] I used `myserver.com` as example domain
- [ ] File numbering follows the sequence (01, 02, 03...)
