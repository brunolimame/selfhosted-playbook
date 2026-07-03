# 08 - Manutencao

## 1. Atualizacao do sistema

### Atualizacoes regulares (recomendado: semanal)

```bash
ssh ubuntu@192.168.1.100
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y
```

### Verificar se precisa reiniciar

```bash
# Verificar se ha pacotes que precisam de reboot
cat /var/run/reboot-required

# Verificar tempo desde o ultimo boot
uptime
```

## 2. Atualizacao do Cloudflare Tunnel

```bash
# Baixar a ultima versao
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb
sudo systemctl restart cloudflared
```

## 3. Atualizacao do Coolify

```bash
curl -fsSL https://cdn.coollabs.io/coolify/upgrade.sh | sudo bash
```

## 4. Atualizacao do Docker

```bash
sudo apt update
sudo apt upgrade -y docker-ce docker-ce-cli containerd.io
sudo systemctl restart docker
```

## 5. Backup

### Backup dos dados do Coolify

```bash
#!/bin/bash
# Script de backup ~/backup.sh

BACKUP_DIR="$HOME/backups"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

mkdir -p $BACKUP_DIR

# Parar Coolify temporariamente para backup consistente
sudo systemctl stop coolify

# Backup dos volumes Docker
docker run --rm \
  -v coolify_db:/source:ro \
  -v $BACKUP_DIR:/backup \
  ubuntu tar czf /backup/coolify-db-$DATE.tar.gz -C /source .

docker run --rm \
  -v coolify_redis:/source:ro \
  -v $BACKUP_DIR:/backup \
  ubuntu tar czf /backup/coolify-redis-$DATE.tar.gz -C /source .

# Backup das configuracoes do Cloudflare Tunnel
cp ~/.cloudflared/config.yml $BACKUP_DIR/cloudflared-config-$DATE.yml

# Reiniciar Coolify
sudo systemctl start coolify

# Remover backups antigos
find $BACKUP_DIR -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete
find $BACKUP_DIR -name "*.yml" -mtime +$RETENTION_DAYS -delete

echo "Backup concluido: $DATE"
```

```bash
chmod +x ~/backup.sh
```

### Agendar backup semanal

```bash
crontab -e
```

Adicione:
```
0 3 * * 0 /home/ubuntu/backup.sh
```

(Executa todo domingo as 03:00)

### Copiar backup para outro local

**Para seu computador host**:
```bash
scp ubuntu@192.168.1.100:~/backups/coolify-db-*.tar.gz D:\backups\
```

**Para nuvem (opcional)**:
Instale o rclone e configure:
```bash
sudo apt install -y rclone
rclone config
rclone copy ~/backups remote:meus-backups
```

## 6. Backup da VM inteira (snapshot VirtualBox)

No VirtualBox (com a VM desligada):
1. Selecione a VM `ubuntu-server`
2. Clique em **Snapshots**
3. Clique em **Take Snapshot**
4. Nome: `Antes-da-atualizacao-YYYY-MM-DD`

Para restaurar:
1. Selecione o snapshot
2. Clique em **Restore**

## 7. Logs e monitoramento

### Verificar status dos servicos

```bash
# Tunnel
sudo systemctl status cloudflared

# Coolify
sudo systemctl status coolify

# Docker
docker ps

# Recursos do sistema
htop

# Disco
df -h
```

### Logs centralizados

```bash
# Logs do tunnel
sudo journalctl -u cloudflared -n 100 --no-pager

# Logs do Coolify
sudo journalctl -u coolify -n 100 --no-pager

# Logs de acesso (se tiver Nginx)
sudo tail -f /var/log/nginx/access.log
```

### Monitoramento simples com cron

Crie um script de health check:

```bash
nano ~/healthcheck.sh
```

```bash
#!/bin/bash
# Verificar servicos e enviar alerta se algo estiver parado

SERVICES=("cloudflared" "coolify" "docker")
DATE=$(date)

for service in "${SERVICES[@]}"; do
  if ! systemctl is-active --quiet "$service"; then
    echo "[$DATE] ERRO: $service nao esta rodando" >> ~/healthcheck.log
    # Enviar email ou notificacao (opcional)
    # curl -s -X POST "https://api.telegram.org/botTOKEN/sendMessage" -d "chat_id=CHAT_ID&text=ERRO: $service parou"
  fi
done

# Verificar espaco em disco
USAGE=$(df / | tail -1 | awk '{print $5}' | tr -d '%')
if [ "$USAGE" -gt 90 ]; then
  echo "[$DATE] ALERTA: Disco com $USAGE% de uso" >> ~/healthcheck.log
fi
```

```bash
chmod +x ~/healthcheck.sh
```

Adicione ao crontab:
```bash
*/30 * * * * /home/ubuntu/healthcheck.sh
```

## 8. Reinicializacao programada (opcional)

Se a VM ficar instavel, agende uma reinicializacao semanal:

```bash
sudo crontab -e
```

```
0 4 * * 0 /sbin/reboot
```

(Reinicia todo domingo as 04:00)

## 9. Troubleshooting comum

| Problema | Causa possivel | Solucao |
|----------|---------------|---------|
| Tunnel nao conecta | VM sem internet | Verificar `ping 8.8.8.8` |
| Coolify nao acessivel | Container parou | `docker ps` e `docker logs coolify` |
| Dominio nao funciona | DNS nao propagou | `dig vm.meuservidor.com` |
| Disco cheio | Logs ou imagens Docker | `docker system prune -a` |
| VM lenta | Pouca RAM | Aumentar RAM no VirtualBox |
| Erro de permissao Docker | Usuario sem grupo docker | `sudo usermod -aG docker $USER` |

## Voltar ao inicio

[Indice da documentacao](./README.md)
