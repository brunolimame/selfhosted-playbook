# Herramientas de Desarrollo

Coleccion de herramientas y servicios gratuitos (y algunos pagos) que ayudan en el desarrollo, prueba y depuracion de webhooks, APIs e integraciones.

## Inspectores de Webhook / HTTP Request

Servicios que reciben peticiones HTTP (POST, GET, etc.) y muestran el contenido en tiempo real. Esenciales para depurar webhooks de evolution-go, n8n, Typebot, Telegram, Meta.

| Herramienta | URL | Gratuito? | Descripcion |
|-------------|-----|-----------|-------------|
| **Webhook.site** | https://webhook.site | Gratuito | Crea URL unica instantanea. Muestra headers, body, query params en tiempo real. Permite respuestas personalizadas. Mejor de la categoria. |
| **Pipedream** | https://pipedream.com | Gratuito (500 req/mes) | Inspector + workflows. Crea fuente de eventos HTTP y ve cada peticion en el dashboard. |
| **Beeceptor** | https://beeceptor.com | Gratuito (50 req/dia) | Crea endpoint unico, muestra peticiones, permite mockear respuestas. |
| **RequestBin** | https://requestbin.com | Gratuito | Crea bin para recolectar peticiones. Version gratuita tiene limite de 20 req/bin. |
| **Hookbin** | https://hookbin.com | Gratuito | Similar a RequestBin. Crea endpoint y recolecta peticiones en tiempo real. |
| **ngrok** | https://ngrok.com | Gratuito (limitado) | Expone localhost a internet + inspector web en http://localhost:4040 para ver todas las peticiones en tiempo real. |

### Uso practico con este proyecto

Para probar si evolution-go esta enviando webhooks correctamente:

```bash
# 1. Cree una URL en Webhook.site
# 2. Configure en evolution-go:
curl -X POST http://192.168.1.100:4000/webhook/create/mi-whatsapp \
  -H "apiKey: SU_API_KEY" \
  -d '{
    "webhook": { "url": "https://webhook.site/SU-UUID" }
  }'

# 3. Envie un mensaje en el WhatsApp conectado
# 4. Vea la peticion llegar en tiempo real en Webhook.site
```

## Clientes API

| Herramienta | URL | Gratuito? | Descripcion |
|-------------|-----|-----------|-------------|
| **Hoppscotch** | https://hoppscotch.io | Gratuito (open-source) | Alternativa a Postman. Web + self-host. Soporta REST, GraphQL, WebSocket, SSE. |
| **Insomnia** | https://insomnia.rest | Gratuito | Cliente API de escritorio con soporte a plugins y generacion de documentacion. |
| **Bruno** | https://www.usebruno.com | Gratuito (open-source) | Cliente API offline-first. Peticiones guardadas en archivos. |
| **Postman** | https://www.postman.com | Gratuito (basico) / Pago (equipo) | Mas conocido. Version gratuita suficiente para uso individual. |
| **HTTPie** | https://httpie.io | Gratuito (open-source) | Cliente API via terminal. `http POST url campo=valor`. |

### Uso con evolution-go

```bash
# Ejemplo con HTTPie (terminal)
http POST http://192.168.1.100:4000/message/sendText \
  apiKey:SU_API_KEY \
  number="5511999999999" \
  textMessage:='{"text": "Hola desde la terminal!"}'
```

## Prueba de Webhook Locales (exponer localhost)

Cuando este desarrollando localmente y necesite recibir webhooks de servicios externos (Telegram, Meta, evolution-go):

| Herramienta | URL | Gratuito? | Descripcion |
|-------------|-----|-----------|-------------|
| **ngrok** | https://ngrok.com | Gratuito (40 req/min, 4 tunnets) | Crea URL publica `https://abc.ngrok-free.app` apuntando a `localhost:PUERTO`. Incluye inspector web. |
| **Bore** | https://github.com/ekzhang/bore | Gratuito (open-source) | Tunnel simple via CLI. Necesita servidor publico o usa el publico `bore.pub`. |
| **Localtunnel** | https://localtunnel.github.io/www/ | Gratuito (open-source) | `npx localtunnel --port 3000` crea URL publica. |
| **Cloudflare Tunnel** | https://developers.cloudflare.com/cloudflare-one/connections/connect | Gratuito | Ya documentado en este proyecto. Puede usarse para desarrollo apuntando a localhost. |
| **Tailscale Funnel** | https://tailscale.com/kb/1223/funnel/ | Gratuito (3 users) | Ya documentado en este proyecto. Expone servicios local via `*.ts.net`. |

## Generacion de Datos Mock / Prueba

| Herramienta | URL | Gratuito? | Descripcion |
|-------------|-----|-----------|-------------|
| **JSONPlaceholder** | https://jsonplaceholder.typicode.com | Gratuito | API REST fake para pruebas. Posts, comments, users, etc. |
| **httpbin.org** | https://httpbin.org | Gratuito | Endpoints para probar peticiones HTTP. Retorna headers, IP, metodo, etc. |
| **Faker API** | https://fakerapi.it | Gratuito | Genera datos falsos realistas (nombre, email, direccion, CPF). |
| **MockAPI** | https://mockapi.io | Gratuito (basico) | Crea APIs REST mockeadas con datos personalizados. |
| **Beeceptor Mocks** | https://beeceptor.com | Gratuito | Mockear respuestas en endpoints. |
| **ReqRes** | https://reqres.in | Gratuito | API REST mockeada para prueba de integracion. |

## Formateadores y Visualizadores

| Herramienta | URL | Descripcion |
|-------------|-----|-------------|
| **JSON Formatter** | https://jsonformatter.org | Formatear y validar JSON |
| **Code Beautify** | https://codebeautify.org | Formateadores para JSON, XML, HTML, CSS |
| **JWT.io** | https://jwt.io | Decodificar y verificar tokens JWT |
| **Base64 Decode** | https://www.base64decode.org | Decodificar/codificar Base64 |
| **Regex101** | https://regex101.com | Probar expresiones regulares |
| **CronTab Guru** | https://crontab.guru | Explicar expresiones cron |
| **CyberChef** | https://gchq.github.io/CyberChef/ | "Navaja suiza" de formateo de datos (cifrados, encoding, etc.) |

## Editores de Flujo / Diagramas

| Herramienta | URL | Gratuito? | Descripcion |
|-------------|-----|-----------|-------------|
| **Draw.io** | https://app.diagrams.net | Gratuito | Diagramas de flujo, arquitectura, UML. |
| **Excalidraw** | https://excalidraw.com | Gratuito | Pizarra colaborativa para dibujar arquitecturas. |
| **Mermaid Live** | https://mermaid.live | Gratuito | Diagramas en texto (Markdown). Codigo se convierte en grafico. |
| **ASCIIFlow** | https://asciiflow.com | Gratuito | Diagramas en ASCII (como los de esta documentacion). |

## CI / CD Gratuitos

| Herramienta | URL | Gratuito? | Descripcion |
|-------------|-----|-----------|-------------|
| **GitHub Actions** | https://github.com/features/actions | Gratuito (2000 min/mes) | CI/CD nativo de GitHub. |
| **GitLab CI** | https://docs.gitlab.com/ee/ci/ | Gratuito (400 min/mes) | CI/CD nativo de GitLab. |
| **Render** | https://render.com | Gratuito (limitado) | Deploy de apps con CI incluido. |
| **Railway** | https://railway.app | Gratuito (limitado) | Deploy con CI integrado. |
| **Cloudflare Pages** | https://pages.cloudflare.com | Gratuito | Deploy de sitios estaticos con CI. |

## Resumen: Stack recomendado para desarrollo

```
Webhook Testing:    Webhook.site (gratuito, sin login)
API Client:         Hoppscotch (gratuito, web)
Local Tunnel:       ngrok (gratuito con limites)
JSON Tools:         JSON Formatter + JWT.io
Diagramas:          Draw.io + Mermaid
Monitor:            Uptime Kuma + Netdata
```

## Como anadir mas herramientas

Vea el template en [ADICIONAR_APLICACION.md](./ADICIONAR_APLICACION.md) para anadir nuevas herramientas con documentacion completa.
