# 06-01 - Coolify Installation

## 1. Install Docker on the VM

Access the VM via SSH:

```bash
ssh ubuntu@192.168.1.100
```

### Official method (recommended)

```bash
curl -fsSL https://get.docker.com | sh
```

### Or manually

```bash
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Add user to docker group

```bash
sudo usermod -aG docker $USER
newgrp docker
```

### Verify Docker

```bash
docker --version
docker compose version
```

## 2. Install Coolify

### Official script

```bash
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | sudo bash
```

The script:
1. Downloads the latest version of Coolify
2. Creates the necessary containers (coolify, postgres, redis)
3. Configures the systemd service
4. Displays the auto-generated admin password

Expected output:
```
Coolify is now running on http://localhost:8000
Email: admin@example.com
Password: [auto-generated-password]
```

**Important**: Write down the email and password displayed.

### Monitor the installation

```bash
sudo journalctl -u coolify -f
```

## 3. Access Coolify

### Via local network

In your host's browser, go to:

```
http://192.168.1.100:8000
```

### Via domain (if the tunnel is already configured)

```
https://vm.meuservidor.com
```

## 4. Initial setup

1. Log in with the email and password displayed during installation
2. Go to **Settings > Instance Name** and set a name (e.g., `My Server`)
3. Go to **Settings > Timezone** and select your timezone (e.g., `America/Sao_Paulo`)
4. Go to **Settings > Wildcard Domain** and configure `*.vm.meuservidor.com` (if you have the domain)
5. Go to **Settings > Registration** and disable public registration (security)

## 5. Check status

```bash
docker ps
```

You should see containers like:
- `coolify` (main application)
- `coolify-db` (PostgreSQL)
- `coolify-redis` (cache/queue)

## 6. Update Coolify

```bash
curl -fsSL https://cdn.coollabs.io/coolify/upgrade.sh | sudo bash
```

## Next step

[First deploy](./02-applications.md) - Deploy a sample application.
