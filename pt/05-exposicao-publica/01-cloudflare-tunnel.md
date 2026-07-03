# 05-01 - Cloudflare Tunnel

## Visao geral

O Cloudflare Tunnel cria um tunel seguro e criptografado entre sua VM e a rede edge do Cloudflare. O `cloudflared` (agente) roda na VM e estabelece conexoes outbound para o Cloudflare. Nenhuma porta precisa ser aberta no roteador.

```
Internet -> vm.meuservidor.com -> Cloudflare DNS -> Cloudflare Edge
                                                         |
                    VM (cloudflared) <--- tunel outbound -+
```

## 1. Instalar o cloudflared na VM

Acesse a VM via SSH:

```bash
ssh ubuntu@192.168.1.100
```

### Download e instalacao

```bash
# Baixar o cloudflared
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb

# Instalar
sudo dpkg -i cloudflared-linux-amd64.deb

# Verificar
cloudflared version
```

## 2. Autenticar com o Cloudflare

```bash
cloudflared tunnel login
```

Isso abrira uma URL no terminal. Como e uma VM sem navegador, copie a URL e abra no navegador do seu host.

1. Copie a URL exibida no terminal
2. Cole no navegador do seu computador host
3. Faca login no Cloudflare (se necessario)
4. Selecione o dominio que voce adicionou ao Cloudflare
5. Clique em **Authorize**

Apos autorizar, o arquivo de certificado sera baixado automaticamente para `~/.cloudflared/cert.pem`.

## 3. Criar o tunel

```bash
cloudflared tunnel create meu-tunel
```

Isso cria um tunel com um ID unico (ex: `abcdef01-1234-5678-9abc-def012345678`). Anote esse ID.

Saida esperada:
```
Created tunnel meu-tunel with id abcdef01-1234-5678-9abc-def012345678
```

## 4. Configurar o tunel

Crie o arquivo de configuracao:

```bash
nano ~/.cloudflared/config.yml
```

Conteudo:

```yaml
tunnel: meu-tunel
credentials-file: /home/ubuntu/.cloudflared/abcdef01-1234-5678-9abc-def012345678.json

ingress:
  - hostname: vm.meuservidor.com
    service: http://localhost:8000
  - service: http_status:404
```

Explicacao:
- `tunnel`: nome do tunel criado
- `credentials-file`: caminho para o arquivo de credenciais do tunel (ajuste o nome do arquivo)
- `ingress`: mapeia o hostname `vm.meuservidor.com` para o Coolify rodando na porta `8000` da VM
- A ultima linha e uma regra catch-all que retorna 404

## 5. Criar registro DNS

Agora aponte o DNS do Cloudflare para o tunel:

```bash
cloudflared tunnel route dns meu-tunel vm.meuservidor.com
```

Isso cria um registro CNAME no Cloudflare apontando `vm.meuservidor.com` para o ID do tunel.

## 6. Executar o tunel como servico

### Testar primeiro

```bash
cloudflared tunnel run meu-tunel
```

Se funcionar, pare com `Ctrl+C` e configure como servico.

### Instalar como servico systemd

```bash
sudo cloudflared service install
```

Ou manualmente:

```bash
sudo nano /etc/systemd/system/cloudflared.service
```

Conteudo:

```ini
[Unit]
Description=Cloudflare Tunnel
After=network.target

[Service]
Type=simple
User=ubuntu
ExecStart=/usr/bin/cloudflared tunnel run meu-tunel
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Ativar e iniciar:

```bash
sudo systemctl daemon-reload
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
```

Verificar status:

```bash
sudo systemctl status cloudflared
```

## 7. Ajustar SSL/TLS no Cloudflare

No painel do Cloudflare:
1. Va em **SSL/TLS > Overview**
2. Selecione **Full (strict)**
3. Va em **Edge Certificates** e ative **Always Use HTTPS**

## 8. Testar o acesso

No navegador, acesse:

```
https://vm.meuservidor.com
```

Se o Coolify ainda nao estiver instalado, voce vera um erro de conexao recusada. Isso e normal - prossiga para a instalacao do Coolify.

## Manutencao do tunel

### Ver logs
```bash
sudo journalctl -u cloudflared -f
```

### Atualizar o cloudflared
```bash
sudo dpkg -i cloudflared-linux-amd64.deb
sudo systemctl restart cloudflared
```

### Remover o tunel (se necessario)
```bash
cloudflared tunnel delete meu-tunel
```

## Limitacoes do plano Free do Cloudflare
- **Sem streaming de video** (viola os termos)
- **Maximo 100 MB por request** para download via proxy
- **3 regras de Page Rules** gratuitas

## Proximo passo

[Coolify - Instalacao](../06-coolify/README.md)
