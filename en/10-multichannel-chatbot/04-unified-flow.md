# 10-04 - Unified Flow

Now that each channel is configured, let's build the **central flow in Typebot** that serves all channels with intelligent routing between Typebot, Dify and Chatwoot.

## Flow overview

```
START
  |
  +-- Variable "channel" defines the context
  |
  v
WELCOME (personalized by channel)
  |
  v
MAIN MENU
  |-- (1) Frequently Asked Questions --> Dify (RAG)
  |-- (2) Speak to an Agent --> Chatwoot
  |-- (3) Schedule Service   --> Scheduling flow
  |-- (4) Exit              --> End
  |
  v
FINAL EVALUATION
```

## 1. Input variables

In Typebot, create the following variables in the first block:

| Variable | Description | Example |
|----------|-----------|---------|
| `channel` | Origin channel | `whatsapp`, `telegram`, `facebook`, `instagram`, `web`, `email` |
| `user_id` | User ID on the channel | `5511999999999@s.whatsapp.net` |
| `name` | User name | `John` |
| `client_email` | Email (if available) | `john@email.com` |

## 2. Conditional welcome by channel

Use a **Switch block**:

```
Case channel = "telegram":
  "Hello {name}! Welcome to our support on Telegram."

Case channel = "whatsapp":
  "Hello {name}! Welcome to our support on WhatsApp."

Case channel = "web":
  "Hello! How can we help you?"

Case channel = "email":
  "We received your message. I will reply shortly."

Default case:
  "Hello {name}! Welcome."
```

## 3. Main menu

Create a **Choice** block:

```
[1] Frequently Asked Questions -> calls Dify (HTTP Request)
[2] Speak to an Agent -> calls Chatwoot (HTTP Request)
[3] Schedule Service -> scheduling flow
[4] Exit
```

## 4. Dify integration (intelligent FAQ)

**HTTP Request block**:

```
POST https://ia.meuservidor.com/v1/chat-messages
Headers:
  Authorization: Bearer app-xxxxx
  Content-Type: application/json
Body:
  {
    "inputs": {},
    "query": "{{ user_message }}",
    "response_mode": "blocking",
    "user": "{{ user_id }}"
  }
```

**Set Variable block**:
```
dify_response = {{ http_request.answer }}
```

**Text block**:
```
{{ dify_response }}

Type your next question or choose:
1. Back to menu
2. Speak to an agent
3. Exit
```

## 5. Chatwoot integration (transfer to human)

**HTTP Request block** (create conversation):

```
POST https://atendimento.meuservidor.com/api/v1/accounts/1/conversations
Headers:
  api_access_token: YOUR_CHATWOOT_TOKEN
  Content-Type: application/json
Body:
  {
    "source_id": "{{ user_id }}",
    "inbox_id": 1,
    "contact_id": 1,
    "status": "pending",
    "additional_attributes": {
      "channel": "{{ channel }}",
      "name": "{{ name }}"
    }
  }
```

**Text block**:
```
You will be assisted by an agent shortly. Thank you for your patience!
```

## 6. Scheduling flow (example)

1. **Text**: "Which service would you like to schedule?"
   - Consultation
   - Technical Support
   - Other

2. **Date Input**: "What is the best date?"

3. **Text**: "What time do you prefer?"

4. **HTTP Request** (n8n):
   ```
   POST https://n8n.meuservidor.com/webhook/agendamento
   Body: { "service": "...", "date": "...", "client": "{{ name }}" }
   ```

5. **Text**: "Scheduling confirmed! We will send a reminder."

## 7. Central n8n router

Create a workflow that receives from all channels and routes:

```javascript
// Function node: Central Router
const channel = $json.channel;
const config = {
  telegram: {
    url: `https://api.telegram.org/bot${process.env.TELEGRAM_TOKEN}/sendMessage`,
    body: { chat_id: $json.chatId, text: $json.reply }
  },
  facebook: {
    url: `https://graph.facebook.com/v19.0/me/messages`,
    headers: { Authorization: `Bearer ${process.env.META_TOKEN}` },
    body: { recipient: { id: $json.senderId }, message: { text: $json.reply }, messaging_type: 'RESPONSE' }
  },
  whatsapp: {
    url: `http://192.168.1.100:4000/message/sendText`,
    headers: { apiKey: process.env.EVOLUTION_API_KEY },
    body: { number: $json.number, textMessage: { text: $json.reply } }
  }
};
return config[channel];
```

## 8. Error handling

### Dify timeout
```
Sorry, I'm having trouble. Could you rephrase?
Or type "agent" to speak to a human.
```

### Typebot session expired
```javascript
if (response.status === 404) {
  const newSession = await startNewSession(typebotId);
  await saveSessionId(userId, newSession.sessionId);
  // Resend the message
}
```

## 9. Logs and metrics

Log each interaction via n8n webhook:

```javascript
await $http.post('https://n8n.meuservidor.com/webhook/log', {
  timestamp: new Date(),
  channel: $json.channel,
  user: $json.user_id,
  message: $json.text,
  response: $json.reply,
  time_ms: responseTime,
  route: 'dify' | 'chatwoot' | 'typebot'
});
```

### Metrics to track
- Volume per channel
- Auto-resolution vs. human rate
- Average response time
- Peak hours
- Most frequent keywords

## 10. Complete Typebot structure

```
[1] VARIABLES: channel, user_id, name, client_email

[2] SWITCH: channel
    -> telegram: "Hello {name}! Welcome to Telegram"
    -> whatsapp: "Hello {name}! Welcome to WhatsApp"
    -> web: "Hello! How can I help?"
    -> email: "We received your email."
    -> default: "Hello {name}!"

[3] CHOICE: "Choose an option"
    -> "Frequently Asked Questions"
         [4] HTTP -> Dify
         [5] TEXT -> response
         [6] CHOICE -> "Anything else?"
              -> Yes: back to [3]
              -> No: end
    -> "Speak to an Agent"
         [7] HTTP -> Chatwoot
         [8] TEXT: "Transferring..."
    -> "Schedule Service"
         [9] TEXT: "Which service?"
         [10] DATE: "What date?"
         [11] TEXT: "What time?"
         [12] HTTP -> n8n
         [13] TEXT: "Confirmed!"
    -> "Exit"
         [14] TEXT: "Thank you!"
```

## 11. Deployment checklist

- [ ] Typebot with multi-channel flow configured
- [ ] n8n with workflows: Telegram, Facebook, Email
- [ ] evolution-go connected to Typebot
- [ ] Dify with knowledge base loaded
- [ ] Chatwoot with Evolution inbox configured
- [ ] MinIO running
- [ ] Cloudflare Tunnel with domains:
  - `bot.meuservidor.com` (Typebot)
  - `n8n.meuservidor.com` (n8n)
  - `ia.meuservidor.com` (Dify)
  - `atendimento.meuservidor.com` (Chatwoot)
- [ ] Telegram webhook registered
- [ ] Meta webhook configured
- [ ] Email IMAP/SMTP tested

## Next step

[Security](../07-security.md) - Protect the entire ecosystem.
