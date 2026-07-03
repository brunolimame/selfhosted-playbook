# 10-01 - Telegram

## How it works

Telegram does not have direct native integration with Typebot. We use **n8n as middleware**:

```
User on Telegram
     |
Bot API (api.telegram.org)
     |
n8n Webhook (receives the message)
     |
n8n calls Typebot API (startChat / continueChat)
     |
Typebot processes the flow and returns response
     |
n8n sends response via Bot API (sendMessage)
     |
User receives on Telegram
```

## Prerequisites

- n8n installed and running ([documentation](../09-applications/02-n8n.md))
- Typebot installed and running ([documentation](../09-applications/04-typebot.md))
- Telegram account

## 1. Create the bot on Telegram (BotFather)

1. Open Telegram and search for **@BotFather**
2. Send `/newbot`
3. Define:
   - **Name**: `My Support` (visible name)
   - **Username**: `my_support_bot` (ends with `bot`)
4. BotFather will return the **API token**. Save it:

```
1234567890:ABCdefGHIjklmNOPqrSTUvWXyz-ABCDEF
```

5. (Optional) Configure commands:
   ```
   /setcommands
   ```
   Send:
   ```
   start - Start support
   menu - View options
   contact - Speak to an agent
   help - Help
   ```

## 2. In n8n: create workflow to receive messages

### Workflow: Telegram Webhook Receiver

**Trigger node**: Webhook
- Path: `/telegram-webhook`
- Method: POST

**Function node**: Extract data
```javascript
const body = $input.first().json.body;

const message = body.message || body.edited_message || body.callback_query?.message;
const chatId = message?.chat?.id;
const text = message?.text || body.callback_query?.data || '';
const username = message?.chat?.first_name || 'User';
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

**IF node**: Check if there is text
- Condition: `{{ $json.text.length > 0 }}`

**HTTP Request node**: Start/Continue chat with Typebot

URL (start - first message):
```
POST {{ $env.TYPEBOT_URL }}/api/v1/typebots/{{ $env.TYPEBOT_PUBLIC_ID }}/startChat
```

URL (continue - subsequent messages):
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

**Function node**: Process Typebot response
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

**HTTP Request node**: Send response to Telegram
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

**Webhook Response node**: Respond 200 to Telegram
```json
{ "status": "ok" }
```

### Environment variables in n8n

Add to n8n's `.env` or workflow credentials:

```env
TELEGRAM_BOT_TOKEN=1234567890:ABCdefGHIjklmNOPqrSTUvWXyz-ABCDEF
TYPEBOT_URL=https://bot.meuservidor.com
TYPEBOT_PUBLIC_ID=seu-typebot-public-id
```

## 3. Register the webhook with Telegram

Replace the URL and token:

```bash
curl -X POST "https://api.telegram.org/botYOUR_TOKEN/setWebhook" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://n8n.meuservidor.com/webhook/telegram-webhook",
    "allowed_updates": ["message", "callback_query"]
  }'
```

Verify the webhook was registered:

```bash
curl "https://api.telegram.org/botYOUR_TOKEN/getWebhookInfo"
```

## 4. Manage sessions in Typebot

n8n needs to store the Typebot `sessionId` for each Telegram user.

### Session store via n8n (in-memory)

In n8n, create an **IF node** to check if sessionId exists:

```
{{ $items("HTTP Request - Start Chat").length > 0 }}
```

Or use a lightweight database (SQLite via n8n) to persist `chatId` -> `sessionId`.

## 5. Create the flow in Typebot

### Basic chatbot structure

```
Block: "Welcome to our support!"
    Options:
      1. "Speak to an agent" ──> Triggers Chatwoot
      2. "Frequently asked questions"  ──> Ask Dify
      3. "Schedule service"    ──> Scheduling flow

Block: "Type your question"
    HTTP Request -> Dify API
    Display response

Block: "Transferring to an agent..."
    HTTP Request -> n8n (which creates ticket in Chatwoot)
```

### Configure variables for Telegram

In Typebot, use variables to personalize:

- `{{ channel }}` = `telegram`
- `{{ user_id }}` = `{{ $json.chatId }}`
- `{{ name }}` = `{{ $json.username }}`

## 6. Test

1. Open Telegram and find your bot
2. Send `/start`
3. The bot should respond with the welcome message
4. Send a question and watch the flow work

## Troubleshooting

| Error | Cause | Solution |
|------|-------|---------|
| `404 Not Found` on webhook | Wrong webhook URL | Check `getWebhookInfo` |
| Bot does not respond | Webhook not registered | Run `setWebhook` again |
| Invalid `chat_id` | Message structure changed | Log the raw body in n8n |
| Session expired | Typebot closed session | Start new session (startChat) |
| Timeout | n8n or Typebot slow | Check n8n logs |

## Next step

[Facebook and Instagram](./02-facebook-instagram.md) - Add more channels.
