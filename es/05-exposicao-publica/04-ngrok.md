# 05-04 - ngrok

## Visión general

[ngrok](https://ngrok.com) crea túneles seguros para exponer servicios locales públicamente. Es extremadamente simple de configurar, ideal para pruebas y demos.

```
Internet -> vm.meuservidor.com -> Cloudflare DNS -> ngrok Edge
                                                         |
                     VM (ngrok agent) <--- túnel outbound -+
```

## 1. Crear cuenta en ngrok

1. Acceda a https://dashboard.ngrok.com/signup
2. Cree una cuenta gratuita
3. Vaya a **Your Authtoken** y copie el token

## 2. Instalar ngrok en la VM

Acceda a la VM vía SSH:

```bash
ssh ubuntu@192.168.1.100
```

```bash
# Descargar
wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz

# Extraer
tar xzf ngrok-v3-stable-linux-amd64.tgz

# Mover a /usr/local/bin
sudo mv ngrok /usr/local/bin/ngrok

# Verificar
ngrok version
```

## 3. Autenticarse

```bash
ngrok config add-authtoken SU_AUTH_TOKEN
```

## 4. Exponer Coolify

```bash
ngrok http 8000
```

Salida esperada:
```
Forwarding  https://abc123.ngrok-free.app -> http://localhost:8000
```

Acceda a `https://abc123.ngrok-free.app` para ver Coolify.

## 5. Usar dominio propio con ngrok

### Opción gratuita: URL fija

En el plan gratuito, puede reservar un subdominio `*.ngrok-free.app`:

1. En el dashboard de ngrok, vaya a **Domains**
2. Cree un dominio (ej: `mivm.ngrok-free.app`)
3. Ejecute:

```bash
ngrok http 8000 --domain=mivm.ngrok-free.app
```

### Opción paga: Dominio personalizado

En los planes pagos, puede agregar su propio dominio:

1. En el dashboard, vaya a **Domains > Add a domain**
2. Agregue `vm.meuservidor.com`
3. En Cloudflare DNS, cree un registro CNAME:

| Tipo | Nombre | Contenido |
|------|--------|-----------|
| CNAME | `vm` | `mivm.ngrok.app` |

Con proxy **DNS Only** (naranja apagada).

4. Ejecute:

```bash
ngrok http 8000 --domain=vm.meuservidor.com
```

## 6. Ejecutar ngrok en background (servicio)

### Usando nohup

```bash
nohup ngrok http 8000 --domain=mivm.ngrok-free.app > ~/ngrok.log 2>&1 &
```

### Usando systemd

```bash
sudo nano /etc/systemd/system/ngrok.service
```

```ini
[Unit]
Description=ngrok tunnel
After=network.target

[Service]
Type=simple
User=ubuntu
ExecStart=/usr/local/bin/ngrok http 8000 --domain=mivm.ngrok-free.app
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable ngrok
sudo systemctl start ngrok
```

## 7. Dashboard de ngrok

Acceda a `http://localhost:4040` en la VM (o mediante túnel SSH) para ver el dashboard local con solicitudes en tiempo real.

## Ventajas de ngrok

- Extremadamente simple (1 comando)
- Dashboard web con inspección de solicitudes
- Webhook testing (replay de solicitudes)
- Autenticación integrada (Basic Auth, OAuth)

## Limitaciones del plan gratuito

- **4 túneles por minuto** (rate limit)
- **URL aleatoria** cambia cada inicio (a menos que reserve)
- **Aviso de "ngrok-free.app"** en la pantalla
- **40 MB/minuto** de tráfico
- **3 túneles simultáneos** como máximo
- Conexiones HTTP solamente (HTTPS en el borde de ngrok)

## Próximo paso

[Coolify - Instalación](../06-coolify/README.md)
