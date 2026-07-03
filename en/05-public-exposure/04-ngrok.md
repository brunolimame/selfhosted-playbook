# 05-04 - ngrok

## Overview

[ngrok](https://ngrok.com) creates secure tunnels to expose local services publicly. It is extremely simple to configure, ideal for testing and demos.

```
Internet -> vm.meuservidor.com -> Cloudflare DNS -> ngrok Edge
                                                         |
                     VM (ngrok agent) <--- outbound tunnel -+
```

## 1. Create an ngrok account

1. Go to https://dashboard.ngrok.com/signup
2. Create a free account
3. Go to **Your Authtoken** and copy the token

## 2. Install ngrok on the VM

Access the VM via SSH:

```bash
ssh ubuntu@192.168.1.100
```

```bash
# Download
wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz

# Extract
tar xzf ngrok-v3-stable-linux-amd64.tgz

# Move to /usr/local/bin
sudo mv ngrok /usr/local/bin/ngrok

# Verify
ngrok version
```

## 3. Authenticate

```bash
ngrok config add-authtoken YOUR_AUTH_TOKEN
```

## 4. Expose Coolify

```bash
ngrok http 8000
```

Expected output:
```
Forwarding  https://abc123.ngrok-free.app -> http://localhost:8000
```

Access `https://abc123.ngrok-free.app` to see Coolify.

## 5. Use a custom domain with ngrok

### Free option: Fixed URL

On the free plan, you can reserve a `*.ngrok-free.app` subdomain:

1. In the ngrok dashboard, go to **Domains**
2. Create a domain (e.g., `meuvm.ngrok-free.app`)
3. Run:

```bash
ngrok http 8000 --domain=meuvm.ngrok-free.app
```

### Paid option: Custom domain

On paid plans, you can add your own domain:

1. In the dashboard, go to **Domains > Add a domain**
2. Add `vm.meuservidor.com`
3. In Cloudflare DNS, create a CNAME record:

| Type | Name | Content |
|------|------|----------|
| CNAME | `vm` | `meuvm.ngrok.app` |

With proxy **DNS Only** (orange cloud off).

4. Run:

```bash
ngrok http 8000 --domain=vm.meuservidor.com
```

## 6. Run ngrok in the background (service)

### Using nohup

```bash
nohup ngrok http 8000 --domain=meuvm.ngrok-free.app > ~/ngrok.log 2>&1 &
```

### Using systemd

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
ExecStart=/usr/local/bin/ngrok http 8000 --domain=meuvm.ngrok-free.app
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

## 7. ngrok Dashboard

Access `http://localhost:4040` on the VM (or via SSH tunnel) to see the local dashboard with real-time requests.

## Advantages of ngrok

- Extremely simple (1 command)
- Web dashboard with request inspection
- Webhook testing (request replay)
- Built-in authentication (Basic Auth, OAuth)

## Free plan limitations

- **4 tunnels per minute** (rate limit)
- **Random URL** changes on each start (unless reserved)
- **"ngrok-free.app" banner** on the screen
- **40 MB/minute** of traffic
- **3 simultaneous tunnels** maximum
- HTTP connections only (HTTPS at the ngrok edge)

## Next step

[Coolify - Installation](../06-coolify/README.md)
