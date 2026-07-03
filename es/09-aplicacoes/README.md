# 09 - Aplicaciones Avanzadas

Guía de instalación detallada para aplicaciones avanzadas que se ejecutan sobre Coolify (o directamente en la VM).

## Índice

| Página | Descripción | Puerto |
|--------|-------------|--------|
| [01 - Odysseus](./01-odysseus.md) | Workspace de IA autoalojado (chat, agentes, email, documentos) | 7000 |
| [02 - n8n](./02-n8n.md) | Automatización de workflows (alternativa a Zapier) | 5678 |
| [03 - Evolution Go](./03-evolution-go.md) | API de WhatsApp en Go (evolution-api) | 4000 |
| [04 - Typebot](./04-typebot.md) | Chatbot visual con flujos no-code (integración nativa evolution) | 3001 / 3002 |
| [05 - Chatwoot](./05-chatwoot.md) | Helpdesk/CRM de atención multiagente | 3000 |
| [06 - Dify](./06-dify.md) | IA con RAG y base de conocimiento (LLM + documentos) | 8080 |
| [07 - MinIO](./07-minio.md) | Almacenamiento S3 para archivos multimedia (compatible con todas las apps) | 9000 / 9001 |
| [08 - Evolution API (Node.js)](./08-evolution-api.md) | API Node.js completa (multiproveedor, panel web, integraciones nativas) | 8080 |
| [09 - Graphify](./09-graphify.md) | Grafo de conocimiento para asistentes de IA (codebase -> query) | 8080 |
| [10 - pgAdmin](./10-pgadmin.md) | Administración web de PostgreSQL | 5050 |
| [11 - Uptime Kuma](./11-uptime-kuma.md) | Monitoreo de uptime con página de estado | 3001 |
| [12 - Netdata](./12-netdata.md) | Monitoreo en tiempo real (CPU, RAM, disco, Docker) | 19999 |

## Forma de instalación

Las aplicaciones se instalarán mediante Docker Compose directamente en la VM (fuera de Coolify por ahora), ya que tienen dependencias complejas (PostgreSQL, Redis, etc.) que Coolify podría no gestionar adecuadamente.

El flujo general es:
1. Acceder a la VM vía SSH
2. Crear un directorio para la aplicación
3. Configurar el `docker-compose.yml` y `.env`
4. Iniciar con `docker compose up -d`
5. Configurar dominio en Cloudflare Tunnel
6. Probar el acceso

## Plantilla para nuevas aplicaciones

Para agregar la documentación de una nueva aplicación, usa la plantilla en [`/ADICIONAR_APLICACAO.md`](../ADICIONAR_APLICACAO.md). Contiene instrucciones específicas para LLMs, incluyendo estructura, checklist y la plantilla markdown completa.

## Arquitectura recomendada para automatización de atención

```
WhatsApp del cliente
     |
evolution-go (conexión WhatsApp)
     |
     +-- Typebot (chatbot automático - FAQ, captura de datos)
     |       |
     |       +-- Dify (IA con base de conocimiento - respuestas inteligentes)
     |
     +-- Chatwoot (atención humana - agentes, cola, historial)
     |
n8n (orquestación entre todos los sistemas)
     |
MinIO (almacenamiento de archivos multimedia - fotos, audios, documentos)
```

## Arquitectura de monitoreo recomendada

```
Uptime Kuma (saber SI cayó)
    |
Netdata (saber POR QUÉ cayó)
    |
pgAdmin (investigar base de datos)
```

## Siguiente paso

[Odysseus](./01-odysseus.md) - Workspace de IA avanzado, o ve directo a:

- [Typebot](./04-typebot.md) - Chatbot para atención automática
- [Chatwoot](./05-chatwoot.md) - Helpdesk para atención humana
- [Dify](./06-dify.md) - IA con base de conocimiento
- [MinIO](./07-minio.md) - Almacenamiento S3 para todas las apps
- [Uptime Kuma](./11-uptime-kuma.md) - Monitoreo de uptime
- [Netdata](./12-netdata.md) - Monitoreo en tiempo real
