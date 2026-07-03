# 10-02 - Facebook Messenger e Instagram

## Como funciona

Usamos o **Meta Graph API** conectado ao **n8n** como middleware, seguindo o mesmo padrao do Telegram:

```
Usuario no Facebook Messenger / Instagram
     |
Meta envia webhook para n8n
     |
n8n chama Typebot API (startChat / continueChat)
     |
Typebot processa o fluxo e retorna resposta
     |
n8n envia resposta via Meta Send API
     |
Usuario recebe no Messenger / Instagram
```

## Pre-requisitos

- n8n instalado e rodando ([documentacao](../09-aplicacoes/02-n8n.md))
- Typebot instalado e rodando ([documentacao](../09-aplicacoes/04-typebot.md))
- Uma **Pagina no Facebook** (para Messenger)
- Uma **Conta Business no Instagram** (para Instagram DM)
- Uma **Conta de Desenvolvedor no Meta**

## 1. Criar aplicacao no Meta Developer

1. Acesse https://developers.facebook.com
2. Clique em **My Apps > Create App**
3. Selecione **Business > Next**
4. Nome: `Meu Chatbot`
5. Adicione o produto **Messenger**
6. Configure:

### Configurar pagina do Facebook

1. Em **Settings > Advanced > Token do Pagina**
2. Selecione sua Pagina do Facebook
3. Gere um **Page Access Token**
4. Salve o token - voce vai precisar

### Configurar Instagram

1. Em **Products > Messenger > Instagram**
2. Conecte sua **Conta Business do Instagram**
3. Siga o fluxo de permissao

## 2. Configurar Webhook no Meta

1. Em **Products > Messenger > Settings > Webhooks**
2. URL de retorno: `https://n8n.meuservidor.com/webhook/meta-webhook`
3. Token de verificacao: `meu-token-de-verificacao-meta`
4. Campos para inscrever:
   - `messages`
   - `messaging_postbacks`
   - `message_deliveries`
   - `message_reads`
5. Clique em **Verify and Save**

## 3. No n8n: criar workflow para Meta

### Workflow: Meta Webhook Receiver

**Webhook node** (dois modos):

**GET** (verificacao do Meta):
```javascript
// Meta envia GET com hub.verify_token
// Retornar hub.challenge se o token bater

const query = $input.first().json.query;
if (query['hub.verify_token'] === 'meu-token-de-verificacao-meta') {
  return query['hub.challenge'];
}
throw new Error('Token invalido');
```

**POST** (mensagens recebidas):
```javascript
const body = $input.first().json.body;

for (const entry of body.entry || []) {
  for (const messaging of entry.messaging || []) {
    const senderId = messaging.sender?.id;
    const message = messaging.message?.text || '';
    const postback = messaging.postback?.payload || '';
    const text = message || postback;

    if (text) {
      return {
        senderId: senderId,
        text: text,
        pageId: entry.id,
        timestamp: messaging.timestamp,
        isInstagram: !!messaging.message?.is_instagram
      };
    }
  }
}
```

**HTTP Request node**: StartChat / ContinueChat com Typebot (mesmo padrao do [Telegram](./01-telegram.md#2-no-n8n-criar-workflow-para-receber-mensagens))

**HTTP Request node**: Enviar resposta ao usuario

```
POST https://graph.facebook.com/v19.0/ME/me/messages
```

Headers:
```
Authorization: Bearer {{ $env.META_PAGE_ACCESS_TOKEN }}
Content-Type: application/json
```

Body:
```json
{
  "recipient": {
    "id": "{{ $json.senderId }}"
  },
  "message": {
    "text": "{{ $json.replyText }}"
  },
  "messaging_type": "RESPONSE"
}
```

### Variaveis de ambiente

```env
META_PAGE_ACCESS_TOKEN=EAAx...
META_VERIFY_TOKEN=meu-token-de-verificacao-meta
TYPEBOT_URL=https://bot.meuservidor.com
TYPEBOT_PUBLIC_ID=seu-typebot-public-id
```

## 4. Assinar pagina no webhook

No painel do Meta Developer:
1. **Webhooks > Edit Subscription**
2. Selecione sua pagina do Facebook
3. Clique em **Subscribe**

## 5. Enviar mensagens proativas (opcional)

Para enviar mensagens sem o usuario iniciar (ex: lembrete de consulta):

```bash
curl -X POST "https://graph.facebook.com/v19.0/ME/messages" \
  -H "Authorization: Bearer EAAx..." \
  -H "Content-Type: application/json" \
  -d '{
    "recipient": {"id": "USUARIO_PSID"},
    "message": {"text": "Oi! Lembrando que sua consulta e amanha as 14h."},
    "messaging_type": "MESSAGE_TAG",
    "tag": "CONFIRMED_EVENT_UPDATE"
  }'
```

**Nota**: Mensagens proativas exigem tags especificas (CONFIRMED_EVENT_UPDATE, POST_PURCHASE_UPDATE, etc.).

## 6. Sessoes unificadas com Typebot

Use os mesmos fluxos do Typebot para Telegram, Facebook e WhatsApp. O n8n roteia com base no canal:

No Typebot, crie uma variavel `canal` e preencha no HTTP Request:

```json
{
  "message": "{{ $json.text }}",
  "variables": {
    "canal": "facebook",
    "usuario_id": "{{ $json.senderId }}"
  }
}
```

## 7. Tratar midia (imagens, audios, documentos)

### No n8n, detectar midia:

```javascript
const message = $input.first().json.body?.entry?.[0]?.messaging?.[0]?.message;

if (message?.attachments) {
  const attachment = message.attachments[0];
  const mediaUrl = attachment.payload?.url;

  // Baixar e enviar para Typebot como URL
  // Ou armazenar no MinIO e enviar URL
  return {
    senderId: message.sender?.id,
    text: '[Midia recebida]',
    mediaUrl: mediaUrl,
    mediaType: attachment.type
  };
}
```

## Proximo passo

[Web Widget e Email](./03-web-email.md) - Canais Web e Email automatizados.
