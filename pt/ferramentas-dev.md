# Ferramentas de Desenvolvimento

Colecao de ferramentas e servicos gratuitos (e alguns pagos) que ajudam no desenvolvimento, teste e depuracao de webhooks, APIs e integracoes.

## Inspetores de Webhook / HTTP Request

Servicos que recebem requisicoes HTTP (POST, GET, etc.) e exibem o conteudo em tempo real. Essenciais para depurar webhooks do evolution-go, n8n, Typebot, Telegram, Meta.

| Ferramenta | URL | Gratuito? | Descricao |
|-----------|-----|-----------|-----------|
| **Webhook.site** | https://webhook.site | Gratuito | Cria URL unica instantanea. Exibe headers, body, query params em tempo real. Permite respostas customizadas. Melhor da categoria. |
| **Pipedream** | https://pipedream.com | Gratuito (500 req/mes) | Inspetor + workflows. Cria fonte de eventos HTTP e ve cada requisicao no dashboard. |
| **Beeceptor** | https://beeceptor.com | Gratuito (50 req/dia) | Cria endpoint unico, exibe requisicoes, permite mockar respostas. |
| **RequestBin** | https://requestbin.com | Gratuito | Cria bin para coletar requisicoes. Versao gratuita tem limite de 20 req/bin. |
| **Hookbin** | https://hookbin.com | Gratuito | Similar ao RequestBin. Cria endpoint e coletar requisicoes em tempo real. |
| **ngrok** | https://ngrok.com | Gratuito (limitado) | Expoe localhost para internet + inspetor web em http://localhost:4040 para ver todas as requisicoes em tempo real. |

### Uso pratico com este projeto

Para testar se o evolution-go esta enviando webhooks corretamente:

```bash
# 1. Crie uma URL no Webhook.site
# 2. Configure no evolution-go:
curl -X POST http://192.168.1.100:4000/webhook/create/meu-whatsapp \
  -H "apiKey: SUA_API_KEY" \
  -d '{
    "webhook": { "url": "https://webhook.site/SEU-UUID" }
  }'

# 3. Envie uma mensagem no WhatsApp conectado
# 4. Veja a requisicao chegar em tempo real no Webhook.site
```

## Clientes API

| Ferramenta | URL | Gratuito? | Descricao |
|-----------|-----|-----------|-----------|
| **Hoppscotch** | https://hoppscotch.io | Gratuito (open-source) | Alternativa ao Postman. Web + self-host. Suporta REST, GraphQL, WebSocket, SSE. |
| **Insomnia** | https://insomnia.rest | Gratuito | Cliente API desktop com suporte a plugins e geracao de documentacao. |
| **Bruno** | https://www.usebruno.com | Gratuito (open-source) | Cliente API offline-first. Requisicoes salvas em arquivos. |
| **Postman** | https://www.postman.com | Gratuito (basico) / Pago (equipe) | Mais conhecido. Versao gratuita suficiente para uso individual. |
| **HTTPie** | https://httpie.io | Gratuito (open-source) | Cliente API via terminal. `http POST url campo=valor`. |

### Uso com evolution-go

```bash
# Exemplo com HTTPie (terminal)
http POST http://192.168.1.100:4000/message/sendText \
  apiKey:SUA_API_KEY \
  number="5511999999999" \
  textMessage:='{"text": "Ola do terminal!"}'
```

## Teste de Webhook Locais (expor localhost)

Quando estiver desenvolvendo localmente e precisar receber webhooks de servicos externos (Telegram, Meta, evolution-go):

| Ferramenta | URL | Gratuito? | Descricao |
|-----------|-----|-----------|-----------|
| **ngrok** | https://ngrok.com | Gratuito (40 req/min, 4 tunnets) | Cria URL publica `https://abc.ngrok-free.app` apontando para `localhost:PORTA`. Inclui inspetor web. |
| **Bore** | https://github.com/ekzhang/bore | Gratuito (open-source) | Tunnel simples via CLI. Precisa de servidor publico ou usa o publico `bore.pub`. |
| **Localtunnel** | https://localtunnel.github.io/www/ | Gratuito (open-source) | `npx localtunnel --port 3000` cria URL publica. |
| **Cloudflare Tunnel** | https://developers.cloudflare.com/cloudflare-one/connections/connect | Gratuito | Ja documentado neste projeto. Pode ser usado para desenvolvimento apontando para localhost. |
| **Tailscale Funnel** | https://tailscale.com/kb/1223/funnel/ | Gratuito (3 users) | Ja documentado neste projeto. Expor servicos local via `*.ts.net`. |

## Geracao de Dados Mock / Teste

| Ferramenta | URL | Gratuito? | Descricao |
|-----------|-----|-----------|-----------|
| **JSONPlaceholder** | https://jsonplaceholder.typicode.com | Gratuito | API REST fake para testes. Posts, comments, users, etc. |
| **httpbin.org** | https://httpbin.org | Gratuito | Endpoints para testar requisicoes HTTP. Retorna headers, IP, metodo, etc. |
| **Faker API** | https://fakerapi.it | Gratuito | Gera dados falsos realistas (nome, email, endereco, CPF). |
| **MockAPI** | https://mockapi.io | Gratuito (basico) | Cria APIs REST mockadas com dados customizados. |
| **Beeceptor Mocks** | https://beeceptor.com | Gratuito | Mockar respostas em endpoints. |
| **ReqRes** | https://reqres.in | Gratuito | API REST mockada para teste de integracao. |

## Formatadores e Visualizadores

| Ferramenta | URL | Descricao |
|-----------|-----|-----------|
| **JSON Formatter** | https://jsonformatter.org | Formatar e validar JSON |
| **Code Beautify** | https://codebeautify.org | Formatadores para JSON, XML, HTML, CSS |
| **JWT.io** | https://jwt.io | Decodificar e verificar tokens JWT |
| **Base64 Decode** | https://www.base64decode.org | Decodificar/encode Base64 |
| **Regex101** | https://regex101.com | Testar expressoes regulares |
| **CronTab Guru** | https://crontab.guru | Explicar expressoes cron |
| **CyberChef** | https://gchq.github.io/CyberChef/ | "Navalha suica" de formatacao de dados (ciphers, encoding, etc.) |

## Editores de Fluxo / Diagramas

| Ferramenta | URL | Gratuito? | Descricao |
|-----------|-----|-----------|-----------|
| **Draw.io** | https://app.diagrams.net | Gratuito | Diagramas de fluxo, arquitetura, UML. |
| **Excalidraw** | https://excalidraw.com | Gratuito | Quadro branco colaborativo para rabiscar arquiteturas. |
| **Mermaid Live** | https://mermaid.live | Gratuito | Diagramas em texto (Markdown). Codigo vira grafico. |
| **ASCIIFlow** | https://asciiflow.com | Gratuito | Diagramas em ASCII (como os desta documentacao). |

## CI / CD Gratuitos

| Ferramenta | URL | Gratuito? | Descricao |
|-----------|-----|-----------|-----------|
| **GitHub Actions** | https://github.com/features/actions | Gratuito (2000 min/mes) | CI/CD nativo do GitHub. |
| **GitLab CI** | https://docs.gitlab.com/ee/ci/ | Gratuito (400 min/mes) | CI/CD nativo do GitLab. |
| **Render** | https://render.com | Gratuito (limitado) | Deploy de apps com CI incluido. |
| **Railway** | https://railway.app | Gratuito (limitado) | Deploy com CI integrado. |
| **Cloudflare Pages** | https://pages.cloudflare.com | Gratuito | Deploy de sites estaticos com CI. |

## Resumo: Stack recomendada para desenvolvimento

```
Webhook Testing:    Webhook.site (gratuito, sem login)
API Client:         Hoppscotch (gratuito, web)
Local Tunnel:       ngrok (gratuito com limites)
JSON Tools:         JSON Formatter + JWT.io
Diagramas:          Draw.io + Mermaid
Monitor:            Uptime Kuma + Netdata
```

## Como adicionar mais ferramentas

Veja o template em [ADICIONAR_APLICACAO.md](./ADICIONAR_APLICACAO.md) para adicionar novas ferramentas com documentacao completa.
