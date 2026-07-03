# 06-01 - Instalacao do Coolify

## 1. Instalar Docker na VM

Acesse a VM via SSH:

```bash
ssh ubuntu@192.168.1.100
```

### Metodo oficial (recomendado)

```bash
curl -fsSL https://get.docker.com | sh
```

### Ou manualmente

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

### Adicionar usuario ao grupo docker

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

O script:
1. Baixa a ultima versao do Coolify
2. Cria os containers necessarios (coolify, postgres, redis)
3. Configura o servico systemd
4. Exibe a senha de administrador gerada automaticamente

Saida esperada:
```
Coolify is now running on http://localhost:8000
Email: admin@example.com
Password: [senha-gerada-automaticamente]
```

**Importante**: Anote o email e senha exibidos.

### Acompanhar a instalacao

```bash
sudo journalctl -u coolify -f
```

## 3. Acessar o Coolify

### Pela rede local

No navegador do seu host, acesse:

```
http://192.168.1.100:8000
```

### Pelo dominio (se o tunel ja estiver configurado)

```
https://vm.meuservidor.com
```

## 4. Configuracao inicial

1. Faca login com o email e senha exibidos na instalacao
2. Va em **Settings > Instance Name** e defina um nome (ex: `Meu Servidor`)
3. Va em **Settings > Timezone** e selecione seu fuso (ex: `America/Sao_Paulo`)
4. Va em **Settings > Wildcard Domain** e configure `*.vm.meuservidor.com` (se tiver o dominio)
5. Va em **Settings > Registration** e desabilite registro publico (seguranca)

## 5. Verificar status

```bash
docker ps
```

Devera ver containers como:
- `coolify` (aplicacao principal)
- `coolify-db` (PostgreSQL)
- `coolify-redis` (cache/queue)

## 6. Atualizar Coolify

```bash
curl -fsSL https://cdn.coollabs.io/coolify/upgrade.sh | sudo bash
```

## Proximo passo

[Primeiro deploy](./02-aplicacoes.md) - Faca o deploy de uma aplicacao exemplo.
