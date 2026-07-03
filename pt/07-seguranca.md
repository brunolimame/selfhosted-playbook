# 07 - Seguranca

## 1. Firewall (UFW)

Habilite o firewall na VM e libere apenas as portas necessarias:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH (essencial)
sudo ufw allow 22/tcp comment 'SSH'

# Se usar DDNS + Port Forwarding (metodo alternativo)
# sudo ufw allow 80/tcp comment 'HTTP'
# sudo ufw allow 443/tcp comment 'HTTPS'
# sudo ufw allow 8000/tcp comment 'Coolify'

sudo ufw enable
sudo ufw status verbose
```

Com Cloudflare Tunnel, nenhuma porta HTTP precisa ser aberta. Apenas SSH para administracao.

## 2. SSH com chave (desabilitar senha)

### Gerar chave no seu host (se ainda nao tiver)

**Windows (PowerShell)**:
```powershell
ssh-keygen -t ed25519 -C "sua-chave"
```

**Linux/macOS**:
```bash
ssh-keygen -t ed25519 -C "sua-chave"
```

### Copiar chave para a VM

```bash
ssh-copy-id ubuntu@192.168.1.100
```

Ou manualmente:
```bash
cat ~/.ssh/id_ed25519.pub | ssh ubuntu@192.168.1.100 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### Desabilitar login por senha

```bash
sudo nano /etc/ssh/sshd_config
```

Altere:
```
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
```

```bash
sudo systemctl restart sshd
```

## 3. Atualizacoes automaticas

```bash
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

Selecione **Yes** para atualizar automaticamente.

## 4. Fail2ban (protecao contra brute force)

```bash
sudo apt install -y fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

## 5. Coolify - Configuracoes de seguranca

No painel do Coolify:
1. **Settings > Registration**: Desabilite registro publico
2. **Settings > Instance**: Force HTTPS se estiver atras de proxy
3. **Settings > Token**: Gere um token de API com permissoes minimas
4. **Applications**: Evite expor portas de gerenciamento (ex: 8000) diretamente

## 6. Cloudflare - WAF (Web Application Firewall)

No painel do Cloudflare:
1. **Security > WAF**: 

**Bloquear acesso direto por IP**:
Regra customizada:
- Campo: `IP Source Address`
- Operador: `equals`
- Valor: `192.168.1.100`
- Acao: `Block`

**Rate Limiting** (gratuito: 10 regras):
- Protege contra ataques de forca bruta
- Limite: 100 requisicoes por minuto por IP

## 7. Cloudflare Tunnel - Acesso restrito

No arquivo `config.yml` do cloudflared, voce pode adicionar autenticacao:

```yaml
ingress:
  - hostname: admin.vm.meuservidor.com
    service: http://localhost:8000
    originRequest:
      accessToken: "seu-token-aqui"
  - service: http_status:404
```

Ou use o **Cloudflare Access** para proteger rotas com autenticacao SSO:
1. No Cloudflare, va em **Access > Applications**
2. Crie uma aplicacao para `vm.meuservidor.com`
3. Configure politicas de acesso (email, SSO, etc.)

## 8. Backups criptografados

```bash
# Backup do diretorio do Coolify
sudo tar czf ~/backup-coolify-$(date +%Y%m%d).tar.gz /var/lib/docker/volumes/coolify_* --transform 's,.*/,,'
```

Criptografe com GPG:
```bash
gpg --symmetric --cipher-algo AES256 backup-coolify-*.tar.gz
```

## 9. Checklist de seguranca

- [ ] Firewall UFW ativo (apenas SSH liberado)
- [ ] Login SSH apenas por chave
- [ ] Root login desabilitado
- [ ] Unattended upgrades ativo
- [ ] Fail2ban instalado e rodando
- [ ] Coolify com registro publico desabilitado
- [ ] Cloudflare SSL/TLS em Full (strict)
- [ ] Cloudflare WAF configurado
- [ ] Backups automatizados
- [ ] Docker acessivel apenas via socket (`/var/run/docker.sock`)

## Proximo passo

[Manutencao](../08-manutencao.md) - Backup, atualizacoes e monitoramento.
