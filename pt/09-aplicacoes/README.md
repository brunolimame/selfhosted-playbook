# 09 - Aplicacoes Avancadas

Guia de instalacao detalhado para aplicacoes avancadas que rodam sobre o Coolify (ou diretamente na VM).

## Indice

| Pagina | Descricao | Porta |
|--------|-----------|-------|
| [01 - Odysseus](./01-odysseus.md) | Workspace de IA auto-hospedado (chat, agentes, email, documentos) | 7000 |
| [02 - n8n](./02-n8n.md) | Automacao de workflows (alternativa ao Zapier) | 5678 |
| [03 - Evolution Go](./03-evolution-go.md) | API do WhatsApp em Go (evolution-api) | 4000 |
| [04 - Typebot](./04-typebot.md) | Chatbot visual com fluxos no-code (integracao nativa evolution) | 3001 / 3002 |
| [05 - Chatwoot](./05-chatwoot.md) | Helpdesk/CRM de atendimento multi-agente | 3000 |
| [06 - Dify](./06-dify.md) | IA com RAG e base de conhecimento (LLM + documentos) | 8080 |
| [07 - MinIO](./07-minio.md) | Armazenamento S3 para midia (compativel com todas as apps) | 9000 / 9001 |
| [08 - Evolution API (Node.js)](./08-evolution-api.md) | API Node.js completa (multi-provedor, painel web, integracoes nativas) | 8080 |
| [09 - Graphify](./09-graphify.md) | Grafo de conhecimento para assistentes de IA (codebase -> query) | 8080 |
| [10 - pgAdmin](./10-pgadmin.md) | Administracao web do PostgreSQL | 5050 |
| [11 - Uptime Kuma](./11-uptime-kuma.md) | Monitoramento de uptime com status page | 3001 |
| [12 - Netdata](./12-netdata.md) | Monitoramento em tempo real (CPU, RAM, disco, Docker) | 19999 |

## Forma de instalacao

As aplicacoes serao instaladas via Docker Compose diretamente na VM (fora do Coolify por enquanto), pois possuem dependencias complexas (PostgreSQL, Redis, etc.) que o Coolify pode nao gerenciar adequadamente.

O fluxo geral e:
1. Acessar a VM via SSH
2. Criar um diretorio para a aplicacao
3. Configurar o `docker-compose.yml` e `.env`
4. Iniciar com `docker compose up -d`
5. Configurar dominio no Cloudflare Tunnel
6. Testar o acesso

## Template para novas aplicacoes

Para adicionar a documentacao de uma nova aplicacao, use o template em [`/ADICIONAR_APLICACAO.md`](../ADICIONAR_APLICACAO.md). Ele contem instrucoes especificas para LLMs, incluindo estrutura, checklist e o template markdown completo.

## Arquitetura recomendada para automacao de atendimento

```
WhatsApp do cliente
     |
evolution-go (conexao WhatsApp)
     |
     +-- Typebot (chatbot automatico - FAQ, captura de dados)
     |       |
     |       +-- Dify (IA com base de conhecimento - respostas inteligentes)
     |
     +-- Chatwoot (atendimento humano - agentes, fila, historico)
     |
n8n (orquestracao entre todos os sistemas)
     |
MinIO (armazenamento de midia - fotos, audios, documentos)
```

## Arquitetura de monitoria recomendada

```
Uptime Kuma (saber SE caiu)
    |
Netdata (saber POR QUE caiu)
    |
pgAdmin (investigar banco de dados)
```

## Proximo passo

[Odysseus](./01-odysseus.md) - Workspace de IA avancado, ou va direto para:

- [Typebot](./04-typebot.md) - Chatbot para atendimento automatico
- [Chatwoot](./05-chatwoot.md) - Helpdesk para atendimento humano
- [Dify](./06-dify.md) - IA com base de conhecimento
- [MinIO](./07-minio.md) - Armazenamento S3 para todas as apps
- [Uptime Kuma](./11-uptime-kuma.md) - Monitoramento de uptime
- [Netdata](./12-netdata.md) - Monitoramento em tempo real
