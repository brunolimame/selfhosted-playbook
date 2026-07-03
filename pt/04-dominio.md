# 04 - Dominio e DNS

## 1. Escolher e registrar um dominio

Voce precisa de um dominio proprio. Exemplos:

| Tipo | Exemplo | Onde registrar |
|------|---------|----------------|
| .com | `meuservidor.com` | Namecheap, GoDaddy, Cloudflare Registrar |
| .com.br | `meuservidor.com.br` | Registro.br |
| .dev | `meuservidor.dev` | Namecheap, Cloudflare Registrar |

> **Dica**: Registrar pelo Cloudflare Registrar custa precos de custo (sem margem de lucro). Se ja tiver um dominio em outro registrar, voce pode move-lo para o Cloudflare.

## 2. Criar conta no Cloudflare

1. Acesse https://dash.cloudflare.com/sign-up
2. Crie uma conta gratuita
3. Confirme o email

## 3. Adicionar o dominio ao Cloudflare

1. Apos o login, clique em **Add a Site**
2. Digite seu dominio (ex: `meuservidor.com`)
3. Selecione o plano **Free** (gratuito)
4. O Cloudflare escaneia os registros DNS existentes (pode demorar alguns segundos)
5. Avance

## 4. Alterar nameservers

O Cloudflare exibira dois nameservers como:
```
dns1.ns.cloudflare.com
dns2.ns.cloudflare.com
```

1. Acesse o painel do seu registrador de dominio
2. Localize a opcao de alterar **Nameservers** (DNS Server)
3. Substitua os nameservers atuais pelos do Cloudflare
4. Salve

A propagacao pode levar de alguns minutos a 48 horas (geralmente < 1 hora).

## 5. Verificar status

No painel do Cloudflare, o status mudara de **Pending** para **Active** quando a alteracao for propagada.

## 6. Criar registros DNS (provisorios)

Enquanto configura o tunel, crie um registro DNS temporario para teste:

No painel do Cloudflare, va em **DNS > Records** e adicione:

**Registro A** (para teste local):
| Tipo | Nome | Conteudo | Proxy |
|------|------|----------|-------|
| A | `vm` | `192.168.1.100` | DNS only (desligado) |

Isso cria `vm.meuservidor.com` apontando para o IP local da VM.

> **Nota**: Este registro funcionara apenas na rede local. A configuracao final usando Cloudflare Tunnel (ou outro metodo) substituira este registro.

## 7. Configurar SSL/TLS

No painel do Cloudflare:
1. Va em **SSL/TLS > Overview**
2. Selecione **Full (strict)** para maior seguranca
3. Em **Origin Server > Create Certificate**, gere um certificado auto-assinado (opcional, mas recomendado)

## 8. Aguardar DNS propagation

Verifique se o DNS ja propagou:

```bash
# No seu host (fora da VM)
nslookup vm.meuservidor.com
# ou
ping vm.meuservidor.com
```

## Proximo passo

[Exposicao Publica](./05-exposicao-publica/README.md) - Escolha como expor sua VM na internet.
