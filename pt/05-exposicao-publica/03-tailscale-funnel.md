# 05-03 - Tailscale Funnel

## Visao geral

[Tailscale](https://tailscale.com) cria uma rede privada (WireGuard) entre seus dispositivos. O **Funnel** e um recurso que permite expor servicos da sua rede Tailscale publicamente na internet, usando um subdominio `*.ts.net`.

```
Internet -> vm.meuservidor.com -> Cloudflare DNS
                                        |
                            Tailscale Funnel (443)
                                        |
                               VM (Tailscale)
                                        |
                               Coolify (:8000)
```

**Diferenca do Cloudflare Tunnel**: Com Tailscale Funnel, os usuarios precisam acessar via um subdominio `*.ts.net` (a menos que voce configure seu dominio para apontar para la).

## 1. Instalar Tailscale na VM

Acesse a VM via SSH:

```bash
ssh ubuntu@192.168.1.100
```

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

## 2. Autenticar

```bash
sudo tailscale up
```

Isso exibira uma URL. Copie e abra no navegador, faca login com sua conta Google/Microsoft/GitHub/Apple.

Apos autenticar, verifique o IP Tailscale da VM:

```bash
tailscale ip -4
```

Exemplo: `100.x.x.x`

## 3. Instalar Tailscale no seu host (opcional)

Para acessar a VM pela rede Tailscale (sem depender do IP local):

- **Windows**: Baixe de https://tailscale.com/download
- **Linux**: `curl -fsSL https://tailscale.com/install.sh | sh`
- **macOS**: Baixe da App Store ou site

Entre na mesma conta e execute `tailscale up`.

Agora voce pode acessar a VM via IP Tailscale:
```bash
ssh ubuntu@100.x.x.x
```

## 4. Habilitar Funnel

O Funnel expoe um servico local publicamente via `https://<nome-da-maquina>.<seudominio>.ts.net`.

```bash
sudo tailscale funnel --bg 8000
```

Isso expoe a porta 8000 (Coolify) publicamente em:
```
https://ubuntu-vm.ts.net
```

Para usar um nome personalizado (antes de `ts.net`):
```bash
sudo tailscale funnel --bg 8000 --set-path=/
```

## 5. Usar dominio proprio com Tailscale Funnel

Para usar `vm.meuservidor.com` com Tailscale Funnel, voce precisa de um proxy reverso adicional ou usar o **Tailscale Serve** com HTTPS customizado.

### Opcao: CNAME do Cloudflare para Tailscale

No Cloudflare DNS, crie um registro CNAME:

| Tipo | Nome | Conteudo |
|------|------|----------|
| CNAME | `vm` | `ubuntu-vm.ts.net` |

Com proxy **DNS Only** (laranja desligada).

Isso faz `vm.meuservidor.com` apontar para `ubuntu-vm.ts.net`.

> **Nota**: O Cloudflare Tunnel (metodo recomendado) e mais flexivel para dominios proprios.

## 6. Gerenciar Funnel

```bash
# Ver status
tailscale funnel status

# Parar o funnel
tailscale funnel off
```

## Vantagens do Tailscale Funnel

- Configuracao muito simples (2 comandos)
- Criptografia ponta-a-ponta (WireGuard + HTTPS)
- Nao requer abrir portas
- Acesso a rede privada entre dispositivos
- Gratuito ate 3 usuarios

## Limitacoes

- URL publica e `*.ts.net` (ou precisa configurar CNAME)
- Depende da infraestrutura Tailscale
- Limite de 3 usuarios no plano gratuito
- Tailscale e uma camada adicional entre o usuario e o servico

## Proximo passo

[Coolify - Instalacao](../06-coolify/README.md)
