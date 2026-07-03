# 07 - Seguridad

## 1. Cortafuegos (UFW)

Habilite el cortafuegos en la VM y permita solo los puertos necesarios:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH (esencial)
sudo ufw allow 22/tcp comment 'SSH'

# Si usa DDNS + Port Forwarding (metodo alternativo)
# sudo ufw allow 80/tcp comment 'HTTP'
# sudo ufw allow 443/tcp comment 'HTTPS'
# sudo ufw allow 8000/tcp comment 'Coolify'

sudo ufw enable
sudo ufw status verbose
```

Con Cloudflare Tunnel, ningun puerto HTTP necesita abrirse. Solo SSH para administracion.

## 2. SSH con clave (deshabilitar contrasena)

### Generar clave en su host (si aun no tiene)

**Windows (PowerShell)**:
```powershell
ssh-keygen -t ed25519 -C "su-clave"
```

**Linux/macOS**:
```bash
ssh-keygen -t ed25519 -C "su-clave"
```

### Copiar clave a la VM

```bash
ssh-copy-id ubuntu@192.168.1.100
```

O manualmente:
```bash
cat ~/.ssh/id_ed25519.pub | ssh ubuntu@192.168.1.100 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### Deshabilitar inicio de sesion por contrasena

```bash
sudo nano /etc/ssh/sshd_config
```

Cambie:
```
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
```

```bash
sudo systemctl restart sshd
```

## 3. Actualizaciones automaticas

```bash
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

Seleccione **Yes** para actualizar automaticamente.

## 4. Fail2ban (proteccion contra fuerza bruta)

```bash
sudo apt install -y fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

## 5. Coolify - Configuraciones de seguridad

En el panel de Coolify:
1. **Settings > Registration**: Deshabilite el registro publico
2. **Settings > Instance**: Force HTTPS si esta detras de proxy
3. **Settings > Token**: Genere un token de API con permisos minimos
4. **Applications**: Evite exponer puertos de gestion (ej: 8000) directamente

## 6. Cloudflare - WAF (Web Application Firewall)

En el panel de Cloudflare:
1. **Security > WAF**:

**Bloquear acceso directo por IP**:
Regla personalizada:
- Campo: `IP Source Address`
- Operador: `equals`
- Valor: `192.168.1.100`
- Accion: `Block`

**Rate Limiting** (gratuito: 10 reglas):
- Protege contra ataques de fuerza bruta
- Limite: 100 peticiones por minuto por IP

## 7. Cloudflare Tunnel - Acceso restringido

En el archivo `config.yml` de cloudflared, puede anadir autenticacion:

```yaml
ingress:
  - hostname: admin.vm.miservidor.com
    service: http://localhost:8000
    originRequest:
      accessToken: "su-token-aqui"
  - service: http_status:404
```

O use **Cloudflare Access** para proteger rutas con autenticacion SSO:
1. En Cloudflare, vaya a **Access > Applications**
2. Cree una aplicacion para `vm.miservidor.com`
3. Configure politicas de acceso (email, SSO, etc.)

## 8. Backups cifrados

```bash
# Backup del directorio de Coolify
sudo tar czf ~/backup-coolify-$(date +%Y%m%d).tar.gz /var/lib/docker/volumes/coolify_* --transform 's,.*/,,'
```

Cifre con GPG:
```bash
gpg --symmetric --cipher-algo AES256 backup-coolify-*.tar.gz
```

## 9. Checklist de seguridad

- [ ] Cortafuegos UFW activo (solo SSH permitido)
- [ ] Inicio de sesion SSH solo por clave
- [ ] Root login deshabilitado
- [ ] Unattended upgrades activo
- [ ] Fail2ban instalado y ejecutandose
- [ ] Coolify con registro publico deshabilitado
- [ ] Cloudflare SSL/TLS en Full (strict)
- [ ] Cloudflare WAF configurado
- [ ] Backups automatizados
- [ ] Docker accesible solo via socket (`/var/run/docker.sock`)

## Proximo paso

[Mantenimiento](../08-mantenimiento.md) - Backup, actualizaciones y monitoreo.
