# Template para Anadir Nueva Aplicacion

Este archivo es un template/instruccion para LLMs (como yo) que necesitan documentar la instalacion de una nueva aplicacion en este proyecto.

## Instrucciones para el LLM

Al recibir la solicitud de anadir una nueva aplicacion, siga estos pasos:

1.  **Investigue** la aplicacion para entender:
    - Que hace (descripcion corta)
    - Arquitectura (puertos, dependencias, containers)
    - Forma de instalacion (Docker Compose, script, manual)
    - Requisitos de hardware y software
    - Dependencias externas (base de datos, colas, almacenamiento)

2.  **Cree el archivo** en `docs/09-aplicacoes/XX-nombre-de-la-app.md` siguiendo el template de abajo.

3.  **Actualice** los indices de navegacion:
    - `docs/09-aplicacoes/README.md` - anada la tabla de contenido
    - `docs/README.md` - anada en la seccion 09 si aplica

## Template de la documentacion

Copie el contenido de abajo y rellene los campos marcados con `[...]`:

```markdown
# 09-XX - Nombre de la Aplicacion

## Que es?

[Nombre de la aplicacion](URL-del-repositorio) es una breve descripcion de lo que hace.

Funcionalidades principales:
- Funcionalidad 1
- Funcionalidad 2
- Funcionalidad 3

## Arquitectura

```
[nombre-app] (container)
    Puerto: [PUERTO]
    |
    +-- [Dependencia 1] (ej: PostgreSQL, Redis, MySQL)
    +-- [Dependencia 2] (si la hay)
```

## Pre-requisitos

- Docker y Docker Compose instalados en la VM
- [Otros requisitos especificos: RAM, CPU, librerias]
- [Cuentas necesarias: API keys, servicios externos]

## Instalacion

### 1. Acceder a la VM

\`\`\`bash
ssh ubuntu@192.168.1.100
\`\`\`

### 2. Crear directorio del proyecto

\`\`\`bash
mkdir -p ~/[nombre-app]
cd ~/[nombre-app]
\`\`\`

### 3. Configurar variables de entorno

\`\`\`bash
nano .env
\`\`\`

\`\`\`env
# Puerto del servidor
PORT=[PUERTO]

# Configuraciones principales
# [anada aqui las variables esenciales]
API_KEY=su-clave-aqui

# Base de datos
DATABASE_URL=postgresql://usuario:contrasena@postgres:5432/[dbname]
\`\`\`

### 4. Crear docker-compose.yml

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
      POSTGRES_USER: usuario
      POSTGRES_PASSWORD: contrasena
      POSTGRES_DB: [dbname]
    volumes:
      - [app]_postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U usuario"]
      interval: 10s
      timeout: 5s
      retries: 5

  [app]:
    image: [imagen-oficial]:[tag]
    container_name: [app]
    restart: unless-stopped
    ports:
      - "[PUERTO]:[PUERTO]"
    environment:
      - PORT=\${PORT}
      - DATABASE_URL=\${DATABASE_URL}
      # [mas variables del .env]
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
cd ~/[nombre-app]
docker compose up -d
\`\`\`

### 6. Verificar

\`\`\`bash
docker compose ps
docker compose logs -f
\`\`\`

### 7. [Paso adicional si es necesario: crear base de datos, ejecutar migrations, etc.]

\`\`\`bash
# Ejemplo: crear base de datos
docker compose exec postgres psql -U usuario -c "CREATE DATABASE [dbname];"
\`\`\`

## Configurar dominio en Cloudflare Tunnel

Edite el config.yml:

\`\`\`bash
nano ~/.cloudflared/config.yml
\`\`\`

Anada antes del catch-all (404):

\`\`\`yaml
  - hostname: [subdominio].miservidor.com
    service: http://localhost:[PUERTO]
\`\`\`

Reinicie:

\`\`\`bash
sudo systemctl restart cloudflared
\`\`\`

## Acceder

- Local: \`http://192.168.1.100:[PUERTO]\`
- Publico: \`https://[subdominio].miservidor.com\`

## Integraciones utiles

- [Integracion 1: describa como esta app se conecta con n8n, Evolution Go, etc.]
- [Integracion 2]

## Mantenimiento

### Actualizar

\`\`\`bash
cd ~/[nombre-app]
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
# ~/[nombre-app]/backup.sh

DATE=\$(date +%Y%m%d_%H%M%S)
BACKUP_DIR=~/backups/[nombre-app]
mkdir -p \$BACKUP_DIR

# [Comandos de backup especificos]
docker compose exec -T postgres pg_dump -U usuario [dbname] > \$BACKUP_DIR/[app]-db-\$DATE.sql
gzip \$BACKUP_DIR/*.sql

echo "Backup completado: \$DATE"
\`\`\`

## Troubleshooting

| Error | Causa | Solucion |
|-------|-------|----------|
| [Error comun 1] | [Causa] | [Solucion] |
| [Error comun 2] | [Causa] | [Solucion] |
| [Error comun 3] | [Causa] | [Solucion] |
```

## Checklist para el LLM antes de finalizar

- [ ] Investigue la aplicacion oficialmente (sitio, GitHub, docs)
- [ ] Verifique el puerto predeterminado de la aplicacion
- [ ] Incluya las variables de entorno minimas para funcionar
- [ ] Incluya configuracion de dominio (Cloudflare Tunnel)
- [ ] Incluya seccion de mantenimiento (backup, logs, actualizacion)
- [ ] Incluya troubleshooting con errores comunes
- [ ] Actualice el `docs/09-aplicacoes/README.md` con la entrada en la tabla
- [ ] Actualice el `docs/README.md` si es necesario
- [ ] Use `192.168.1.100` como IP de ejemplo de la VM
- [ ] Use `miservidor.com` como dominio de ejemplo
- [ ] Numeracion del archivo sigue la secuencia (01, 02, 03...)
