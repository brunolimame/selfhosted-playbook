# 05-01 - Cloudflare Tunnel

## Visión general

Cloudflare Tunnel crea un túnel seguro y cifrado entre su VM y la red edge de Cloudflare. El `cloudflared` (agente) se ejecuta en la VM y establece conexiones outbound hacia Cloudflare. Ningún puerto necesita ser abierto en el router.

```
Internet -> vm.meuservidor.com -> Cloudflare DNS -> Cloudflare Edge
                                                         |
                     VM (cloudflared) <--- túnel outbound -+
```

## 1. Instalar cloudflared en la VM

Acceda a la VM vía SSH:

```bash
ssh ubuntu@192.168.1.100
```

### Descarga e instalación

```bash
# Descargar cloudflared
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb

# Instalar
sudo dpkg -i cloudflared-linux-amd64.deb

# Verificar
cloudflared version
```

## 2. Autenticarse con Cloudflare

```bash
cloudflared tunnel login
```

Esto abrirá una URL en el terminal. Como es una VM sin navegador, copie la URL y ábrala en el navegador de su host.

1. Copie la URL mostrada en el terminal
2. Péguela en el navegador de su computadora host
3. Inicie sesión en Cloudflare (si es necesario)
4. Seleccione el dominio que añadió a Cloudflare
5. Haga clic en **Authorize**

Después de autorizar, el archivo de certificado se descargará automáticamente en `~/.cloudflared/cert.pem`.

## 3. Crear el túnel

```bash
cloudflared tunnel create mi-tunel
```

Esto crea un túnel con un ID único (ej: `abcdef01-1234-5678-9abc-def012345678`). Anote este ID.

Salida esperada:
```
Created tunnel mi-tunel with id abcdef01-1234-5678-9abc-def012345678
```

## 4. Configurar el túnel

Cree el archivo de configuración:

```bash
nano ~/.cloudflared/config.yml
```

Contenido:

```yaml
tunnel: mi-tunel
credentials-file: /home/ubuntu/.cloudflared/abcdef01-1234-5678-9abc-def012345678.json

ingress:
  - hostname: vm.meuservidor.com
    service: http://localhost:8000
  - service: http_status:404
```

Explicación:
- `tunnel`: nombre del túnel creado
- `credentials-file`: ruta al archivo de credenciales del túnel (ajuste el nombre del archivo)
- `ingress`: mapea el hostname `vm.meuservidor.com` a Coolify ejecutándose en el puerto `8000` de la VM
- La última línea es una regla catch-all que retorna 404

## 5. Crear registro DNS

Ahora apunte el DNS de Cloudflare al túnel:

```bash
cloudflared tunnel route dns mi-tunel vm.meuservidor.com
```

Esto crea un registro CNAME en Cloudflare apuntando `vm.meuservidor.com` al ID del túnel.

## 6. Ejecutar el túnel como servicio

### Probar primero

```bash
cloudflared tunnel run mi-tunel
```

Si funciona, detenga con `Ctrl+C` y configúrelo como servicio.

### Instalar como servicio systemd

```bash
sudo cloudflared service install
```

O manualmente:

```bash
sudo nano /etc/systemd/system/cloudflared.service
```

Contenido:

```ini
[Unit]
Description=Cloudflare Tunnel
After=network.target

[Service]
Type=simple
User=ubuntu
ExecStart=/usr/bin/cloudflared tunnel run mi-tunel
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Activar e iniciar:

```bash
sudo systemctl daemon-reload
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
```

Verificar estado:

```bash
sudo systemctl status cloudflared
```

## 7. Ajustar SSL/TLS en Cloudflare

En el panel de Cloudflare:
1. Vaya a **SSL/TLS > Overview**
2. Seleccione **Full (strict)**
3. Vaya a **Edge Certificates** y active **Always Use HTTPS**

## 8. Probar el acceso

En el navegador, acceda a:

```
https://vm.meuservidor.com
```

Si Coolify aún no está instalado, verá un error de conexión rechazada. Esto es normal — continúe con la instalación de Coolify.

## Mantenimiento del túnel

### Ver logs
```bash
sudo journalctl -u cloudflared -f
```

### Actualizar cloudflared
```bash
sudo dpkg -i cloudflared-linux-amd64.deb
sudo systemctl restart cloudflared
```

### Eliminar el túnel (si es necesario)
```bash
cloudflared tunnel delete mi-tunel
```

## Limitaciones del plan Free de Cloudflare
- **Sin streaming de video** (viola los términos)
- **Máximo 100 MB por request** para descarga vía proxy
- **3 reglas de Page Rules** gratuitas

## Próximo paso

[Coolify - Instalación](../06-coolify/README.md)
