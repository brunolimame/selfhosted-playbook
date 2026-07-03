# 05-02 - DDNS + Port Forwarding

## Visao geral

Este metodo tradicional expoe a VM diretamente na internet usando:
- **DDNS (Dynamic DNS)**: Um servico que atualiza automaticamente um registro DNS quando seu IP publico muda
- **Port Forwarding**: Redirecionamento de portas do roteador para a VM

```
Internet -> vm.meuservidor.com -> Roteador (IP dinamico)
                                       |
                                    Port Forwarding
                                       |
                               VM (192.168.1.100)
```

## Aviso de seguranca

Este metodo **abre portas no roteador**, expondo sua rede diretamente. Apenas use se:
- Voce entende os riscos de seguranca
- Tem configuracao de firewall adequada
- Mantem o sistema sempre atualizado

## 1. Configurar DDNS (DNS Dinamico)

### Opcao 1: DuckDNS (gratuito, simples)

1. Acesse https://www.duckdns.org
2. Faca login com uma conta Google, GitHub ou Twitter
3. Crie um subdominio (ex: `meuvm.duckdns.org`)
4. Anote o **token** gerado

### Opcao 2: No-IP (gratuito, precisa renovar mensalmente)

1. Acesse https://www.noip.com
2. Crie uma conta gratuita
3. Crie um hostname (ex: `meuvm.hopto.org`)
4. Instale o cliente No-IP na VM

### Instalar cliente DDNS na VM

**Para DuckDNS**:

Crie um script de atualizacao:

```bash
nano ~/duckdns.sh
```

```bash
#!/bin/bash
echo url="https://www.duckdns.org/update?domains=meuvm&token=SEU_TOKEN&ip=" | curl -k -o ~/duckdns.log -K -
```

```bash
chmod +x ~/duckdns.sh
```

Adicione ao crontab para executar a cada 5 minutos:

```bash
crontab -e
```

Adicione a linha:
```
*/5 * * * * /home/ubuntu/duckdns.sh
```

Se voce tem um dominio proprio e quer usar DDNS com Cloudflare, use a API do Cloudflare para atualizar o DNS. Ferramentas como `ddclient` ou `cloudflare-ddns` podem ajudar.

## 2. Configurar Port Forwarding no roteador

1. Acesse o painel do roteador (geralmente `192.168.1.1`)
2. Faca login (admin/admin ou usuario/senha no rotulo do roteador)
3. Localize **Port Forwarding** ou **Virtual Server** ou **NAT**
4. Crie as regras:

| Porta externa | IP interno | Porta interna | Protocolo | Descricao |
|--------------|------------|---------------|-----------|-----------|
| 80 | 192.168.1.100 | 80 | TCP | HTTP (redirecionar para HTTPS) |
| 443 | 192.168.1.100 | 443 | TCP | HTTPS (Coolify ou proxy) |
| 8000 | 192.168.1.100 | 8000 | TCP | Coolify Web UI |

5. Salve as configuracoes

## 3. Configurar firewall na VM (UFW)

```bash
sudo ufw allow 22/tcp   # SSH
sudo ufw allow 80/tcp   # HTTP
sudo ufw allow 443/tcp  # HTTPS
sudo ufw allow 8000/tcp # Coolify

sudo ufw enable
sudo ufw status verbose
```

## 4. Proxy reverso (recomendado)

Para servir multiplas aplicacoes na porta 80/443, instale um proxy reverso como Nginx ou Caddy:

```bash
sudo apt install -y nginx
```

Exemplo de configuracao Nginx para Coolify:

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

## 5. Testar

1. Descubra seu IP publico: `curl ifconfig.me`
2. Acesse `http://SEU_IP_PUBLICO:8000` (temporario)
3. Se funcionar, acesse `https://vm.meuservidor.com`

## Limitacoes e riscos

- **IP pode mudar**: Se o DDNS falhar, o servico fica inacessivel
- **Portas abertas**: Qualquer vulnerabilidade na VM expoe sua rede interna
- **ISP pode bloquear**: Alguns provedores bloqueiam portas 80/443 em planos residenciais
- **Sem protecao DDoS**: Ataques vao direto para sua conexao

## Proximo passo

[Coolify - Instalacao](../06-coolify/README.md)
