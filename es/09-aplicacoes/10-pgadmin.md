# 09-10 - pgAdmin

## ¿Qué es?

[pgAdmin](https://www.pgadmin.org) es la herramienta de administración open-source más popular para PostgreSQL. Ofrece interfaz web completa para gestionar bases de datos, ejecutar queries SQL, monitorear actividades y configurar replicación.

Funcionalidades principales:
- Interfaz web para gestionar múltiples servidores PostgreSQL
- Editor SQL con syntax highlighting y autocomplete
- Visualizador de esquemas (tablas, índices, vistas)
- Grafos de dependencias entre objetos
- Monitoreo de conexiones activas y locks
- Backup y restore con interfaz gráfica
- Importación/exportación de datos

## ¿Por qué pgAdmin en este proyecto?

Casi todas las aplicaciones de la VM usan PostgreSQL:

| App | Base de datos | Uso |
|-----|-------|-----|
| Coolify | `coolify` | Datos de la plataforma |
| n8n | `n8n` | Workflows, credenciales |
| evolution-go | `evogo_auth`, `evogo_users` | Instancias, usuarios |
| Typebot | `typebot` | Flujos, configuraciones |
| Chatwoot | `chatwoot` | Conversaciones, contactos, agentes |
| Dify | `dify` | Aplicaciones, documentos |
| Odysseus | `odysseus` | Datos del workspace |

Con pgAdmin, usted gestiona todos desde un solo lugar.

## Arquitectura

```
pgadmin (contenedor) -> Puerto 5050 (interfaz web)
    |
    Se conecta a cualquier PostgreSQL de la red
    (misma VM o externo)
```

## Prerrequisitos

- Docker instalado en la VM
- Al menos un servidor PostgreSQL funcionando (debe tenerlo de otras aplicaciones)

## Instalación

### 1. Acceder a la VM

```bash
ssh ubuntu@192.168.1.100
```

### 2. Crear directorio

```bash
mkdir -p ~/pgadmin
cd ~/pgadmin
```

### 3. Crear docker-compose.yml

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

Para que pgAdmin ya venga con los servidores configurados, cree el archivo `servers.json`:

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

Y el archivo de contraseñas:

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

Actualice el `docker-compose.yml` para incluir el pgpass:

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

## Acceder

- **Local**: `http://192.168.1.100:5050`
- **Login**: `admin@meuservidor.com` / `pgadmin-senha-forte`

## Configurar dominio en Cloudflare Tunnel

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

## Conectar manualmente a una base de datos

1. Acceda a pgAdmin
2. Haga clic derecho en **Servers > Register > Server**
3. Pestaña **General**: Nombre = `evolution-go`
4. Pestaña **Connection**:
   - Host: `192.168.1.100`
   - Port: `5432`
   - Username: `postgres`
   - Password: contraseña configurada
5. Guarde

## Comandos útiles en Query Tool

```sql
-- Listar todas las bases de datos
SELECT datname FROM pg_database;

-- Ver conexiones activas
SELECT pid, usename, application_name, state FROM pg_stat_activity;

-- Tamaño de las bases de datos
SELECT datname, pg_size_pretty(pg_database_size(datname)) FROM pg_database;

-- Limpiar conexiones ociosas (antes de restaurar backup)
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE state = 'idle';
```

## Mantenimiento

### Actualizar

```bash
cd ~/pgadmin
docker compose pull
docker compose up -d
```

### Backup de las configuraciones de pgAdmin

```bash
docker run --rm -v pgadmin_data:/source -v ~/backups:/backup alpine tar czf /backup/pgadmin-config-$(date +%Y%m%d).tar.gz -C /source .
```

## Troubleshooting

| Error | Causa | Solución |
|------|-------|---------|
| `Connection refused` al conectar | PostgreSQL no accesible | Verificar que Postgres esté funcionando y accesible por la IP de la VM |
| `Password authentication failed` | Contraseña incorrecta | Verificar el `pgpass` y las configuraciones de PostgreSQL |
| Pantalla de login en bucle | `PGADMIN_CONFIG_SERVER_MODE` = True | Cambiar a `False` y reiniciar |
| `FATAL: Ident authentication failed` | `pg_hba.conf` restrictivo | Cambiar `peer` a `md5` en el `pg_hba.conf` de Postgres |

## Próximo paso

[Uptime Kuma](./11-uptime-kuma.md) - Monitoreo de uptime de las aplicaciones.
