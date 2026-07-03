# 05-04 - ngrok

## Visao geral

[ngrok](https://ngrok.com) cria tunels seguros para expor servicos locais publicamente. E extremamente simples de configurar, ideal para testes e demos.

```
Internet -> vm.meuservidor.com -> Cloudflare DNS -> ngrok Edge
                                                         |
                    VM (ngrok agent) <--- tunel outbound -+
```

## 1. Criar conta no ngrok

1. Acesse https://dashboard.ngrok.com/signup
2. Crie uma conta gratuita
3. Va em **Your Authtoken** e copie o token

## 2. Instalar ngrok na VM

Acesse a VM via SSH:

```bash
ssh ubuntu@192.168.1.100
```

```bash
# Baixar
wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz

# Extrair
tar xzf ngrok-v3-stable-linux-amd64.tgz

# Mover para /usr/local/bin
sudo mv ngrok /usr/local/bin/ngrok

# Verificar
ngrok version
```

## 3. Autenticar

```bash
ngrok config add-authtoken SEU_AUTH_TOKEN
```

## 4. Expor o Coolify

```bash
ngrok http 8000
```

Saida esperada:
```
Forwarding  https://abc123.ngrok-free.app -> http://localhost:8000
```

Acesse `https://abc123.ngrok-free.app` para ver o Coolify.

## 5. Usar dominio proprio com ngrok

### Opcao gratuita: URL fixa

No plano gratuito, voce pode reservar um subdominio `*.ngrok-free.app`:

1. No dashboard do ngrok, va em **Domains**
2. Crie um dominio (ex: `meuvm.ngrok-free.app`)
3. Execute:

```bash
ngrok http 8000 --domain=meuvm.ngrok-free.app
```

### Opcao paga: Dominio customizado

Nos planos pagos, voce pode adicionar seu proprio dominio:

1. No dashboard, va em **Domains > Add a domain**
2. Adicione `vm.meuservidor.com`
3. No Cloudflare DNS, crie um registro CNAME:

| Tipo | Nome | Conteudo |
|------|------|----------|
| CNAME | `vm` | `meuvm.ngrok.app` |

Com proxy **DNS Only** (laranja desligada).

4. Execute:

```bash
ngrok http 8000 --domain=vm.meuservidor.com
```

## 6. Executar ngrok em background (servico)

### Usando nohup

```bash
nohup ngrok http 8000 --domain=meuvm.ngrok-free.app > ~/ngrok.log 2>&1 &
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

## 7. Dashboard do ngrok

Acesse `http://localhost:4040` na VM (ou via tunel SSH) para ver o dashboard local com requisicoes em tempo real.

## Vantagens do ngrok

- Extremamente simples (1 comando)
- Dashboard web com inspecao de requisicoes
- Webhook testing (replay de requisicoes)
- Autenticacao integrada (Basic Auth, OAuth)

## Limitacoes do plano gratuito

- **4 tunnets por minuto** (rate limit)
- **URL aleatoria** muda a cada inicio (a menos que reserve)
- **Aviso de "ngrok-free.app"** na tela
- **40 MB/minuto** de trafego
- **3 tunnets simultaneos** no maximo
- Conexoes HTTP apenas (HTTPS na borda do ngrok)

## Proximo passo

[Coolify - Instalacao](../06-coolify/README.md)
