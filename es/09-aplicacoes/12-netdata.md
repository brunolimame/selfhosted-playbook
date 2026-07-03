# 09-12 - Netdata

## ¿Qué es?

[Netdata](https://www.netdata.cloud) es un sistema de monitoreo en **tiempo real** que recolecta miles de métricas por segundo y las muestra en gráficos interactivos. Zero configuración: al iniciar, ya detecta y monitorea automáticamente todos los servicios y contenedores.

Funcionalidades principales:
- Métricas cada 1 segundo (no cada 60s como herramientas tradicionales)
- Auto-descubrimiento de servicios: PostgreSQL, Redis, Nginx, Docker, y 200+ más
- Dashboard web interactivo con gráficos zoom y arrastrables
- Alertas inteligentes configuradas automáticamente
- Consumo ligero: ~100-200 MB RAM, ~1% CPU
- Soporte a métricas históricas (configurable)

## ¿Por qué Netdata en este proyecto?

Uptime Kuma dice **si** el servicio está en línea. Netdata dice **por qué** se cayó:

| Escenario | Uptime Kuma | Netdata |
|---------|-------------|---------|
| "Sitio fuera de línea" | Alerta: down | - |
| "RAM se agotó" | - | Gráfico de memoria mostrando el pico |
| "CPU al 100%" | - | Qué proceso consumió |
| "Disco lleno" | - | Exactamente qué directorio |
| "Puerto se cerró" | Alerta: timeout | Logs del sistema |

## Arquitectura

```
netdata (contenedor)
    Puerto: 19999 (dashboard)
    |
    +-- Accede a /proc, /sys del host (read-only)
    +-- Accede a /var/run/docker.sock (contenedores)
    +-- Monitorea automáticamente: CPU, RAM, disco, red, Docker, PostgreSQL, Redis...
```

## Prerrequisitos

- Docker instalado en la VM
- Mínimo 512 MB RAM libres (Netdata usa ~100-200 MB)
- Privilegios especiales en el contenedor para acceder a métricas del sistema

## Instalación

### 1. Acceder a la VM

```bash
ssh ubuntu@192.168.1.100
```

### 2. Crear directorio

```bash
mkdir -p ~/netdata
cd ~/netdata
```

### 3. Crear docker-compose.yml

```bash
nano docker-compose.yml
```

```yaml
services:
  netdata:
    image: netdata/netdata:stable
    container_name: netdata
    hostname: ubuntu-vm
    restart: unless-stopped
    pid: host
    network_mode: host
    cap_add:
      - SYS_PTRACE
      - SYS_ADMIN
    security_opt:
      - apparmor:unconfined
    volumes:
      - netdata_config:/etc/netdata
      - netdata_lib:/var/lib/netdata
      - netdata_cache:/var/cache/netdata
      - /:/host/root:ro,rslave
      - /etc/passwd:/host/etc/passwd:ro
      - /etc/group:/host/etc/group:ro
      - /etc/localtime:/etc/localtime:ro
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /etc/os-release:/host/etc/os-release:ro
      - /var/log:/host/var/log:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro

volumes:
  netdata_config:
  netdata_lib:
  netdata_cache:
```

> **Nota**: `network_mode: host` hace que Netdata use el puerto 19999 directamente en la IP de la VM, sin mapeo de puerto.

### 4. Iniciar

```bash
docker compose up -d
```

### 5. Verificar

```bash
docker compose ps
docker compose logs -f
```

## Acceder

- **Local**: `http://192.168.1.100:19999`
- No necesita login - el dashboard abre directamente con todas las métricas.

## Configurar dominio en Cloudflare Tunnel

```bash
nano ~/.cloudflared/config.yml
```

```yaml
  - hostname: netdata.meuservidor.com
    service: http://localhost:19999
```

```bash
sudo systemctl restart cloudflared
```

## Qué monitorea Netdata automáticamente

### Sistema
- CPU (por núcleo, por proceso)
- RAM (usada, buffer, caché, swap)
- Disco (I/O, uso, inodes)
- Red (interfaz por interfaz, protocolos)
- Procesos (top 10 por CPU/RAM)

### Docker
- Cada contenedor: CPU, RAM, red, disco
- Logs centralizados

### Servicios (detectados automáticamente)
- **PostgreSQL**: queries, conexiones, locks, cache hit ratio
- **Redis**: hits, misses, memoria, conexiones
- **Nginx**: peticiones, conexiones, errores (si tiene)
- **MySQL/MariaDB**: queries, threads, buffer pool

## Alertas útiles preconfiguradas

Netdata ya viene con alertas inteligentes:

| Alerta | Gatillo | Acción sugerida |
|--------|---------|---------------|
| RAM > 80% | Uso de memoria alto | Verificar contenedores con mayor consumo |
| CPU > 90% | Procesador sobrecargado | Identificar proceso en el dashboard |
| Disco > 85% | Casi lleno | Ejecutar `docker system prune` |
| Swap > 50% | RAM insuficiente | Aumentar RAM de la VM |
| PostgreSQL connections > 100 | Muchas conexiones | Verificar pooling o app con leak |

Para configurar notificaciones:
1. Acceda a `http://192.168.1.100:19999`
2. Haga clic en **Alertas > Notificaciones**
3. Añada Telegram, Discord, Email o Slack

## Personalizar retención de datos

Por defecto Netdata almacena ~2 horas de métricas en memoria. Para aumentar:

```bash
nano docker-compose.yml
```

Añada en el `environment`:

```yaml
    environment:
      - NETDATA_PAGE_CACHE_SIZE=32
      - NETDATA_DBENGINE_SIZE=256
```

Esto aumenta la retención a varios días.

O edite el archivo de configuración:

```bash
docker compose exec netdata /etc/netdata/edit-config netdata.conf
```

```ini
[global]
    page cache size = 32
    dbengine multihost disk space = 256
```

## Integraciones con otras herramientas

### Grafana (si tiene)
Netdata puede exportar métricas a Prometheus, que Grafana consume:

```bash
# En el archivo de configuración
docker compose exec netdata /etc/netdata/edit-config go.d/prometheus.conf
```

### n8n
n8n puede consultar la API de Netdata para tomar decisiones:

```bash
# Obtener uso actual de CPU
curl -s http://192.168.1.100:19999/api/v1/data?chart=system.cpu | jq '.result[0].value[1]'
```

## Mantenimiento

### Actualizar

```bash
cd ~/netdata
docker compose pull
docker compose up -d
```

### Verificar espacio usado

```bash
docker run --rm -v netdata_cache:/source alpine du -sh /source
```

### Logs

```bash
docker compose logs -f --tail 100
```

## Troubleshooting

| Error | Causa | Solución |
|------|-------|---------|
| Dashboard no carga | Puerto no accesible | `docker compose ps` para ver si subió |
| "Permission denied" | Faltan privilegios | Verificar `cap_add` y `security_opt` |
| Métricas de Docker vacías | Socket no montado | Verificar `/var/run/docker.sock` en volumes |
| Consumo alto de RAM | DBENGINE muy grande | Reducir `dbengine multihost disk space` |
| Gráficos sin datos | Acaba de iniciar | Esperar 30s para primera recolección |

## Próximo paso

Volver a [Índice de Aplicaciones](./README.md).
