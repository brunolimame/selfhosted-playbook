# 08 - Mantenimiento

## 1. Actualizacion del sistema

### Actualizaciones regulares (recomendado: semanal)

```bash
ssh ubuntu@192.168.1.100
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y
```

### Verificar si necesita reiniciar

```bash
# Verificar si hay paquetes que requieren reboot
cat /var/run/reboot-required

# Verificar tiempo desde el ultimo boot
uptime
```

## 2. Actualizacion de Cloudflare Tunnel

```bash
# Descargar la ultima version
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb
sudo systemctl restart cloudflared
```

## 3. Actualizacion de Coolify

```bash
curl -fsSL https://cdn.coollabs.io/coolify/upgrade.sh | sudo bash
```

## 4. Actualizacion de Docker

```bash
sudo apt update
sudo apt upgrade -y docker-ce docker-ce-cli containerd.io
sudo systemctl restart docker
```

## 5. Backup

### Backup de los datos de Coolify

```bash
#!/bin/bash
# Script de backup ~/backup.sh

BACKUP_DIR="$HOME/backups"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

mkdir -p $BACKUP_DIR

# Detener Coolify temporalmente para backup consistente
sudo systemctl stop coolify

# Backup de los volumes Docker
docker run --rm \
  -v coolify_db:/source:ro \
  -v $BACKUP_DIR:/backup \
  ubuntu tar czf /backup/coolify-db-$DATE.tar.gz -C /source .

docker run --rm \
  -v coolify_redis:/source:ro \
  -v $BACKUP_DIR:/backup \
  ubuntu tar czf /backup/coolify-redis-$DATE.tar.gz -C /source .

# Backup de las configuraciones de Cloudflare Tunnel
cp ~/.cloudflared/config.yml $BACKUP_DIR/cloudflared-config-$DATE.yml

# Reiniciar Coolify
sudo systemctl start coolify

# Eliminar backups antiguos
find $BACKUP_DIR -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete
find $BACKUP_DIR -name "*.yml" -mtime +$RETENTION_DAYS -delete

echo "Backup completado: $DATE"
```

```bash
chmod +x ~/backup.sh
```

### Programar backup semanal

```bash
crontab -e
```

Anada:
```
0 3 * * 0 /home/ubuntu/backup.sh
```

(Ejecuta cada domingo a las 03:00)

### Copiar backup a otro lugar

**A su computador host**:
```bash
scp ubuntu@192.168.1.100:~/backups/coolify-db-*.tar.gz D:\backups\
```

**A la nube (opcional)**:
Instale rclone y configure:
```bash
sudo apt install -y rclone
rclone config
rclone copy ~/backups remote:mis-backups
```

## 6. Backup de la VM completa (snapshot VirtualBox)

En VirtualBox (con la VM apagada):
1. Seleccione la VM `ubuntu-server`
2. Haga clic en **Snapshots**
3. Haga clic en **Take Snapshot**
4. Nombre: `Antes-de-la-actualizacion-YYYY-MM-DD`

Para restaurar:
1. Seleccione el snapshot
2. Haga clic en **Restore**

## 7. Logs y monitoreo

### Verificar estado de los servicios

```bash
# Tunnel
sudo systemctl status cloudflared

# Coolify
sudo systemctl status coolify

# Docker
docker ps

# Recursos del sistema
htop

# Disco
df -h
```

### Logs centralizados

```bash
# Logs del tunnel
sudo journalctl -u cloudflared -n 100 --no-pager

# Logs de Coolify
sudo journalctl -u coolify -n 100 --no-pager

# Logs de acceso (si tiene Nginx)
sudo tail -f /var/log/nginx/access.log
```

### Monitoreo simple con cron

Cree un script de health check:

```bash
nano ~/healthcheck.sh
```

```bash
#!/bin/bash
# Verificar servicios y enviar alerta si algo esta detenido

SERVICES=("cloudflared" "coolify" "docker")
DATE=$(date)

for service in "${SERVICES[@]}"; do
  if ! systemctl is-active --quiet "$service"; then
    echo "[$DATE] ERROR: $service no esta ejecutandose" >> ~/healthcheck.log
    # Enviar email o notificacion (opcional)
    # curl -s -X POST "https://api.telegram.org/botTOKEN/sendMessage" -d "chat_id=CHAT_ID&text=ERROR: $service se detuvo"
  fi
done

# Verificar espacio en disco
USAGE=$(df / | tail -1 | awk '{print $5}' | tr -d '%')
if [ "$USAGE" -gt 90 ]; then
  echo "[$DATE] ALERTA: Disco con $USAGE% de uso" >> ~/healthcheck.log
fi
```

```bash
chmod +x ~/healthcheck.sh
```

Anada al crontab:
```bash
*/30 * * * * /home/ubuntu/healthcheck.sh
```

## 8. Reinicio programado (opcional)

Si la VM se vuelve inestable, programe un reinicio semanal:

```bash
sudo crontab -e
```

```
0 4 * * 0 /sbin/reboot
```

(Reinicia cada domingo a las 04:00)

## 9. Troubleshooting comun

| Problema | Causa posible | Solucion |
|----------|---------------|----------|
| Tunnel no conecta | VM sin internet | Verificar `ping 8.8.8.8` |
| Coolify no accesible | Container detenido | `docker ps` y `docker logs coolify` |
| Dominio no funciona | DNS no propagó | `dig vm.miservidor.com` |
| Disco lleno | Logs o imagenes Docker | `docker system prune -a` |
| VM lenta | Poca RAM | Aumentar RAM en VirtualBox |
| Error de permiso Docker | Usuario sin grupo docker | `sudo usermod -aG docker $USER` |

## Volver al inicio

[Indice de la documentacion](./README.md)
