# 10 - Chatbot Multicanal

Guia completo para construir um chatbot que atende em **WhatsApp, Telegram, Facebook, Instagram, Web e Email** usando as aplicacoes ja documentadas neste projeto.

## Arquitetura geral

```
CANAIS DE ENTRADA                     ORQUESTRACAO                      CENTRAL DE IA
                                                                     
Telegram ─────┐                                                       
WhatsApp ─────┤                  ┌──────────┐                  ┌──────────────┐
Facebook ─────┼── webhook ─────> │   n8n    │ ── Typebot ──>  │    Dify      │
Instagram ────┤                  │ (roteador│    API          │ (RAG + LLM)  │
Web Widget ───┘                  │  central)│                  └──────────────┘
Email ────────────────────────>  └──────────┘                       
                                       │                           
                                       │                    ┌──────────────┐
                                       └───> Chatwoot ────> │  Agente      │
                                                             │  Humano      │
                                                             └──────────────┘
```

## Componentes do sistema

| Componente | Funcao | Documentacao |
|-----------|--------|--------------|
| **Typebot** | Motor de chatbot (fluxos visuais) | [Typebot](../09-aplicacoes/04-typebot.md) |
| **n8n** | Roteador universal de mensagens | [n8n](../09-aplicacoes/02-n8n.md) |
| **evolution-go** | Conectividade WhatsApp | [evolution-go](../09-aplicacoes/03-evolution-go.md) |
| **Dify** | IA com base de conhecimento (RAG) | [Dify](../09-aplicacoes/06-dify.md) |
| **Chatwoot** | Atendimento humano (helpdesk) | [Chatwoot](../09-aplicacoes/05-chatwoot.md) |
| **MinIO** | Armazenamento de midia | [MinIO](../09-aplicacoes/07-minio.md) |

## Mapa de canais

| Canal | Conectividade | Middleware | Resposta |
|-------|--------------|------------|----------|
| WhatsApp | evolution-go | n8n webhook | evolution-go |
| Telegram | Bot API (Telegram) | n8n webhook | Bot API |
| Facebook | Meta Graph API | n8n webhook | Meta API |
| Instagram | Meta Graph API | n8n webhook | Meta API |
| Web | Typebot embed | direto | Typebot |
| Email | IMAP/SMTP | n8n | SMTP |

## Fluxo padrao de uma mensagem

```
1. Usuario envia "Qual o horario de funcionamento?" no Telegram
2. Telegram chama webhook do n8n
3. n8n identifica o canal (Telegram) e o usuario
4. n8n chama Typebot (startChat ou continueChat)
5. Typebot executa o fluxo:
      a. Bloco "Pergunta": "Qual o horario?"
      b. Bloco "HTTP Request": consulta Dify (RAG) com a pergunta
      c. Dify busca nos documentos e retorna: "Funcionamos das 8h as 18h"
6. Typebot retorna a resposta para n8n
7. n8n envia a resposta para o Telegram via Bot API
8. Usuario recebe: "Funcionamos das 8h as 18h"
```

## Roteamento inteligente (quem resolve?)

```
Mensagem do cliente
     |
     v
Typebot avalia o contexto
     |
     +-- Pergunta simples/FAQ? ──> Dify responde com base de conhecimento
     |
     +-- Fluxo previsivel? ──> Typebot executa o fluxo (agendamento, cadastro)
     |
     +-- Palavra-chave "atendente"? ──> Chatwoot assume (humano)
     |
     +-- Cliente irritado/repetindo? ──> Escala para Chatwoot automaticamente
     |
     +-- Fora do horario? ──> Responde "Fora do horario" e cria ticket no Chatwoot
```

## Proximo passo

[Telegram](./01-telegram.md) - Configure o bot do Telegram.
