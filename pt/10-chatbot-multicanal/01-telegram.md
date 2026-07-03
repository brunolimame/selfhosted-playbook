# 10-01 - Telegram

## Como funciona

O Telegram nao tem integracao nativa direta com Typebot. Usamos o **n8n como middleware**:

```
Usuario no Telegram
     |
Bot API (api.telegram.org)
     |
Webhook do n8n (recebe a mensagem)
     |
n8n chama Typebot API (startChat / continueChat)
     |
Typebot processa o fluxo e retorna resposta
     |
n8n envia resposta via Bot API (sendMessage)
     |
Usuario recebe no Telegram
```

## Pre-requisitos

- n8n instalado e rodando ([documentacao](../09-aplicacoes/02-n8n.md))
- Typebot instalado e rodando ([documentacao](../09-aplicacoes/04-typebot.md))
- Conta no Telegram

## 1. Criar o bot no Telegram (BotFather)

1. Abra o Telegram e pesquise por **@BotFather**
2. Envie `/newbot`
3. Defina:
   - **Name**: `Meu Atendimento` (nome visivel)
   - **Username**: `meu_atendimento_bot` (termina com `bot`)
4. O BotFather retornara o **token da API**. Salve-o:

```
1234567890:ABCdefGHIjklmNOPqrSTUvWXyz-ABCDEF
```

5. (Opcional) Configure comandos:
   ```
   /setcommands
   ```
   Envie:
   ```
   start - Iniciar atendimento
   menu - Ver opcoes
   contato - Falar com atendente
   ajuda - Ajuda
   ```

## 2. No n8n: criar workflow para receber mensagens

### Workflow: Telegram Webhook Receiver

**Trigger node**: Webhook
- Path: `/telegram-webhook`
- Method: POST

**Function node**: Extrair dados
```javascript
const body = $input.first().json.body;

const message = body.message || body.edited_message || body.callback_query?.message;
const chatId = message?.chat?.id;
const text = message?.text || body.callback_query?.data || '';
const username = message?.chat?.first_name || 'Usuario';
const messageId = message?.message_id;
const chatType = message?.chat?.type;

return {
  chatId: String(chatId),
  text: text,
  username: username,
  messageId: messageId,
  chatType: chatType,
  raw: body
};
```

**IF node**: Verificar se tem texto
- Condition: `{{ $json.text.length > 0 }}`

**HTTP Request node**: Start/Continue chat com Typebot

URL (start - primeira mensagem):
```
POST {{ $env.TYPEBOT_URL }}/api/v1/typebots/{{ $env.TYPEBOT_PUBLIC_ID }}/startChat
```

URL (continue - mensagens seguintes):
```
POST {{ $env.TYPEBOT_URL }}/api/v1/sessions/{{ $json.sessionId }}/continueChat
```

Body (start):
```json
{
  "isStreaming": false
}
```

Body (continue):
```json
{
  "message": "{{ $json.text }}"
}
```

**Function node**: Processar resposta do Typebot
```javascript
const response = $input.first().json;
const messages = response.messages || [];
const sessionId = response.sessionId;

let replies = [];

for (const msg of messages) {
  if (msg.type === 'text') {
    replies.push({
      type: 'text',
      content: msg.content?.richText?.[0]?.children?.[0]?.text || msg.content?.plainText || ''
    });
  } else if (msg.type === 'image') {
    replies.push({ type: 'image', content: msg.content?.url });
  }
}

return {
  chatId: $json.chatId,
  sessionId: sessionId,
  replies: replies,
  input: response.input
};
```

**HTTP Request node**: Enviar resposta ao Telegram
```
POST https://api.telegram.org/bot{{ $env.TELEGRAM_BOT_TOKEN }}/sendMessage
```

Body:
```json
{
  "chat_id": "{{ $json.chatId }}",
  "text": "{{ $json.replies[0].content }}",
  "parse_mode": "Markdown"
}
```

**Webhook Response node**: Responder 200 ao Telegram
```json
{ "status": "ok" }
```

### Variaveis de ambiente no n8n

Adicione no `.env` do n8n ou nas credenciais do workflow:

```env
TELEGRAM_BOT_TOKEN=1234567890:ABCdefGHIjklmNOPqrSTUvWXyz-ABCDEF
TYPEBOT_URL=https://bot.meuservidor.com
TYPEBOT_PUBLIC_ID=seu-typebot-public-id
```

## 3. Registrar o webhook no Telegram

Substitua a URL e o token:

```bash
curl -X POST "https://api.telegram.org/botSEU_TOKEN/setWebhook" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://n8n.meuservidor.com/webhook/telegram-webhook",
    "allowed_updates": ["message", "callback_query"]
  }'
```

Verificar se o webhook foi registrado:

```bash
curl "https://api.telegram.org/botSEU_TOKEN/getWebhookInfo"
```

## 4. Gerenciar sessoes no Typebot

O n8n precisa armazenar o `sessionId` do Typebot para cada usuario do Telegram.

### Banco de sessoes via n8n (in-memory)

No n8n, crie uma **IF node** para verificar se existe sessionId:

```
{{ $items("HTTP Request - Start Chat").length > 0 }}
```

Ou use um banco de dados leve (SQLite via n8n) para persistir `chatId` -> `sessionId`.

## 5. Criar o fluxo no Typebot

### Estrutura basica do chatbot

```
Bloco: "Bem-vindo ao atendimento!"
    Opcoes:
      1. "Falar com atendente" ──> Aciona Chatwoot
      2. "Duvidas frequentes"  ──> Pergunta ao Dify
      3. "Agendar servico"    ──> Fluxo de agendamento

Bloco: "Digite sua duvida"
    HTTP Request -> Dify API
    Exibe resposta

Bloco: "Transferindo para atendente..."
    HTTP Request -> n8n (que cria ticket no Chatwoot)
```

### Configurar variaveis para o Telegram

No Typebot, use variaveis para personalizar:

- `{{ canal }}` = `telegram`
- `{{ usuario_id }}` = `{{ $json.chatId }}`
- `{{ nome }}` = `{{ $json.username }}`

## 6. Testar

1. Abra o Telegram e encontre seu bot
2. Envie `/start`
3. O bot deve responder com a mensagem de boas-vindas
4. Envie uma pergunta e veja o fluxo funcionar

## Troubleshooting

| Erro | Causa | Solucao |
|------|-------|---------|
| `404 Not Found` no webhook | URL do webhook errada | Verificar `getWebhookInfo` |
| Bot nao responde | Webhook nao registrado | Rodar `setWebhook` novamente |
| `chat_id` invalido | Estrutura da mensagem mudou | Logar o body bruto no n8n |
| Sessao expirada | Typebot fechou sessao | Iniciar nova sessao (startChat) |
| Timeout | n8n ou Typebot lentos | Verificar logs do n8n |

## Proximo passo

[Facebook e Instagram](./02-facebook-instagram.md) - Adicione mais canais.
