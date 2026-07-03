# 10 - Multi-channel Chatbot

Complete guide to building a chatbot that serves **WhatsApp, Telegram, Facebook, Instagram, Web and Email** using the applications already documented in this project.

## General architecture

```
INPUT CHANNELS                       ORCHESTRATION                      AI CENTER
                                                                     
Telegram ─────┐                                                      
WhatsApp ─────┤                  ┌──────────┐                  ┌──────────────┐
Facebook ─────┼── webhook ─────> │   n8n    │ ── Typebot ──>  │    Dify      │
Instagram ────┤                  │ (router) │    API          │ (RAG + LLM)  │
Web Widget ───┘                  │ central  │                  └──────────────┘
Email ────────────────────────>  └──────────┘                       
                                        │                           
                                        │                    ┌──────────────┐
                                        └───> Chatwoot ────> │  Human       │
                                                              │  Agent       │
                                                              └──────────────┘
```

## System components

| Component | Function | Documentation |
|-----------|--------|--------------|
| **Typebot** | Chatbot engine (visual flows) | [Typebot](../09-applications/04-typebot.md) |
| **n8n** | Universal message router | [n8n](../09-applications/02-n8n.md) |
| **evolution-go** | WhatsApp connectivity | [evolution-go](../09-applications/03-evolution-go.md) |
| **Dify** | AI with knowledge base (RAG) | [Dify](../09-applications/06-dify.md) |
| **Chatwoot** | Human support (helpdesk) | [Chatwoot](../09-applications/05-chatwoot.md) |
| **MinIO** | Media storage | [MinIO](../09-applications/07-minio.md) |

## Channel map

| Channel | Connectivity | Middleware | Response |
|-------|--------------|------------|----------|
| WhatsApp | evolution-go | n8n webhook | evolution-go |
| Telegram | Bot API (Telegram) | n8n webhook | Bot API |
| Facebook | Meta Graph API | n8n webhook | Meta API |
| Instagram | Meta Graph API | n8n webhook | Meta API |
| Web | Typebot embed | direct | Typebot |
| Email | IMAP/SMTP | n8n | SMTP |

## Standard message flow

```
1. User sends "What are your business hours?" on Telegram
2. Telegram calls n8n webhook
3. n8n identifies the channel (Telegram) and the user
4. n8n calls Typebot (startChat or continueChat)
5. Typebot executes the flow:
      a. "Question" block: "What are the hours?"
      b. "HTTP Request" block: queries Dify (RAG) with the question
      c. Dify searches documents and returns: "We operate from 8am to 6pm"
6. Typebot returns the response to n8n
7. n8n sends the response to Telegram via Bot API
8. User receives: "We operate from 8am to 6pm"
```

## Intelligent routing (who resolves?)

```
Customer message
     |
     v
Typebot evaluates context
     |
     +-- Simple question/FAQ? ──> Dify answers from knowledge base
     |
     +-- Predictable flow? ──> Typebot executes the flow (scheduling, registration)
     |
     +-- Keyword "agent"? ──> Chatwoot takes over (human)
     |
     +-- Angry/repeating customer? ──> Escalate to Chatwoot automatically
     |
     +-- Outside business hours? ──> Replies "Outside hours" and creates ticket in Chatwoot
```

## Next step

[Telegram](./01-telegram.md) - Set up the Telegram bot.
