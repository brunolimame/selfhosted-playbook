# 05-02 - DDNS + Port Forwarding

## Visión general

Este método tradicional expone la VM directamente en Internet usando:
- **DDNS (Dynamic DNS)**: Un servicio que actualiza automáticamente un registro DNS cuando su IP pública cambia
- **Port Forwarding**: Redireccionamiento de puertos del router a la VM

```
Internet -> vm.meuservidor.com -> Router (IP dinámica)
                                       |
                                    Port Forwarding
                                       |
                               VM (192.168.1.100)
```

## Aviso de seguridad

Este método **abre puertos en el router**, exponiendo su red directamente. Solo úselo si:
- Entiende los riesgos de seguridad
- Tiene configuración de firewall adecuada
- Mantiene el sistema siempre actualizado

## 1. Configurar DDNS (DNS Dinámico)

### Opción 1: DuckDNS (gratuito, simple)

1. Acceda a https://www.duckdns.org
2. Inicie sesión con una cuenta Google, GitHub o Twitter
3. Cree un subdominio (ej: `mivm.duckdns.org`)
4. Anote el **token** generado

### Opción 2: No-IP (gratuito, necesita renovar mensualmente)

1. Acceda a https://www.noip.com
2. Cree una cuenta gratuita
3. Cree un hostname (ej: `mivm.hopto.org`)
4. Instale el cliente No-IP en la VM

### Instalar cliente DDNS en la VM

**Para DuckDNS**:

Cree un script de actualización:

```bash
nano ~/duckdns.sh
```

```bash
#!/bin/bash
echo url="https://www.duckdns.org/update?domains=mivm&token=SU_TOKEN&ip=" | curl -k -o ~/duckdns.log -K -
```

```bash
chmod +x ~/duckdns.sh
```

Agréguelo al crontab para ejecutar cada 5 minutos:

```bash
crontab -e
```

Agregue la línea:
```
*/5 * * * * /home/ubuntu/duckdns.sh
```

Si tiene un dominio propio y quiere usar DDNS con Cloudflare, use la API de Cloudflare para actualizar el DNS. Herramientas como `ddclient` o `cloudflare-ddns` pueden ayudar.

## 2. Configurar Port Forwarding en el router

1. Acceda al panel del router (generalmente `192.168.1.1`)
2. Inicie sesión (admin/admin o usuario/contraseña en la etiqueta del router)
3. Localice **Port Forwarding** o **Virtual Server** o **NAT**
4. Cree las reglas:

| Puerto externo | IP interna | Puerto interno | Protocolo | Descripción |
|----------------|------------|----------------|-----------|-------------|
| 80 | 192.168.1.100 | 80 | TCP | HTTP (redirigir a HTTPS) |
| 443 | 192.168.1.100 | 443 | TCP | HTTPS (Coolify o proxy) |
| 8000 | 192.168.1.100 | 8000 | TCP | Coolify Web UI |

5. Guarde las configuraciones

## 3. Configurar firewall en la VM (UFW)

```bash
sudo ufw allow 22/tcp   # SSH
sudo ufw allow 80/tcp   # HTTP
sudo ufw allow 443/tcp  # HTTPS
sudo ufw allow 8000/tcp # Coolify

sudo ufw enable
sudo ufw status verbose
```

## 4. Proxy inverso (recomendado)

Para servir múltiples aplicaciones en el puerto 80/443, instale un proxy inverso como Nginx o Caddy:

```bash
sudo apt install -y nginx
```

Ejemplo de configuración Nginx para Coolify:

```nginx
server {
    listen 80;
    server_name vm.meuservidor.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name vm.meuservidor.com;

    ssl_certificate /etc/ssl/certs/cloudflare.crt;
    ssl_certificate_key /etc/ssl/private/cloudflare.key;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Para SSL, use Let's Encrypt (certbot):
```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d vm.meuservidor.com
```

## 5. Probar

1. Descubra su IP pública: `curl ifconfig.me`
2. Acceda a `http://SU_IP_PUBLICA:8000` (temporal)
3. Si funciona, acceda a `https://vm.meuservidor.com`

## Limitaciones y riesgos

- **La IP puede cambiar**: Si el DDNS falla, el servicio queda inaccesible
- **Puertos abiertos**: Cualquier vulnerabilidad en la VM expone su red interna
- **ISP puede bloquear**: Algunos proveedores bloquean puertos 80/443 en planes residenciales
- **Sin protección DDoS**: Los ataques van directo a su conexión

## Próximo paso

[Coolify - Instalación](../06-coolify/README.md)
