# 06-01 - Instalación de Coolify

## 1. Instalar Docker en la VM

Acceda a la VM vía SSH:

```bash
ssh ubuntu@192.168.1.100
```

### Método oficial (recomendado)

```bash
curl -fsSL https://get.docker.com | sh
```

### O manualmente

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

### Agregar usuario al grupo docker

```bash
sudo usermod -aG docker $USER
newgrp docker
```

### Verificar Docker

```bash
docker --version
docker compose version
```

## 2. Instalar Coolify

### Script oficial

```bash
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | sudo bash
```

El script:
1. Descarga la última versión de Coolify
2. Crea los contenedores necesarios (coolify, postgres, redis)
3. Configura el servicio systemd
4. Muestra la contraseña de administrador generada automáticamente

Salida esperada:
```
Coolify is now running on http://localhost:8000
Email: admin@example.com
Password: [contraseña-generada-automáticamente]
```

**Importante**: Anote el email y la contraseña mostrados.

### Seguir la instalación

```bash
sudo journalctl -u coolify -f
```

## 3. Acceder a Coolify

### Por la red local

En el navegador de su host, acceda a:

```
http://192.168.1.100:8000
```

### Por el dominio (si el túnel ya está configurado)

```
https://vm.meuservidor.com
```

## 4. Configuración inicial

1. Inicie sesión con el email y la contraseña mostrados en la instalación
2. Vaya a **Settings > Instance Name** y defina un nombre (ej: `Mi Servidor`)
3. Vaya a **Settings > Timezone** y seleccione su zona horaria (ej: `America/Sao_Paulo`)
4. Vaya a **Settings > Wildcard Domain** y configure `*.vm.meuservidor.com` (si tiene el dominio)
5. Vaya a **Settings > Registration** y deshabilite el registro público (seguridad)

## 5. Verificar estado

```bash
docker ps
```

Debería ver contenedores como:
- `coolify` (aplicación principal)
- `coolify-db` (PostgreSQL)
- `coolify-redis` (cache/queue)

## 6. Actualizar Coolify

```bash
curl -fsSL https://cdn.coollabs.io/coolify/upgrade.sh | sudo bash
```

## Próximo paso

[Primer deploy](./02-aplicacoes.md) — Haga el deploy de una aplicación de ejemplo.
