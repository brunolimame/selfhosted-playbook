# Development Tools

Collection of free (and some paid) tools and services that help with development, testing and debugging of webhooks, APIs and integrations.

## Webhook / HTTP Request Inspectors

Services that receive HTTP requests (POST, GET, etc.) and display the content in real time. Essential for debugging webhooks from evolution-go, n8n, Typebot, Telegram, Meta.

| Tool | URL | Free? | Description |
|------|-----|-------|-------------|
| **Webhook.site** | https://webhook.site | Free | Creates instant unique URL. Displays headers, body, query params in real time. Allows custom responses. Best in class. |
| **Pipedream** | https://pipedream.com | Free (500 req/month) | Inspector + workflows. Creates HTTP event source and see each request in the dashboard. |
| **Beeceptor** | https://beeceptor.com | Free (50 req/day) | Creates unique endpoint, displays requests, allows mocking responses. |
| **RequestBin** | https://requestbin.com | Free | Creates bin to collect requests. Free version has a 20 req/bin limit. |
| **Hookbin** | https://hookbin.com | Free | Similar to RequestBin. Creates endpoint and collects requests in real time. |
| **ngrok** | https://ngrok.com | Free (limited) | Exposes localhost to the internet + web inspector at http://localhost:4040 to see all requests in real time. |

### Practical use with this project

To test if evolution-go is sending webhooks correctly:

```bash
# 1. Create a URL on Webhook.site
# 2. Configure in evolution-go:
curl -X POST http://192.168.1.100:4000/webhook/create/my-whatsapp \
  -H "apiKey: YOUR_API_KEY" \
  -d '{
    "webhook": { "url": "https://webhook.site/YOUR-UUID" }
  }'

# 3. Send a message on the connected WhatsApp
# 4. See the request arrive in real time on Webhook.site
```

## API Clients

| Tool | URL | Free? | Description |
|------|-----|-------|-------------|
| **Hoppscotch** | https://hoppscotch.io | Free (open-source) | Alternative to Postman. Web + self-host. Supports REST, GraphQL, WebSocket, SSE. |
| **Insomnia** | https://insomnia.rest | Free | Desktop API client with plugin support and documentation generation. |
| **Bruno** | https://www.usebruno.com | Free (open-source) | Offline-first API client. Requests saved in files. |
| **Postman** | https://www.postman.com | Free (basic) / Paid (team) | Most well-known. Free version sufficient for individual use. |
| **HTTPie** | https://httpie.io | Free (open-source) | Terminal API client. `http POST url field=value`. |

### Use with evolution-go

```bash
# Example with HTTPie (terminal)
http POST http://192.168.1.100:4000/message/sendText \
  apiKey:YOUR_API_KEY \
  number="5511999999999" \
  textMessage:='{"text": "Hello from terminal!"}'
```

## Local Webhook Testing (expose localhost)

When developing locally and needing to receive webhooks from external services (Telegram, Meta, evolution-go):

| Tool | URL | Free? | Description |
|------|-----|-------|-------------|
| **ngrok** | https://ngrok.com | Free (40 req/min, 4 tunnels) | Creates public URL `https://abc.ngrok-free.app` pointing to `localhost:PORT`. Includes web inspector. |
| **Bore** | https://github.com/ekzhang/bore | Free (open-source) | Simple CLI tunnel. Needs a public server or uses the public `bore.pub`. |
| **Localtunnel** | https://localtunnel.github.io/www/ | Free (open-source) | `npx localtunnel --port 3000` creates public URL. |
| **Cloudflare Tunnel** | https://developers.cloudflare.com/cloudflare-one/connections/connect | Free | Already documented in this project. Can be used for development pointing to localhost. |
| **Tailscale Funnel** | https://tailscale.com/kb/1223/funnel/ | Free (3 users) | Already documented in this project. Expose local services via `*.ts.net`. |

## Mock / Test Data Generation

| Tool | URL | Free? | Description |
|------|-----|-------|-------------|
| **JSONPlaceholder** | https://jsonplaceholder.typicode.com | Free | Fake REST API for testing. Posts, comments, users, etc. |
| **httpbin.org** | https://httpbin.org | Free | Endpoints to test HTTP requests. Returns headers, IP, method, etc. |
| **Faker API** | https://fakerapi.it | Free | Generates realistic fake data (name, email, address, phone). |
| **MockAPI** | https://mockapi.io | Free (basic) | Creates mocked REST APIs with custom data. |
| **Beeceptor Mocks** | https://beeceptor.com | Free | Mock responses on endpoints. |
| **ReqRes** | https://reqres.in | Free | Mocked REST API for integration testing. |

## Formatters and Viewers

| Tool | URL | Description |
|------|-----|-------------|
| **JSON Formatter** | https://jsonformatter.org | Format and validate JSON |
| **Code Beautify** | https://codebeautify.org | Formatters for JSON, XML, HTML, CSS |
| **JWT.io** | https://jwt.io | Decode and verify JWT tokens |
| **Base64 Decode** | https://www.base64decode.org | Decode/encode Base64 |
| **Regex101** | https://regex101.com | Test regular expressions |
| **CronTab Guru** | https://crontab.guru | Explain cron expressions |
| **CyberChef** | https://gchq.github.io/CyberChef/ | "Swiss army knife" for data formatting (ciphers, encoding, etc.) |

## Flow / Diagram Editors

| Tool | URL | Free? | Description |
|------|-----|-------|-------------|
| **Draw.io** | https://app.diagrams.net | Free | Flow diagrams, architecture, UML. |
| **Excalidraw** | https://excalidraw.com | Free | Collaborative whiteboard for sketching architectures. |
| **Mermaid Live** | https://mermaid.live | Free | Text-based diagrams (Markdown). Code becomes graphic. |
| **ASCIIFlow** | https://asciiflow.com | Free | ASCII diagrams (like those in this documentation). |

## Free CI / CD

| Tool | URL | Free? | Description |
|------|-----|-------|-------------|
| **GitHub Actions** | https://github.com/features/actions | Free (2000 min/month) | Native GitHub CI/CD. |
| **GitLab CI** | https://docs.gitlab.com/ee/ci/ | Free (400 min/month) | Native GitLab CI/CD. |
| **Render** | https://render.com | Free (limited) | App deployment with included CI. |
| **Railway** | https://railway.app | Free (limited) | Deploy with integrated CI. |
| **Cloudflare Pages** | https://pages.cloudflare.com | Free | Static site deployment with CI. |

## Summary: Recommended development stack

```
Webhook Testing:    Webhook.site (free, no login)
API Client:         Hoppscotch (free, web)
Local Tunnel:       ngrok (free with limits)
JSON Tools:         JSON Formatter + JWT.io
Diagrams:           Draw.io + Mermaid
Monitor:            Uptime Kuma + Netdata
```

## How to add more tools

See the template in [ADD_APPLICATION.md](./ADD_APPLICATION.md) to add new tools with full documentation.
