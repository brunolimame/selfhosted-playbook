# 10 - Chatbot Multicanal

Guía completa para construir un chatbot que atiende en **WhatsApp, Telegram, Facebook, Instagram, Web y Email** usando las aplicaciones ya documentadas en este proyecto.

## Arquitectura general

```
CANALES DE ENTRADA                     ORQUESTRACIÓN                      CENTRAL DE IA
                                                                      
Telegram ─────┐                                                       
WhatsApp ─────┤                  ┌──────────┐                  ┌──────────────┐
Facebook ─────┼── webhook ─────> │   n8n    │ ── Typebot ──>  │    Dify      │
Instagram ────┤                  │ (router  │    API          │ (RAG + LLM)  │
Web Widget ───┘                  │  central)│                  └──────────────┘
Email ────────────────────────>  └──────────┘                       
                                        │                           
                                        │                    ┌──────────────┐
                                        └───> Chatwoot ────> │  Agente      │
                                                              │  Humano      │
                                                              └──────────────┘
```

## Componentes del sistema

| Componente | Función | Documentación |
|-----------|--------|--------------|
| **Typebot** | Motor de chatbot (flujos visuales) | [Typebot](../09-aplicacoes/04-typebot.md) |
| **n8n** | Router universal de mensajes | [n8n](../09-aplicacoes/02-n8n.md) |
| **evolution-go** | Conectividad WhatsApp | [evolution-go](../09-aplicacoes/03-evolution-go.md) |
| **Dify** | IA con base de conocimiento (RAG) | [Dify](../09-aplicacoes/06-dify.md) |
| **Chatwoot** | Atención humana (helpdesk) | [Chatwoot](../09-aplicacoes/05-chatwoot.md) |
| **MinIO** | Almacenamiento de medios | [MinIO](../09-aplicacoes/07-minio.md) |

## Mapa de canales

| Canal | Conectividad | Middleware | Respuesta |
|-------|--------------|------------|----------|
| WhatsApp | evolution-go | n8n webhook | evolution-go |
| Telegram | Bot API (Telegram) | n8n webhook | Bot API |
| Facebook | Meta Graph API | n8n webhook | Meta API |
| Instagram | Meta Graph API | n8n webhook | Meta API |
| Web | Typebot embed | directo | Typebot |
| Email | IMAP/SMTP | n8n | SMTP |

## Flujo estándar de un mensaje

```
1. Usuario envía "¿Cuál es el horario de atención?" en Telegram
2. Telegram llama webhook de n8n
3. n8n identifica el canal (Telegram) y el usuario
4. n8n llama a Typebot (startChat o continueChat)
5. Typebot ejecuta el flujo:
      a. Bloque "Pregunta": "¿Cuál es el horario?"
      b. Bloque "HTTP Request": consulta Dify (RAG) con la pregunta
      c. Dify busca en los documentos y responde: "Atendemos de 8h a 18h"
6. Typebot devuelve la respuesta a n8n
7. n8n envía la respuesta a Telegram via Bot API
8. Usuario recibe: "Atendemos de 8h a 18h"
```

## Enrutamiento inteligente (¿quién resuelve?)

```
Mensaje del cliente
     |
     v
Typebot evalúa el contexto
     |
     +-- ¿Pregunta simple/FAQ? ──> Dify responde con base de conocimiento
     |
     +-- ¿Flujo predecible? ──> Typebot ejecuta el flujo (agendamiento, registro)
     |
     +-- ¿Palabra clave "agente"? ──> Chatwoot asume (humano)
     |
     +-- ¿Cliente irritado/repitiendo? ──> Escala a Chatwoot automáticamente
     |
     +-- ¿Fuera de horario? ──> Responde "Fuera de horario" y crea ticket en Chatwoot
```

## Próximo paso

[Telegram](./01-telegram.md) - Configure el bot de Telegram.
