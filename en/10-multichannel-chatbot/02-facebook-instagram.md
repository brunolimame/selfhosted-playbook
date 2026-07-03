# 10-02 - Facebook Messenger and Instagram

## How it works

We use the **Meta Graph API** connected to **n8n** as middleware, following the same pattern as Telegram:

```
User on Facebook Messenger / Instagram
     |
Meta sends webhook to n8n
     |
n8n calls Typebot API (startChat / continueChat)
     |
Typebot processes the flow and returns response
     |
n8n sends response via Meta Send API
     |
User receives on Messenger / Instagram
```

## Prerequisites

- n8n installed and running ([documentation](../09-applications/02-n8n.md))
- Typebot installed and running ([documentation](../09-applications/04-typebot.md))
- A **Facebook Page** (for Messenger)
- An **Instagram Business Account** (for Instagram DM)
- A **Meta Developer Account**

## 1. Create app on Meta Developer

1. Access https://developers.facebook.com
2. Click on **My Apps > Create App**
3. Select **Business > Next**
4. Name: `My Chatbot`
5. Add the **Messenger** product
6. Configure:

### Configure Facebook page

1. In **Settings > Advanced > Page Token**
2. Select your Facebook Page
3. Generate a **Page Access Token**
4. Save the token - you will need it

### Configure Instagram

1. In **Products > Messenger > Instagram**
2. Connect your **Instagram Business Account**
3. Follow the permission flow

## 2. Configure Webhook on Meta

1. In **Products > Messenger > Settings > Webhooks**
2. Callback URL: `https://n8n.meuservidor.com/webhook/meta-webhook`
3. Verify token: `meu-token-de-verificacao-meta`
4. Fields to subscribe:
   - `messages`
   - `messaging_postbacks`
   - `message_deliveries`
   - `message_reads`
5. Click on **Verify and Save**

## 3. In n8n: create workflow for Meta

### Workflow: Meta Webhook Receiver

**Webhook node** (two modes):

**GET** (Meta verification):
```javascript
// Meta sends GET with hub.verify_token
// Return hub.challenge if the token matches

const query = $input.first().json.query;
if (query['hub.verify_token'] === 'meu-token-de-verificacao-meta') {
  return query['hub.challenge'];
}
throw new Error('Invalid token');
```

**POST** (received messages):
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

**HTTP Request node**: StartChat / ContinueChat with Typebot (same pattern as [Telegram](./01-telegram.md#2-no-n8n-criar-workflow-para-receber-mensagens))

**HTTP Request node**: Send response to user

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

### Environment variables

```env
META_PAGE_ACCESS_TOKEN=EAAx...
META_VERIFY_TOKEN=meu-token-de-verificacao-meta
TYPEBOT_URL=https://bot.meuservidor.com
TYPEBOT_PUBLIC_ID=seu-typebot-public-id
```

## 4. Subscribe page to webhook

In Meta Developer panel:
1. **Webhooks > Edit Subscription**
2. Select your Facebook page
3. Click on **Subscribe**

## 5. Send proactive messages (optional)

To send messages without the user initiating (e.g., appointment reminder):

```bash
curl -X POST "https://graph.facebook.com/v19.0/ME/messages" \
  -H "Authorization: Bearer EAAx..." \
  -H "Content-Type: application/json" \
  -d '{
    "recipient": {"id": "USER_PSID"},
    "message": {"text": "Hi! Reminder that your appointment is tomorrow at 2pm."},
    "messaging_type": "MESSAGE_TAG",
    "tag": "CONFIRMED_EVENT_UPDATE"
  }'
```

**Note**: Proactive messages require specific tags (CONFIRMED_EVENT_UPDATE, POST_PURCHASE_UPDATE, etc.).

## 6. Unified sessions with Typebot

Use the same Typebot flows for Telegram, Facebook and WhatsApp. n8n routes based on the channel:

In Typebot, create a `channel` variable and fill it in the HTTP Request:

```json
{
  "message": "{{ $json.text }}",
  "variables": {
    "channel": "facebook",
    "user_id": "{{ $json.senderId }}"
  }
}
```

## 7. Handle media (images, audio, documents)

### In n8n, detect media:

```javascript
const message = $input.first().json.body?.entry?.[0]?.messaging?.[0]?.message;

if (message?.attachments) {
  const attachment = message.attachments[0];
  const mediaUrl = attachment.payload?.url;

  // Download and send to Typebot as URL
  // Or store in MinIO and send URL
  return {
    senderId: message.sender?.id,
    text: '[Media received]',
    mediaUrl: mediaUrl,
    mediaType: attachment.type
  };
}
```

## Next step

[Web Widget and Email](./03-web-email.md) - Automated Web and Email channels.
