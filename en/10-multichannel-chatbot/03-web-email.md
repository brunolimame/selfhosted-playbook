# 10-03 - Web Widget and Email

## Web Widget (Typebot Embed)

Unlike Telegram and Facebook, Typebot has **native** support for web embed. No need for n8n as middleware.

### 1. Publish the Typebot

1. Access `https://bot.meuservidor.com`
2. Open the desired flow
3. Click on **Share** (or **Publish**)
4. Select **Embed**

### 2. Embed in your site

Copy the code and paste it into the `<body>` of your site:

```html
<script type="module">
import Typebot from 'https://bot.meuservidor.com/api/v1/typebots/PUBLIC_ID/embed.js';
Typebot.init({
  typebot: 'PUBLIC_ID',
  apiHost: 'https://bot.meuservidor.com',
  prefilledVariables: {
    "channel": "web",
    "name": "Visitor"
  }
});
</script>
```

### 3. Customize the launcher

```html
<script type="module">
import Typebot from 'https://bot.meuservidor.com/api/v1/typebots/PUBLIC_ID/embed.js';
Typebot.init({
  typebot: 'PUBLIC_ID',
  apiHost: 'https://bot.meuservidor.com',
  theme: {
    button: {
      backgroundColor: '#2563eb',
      iconUrl: 'https://icones.io/chat.svg',
      customCss: 'border-radius: 50%; width: 60px; height: 60px;'
    },
    chatWindow: {
      backgroundColor: '#ffffff',
      bubbles: {
        user: { backgroundColor: '#2563eb', color: '#ffffff' },
        bot: { backgroundColor: '#f1f5f9', color: '#0f172a' }
      }
    }
  }
});
</script>
```

### 4. Prefill variables automatically

```html
<script type="module">
import Typebot from 'https://bot.meuservidor.com/api/v1/typebots/PUBLIC_ID/embed.js';
Typebot.init({
  typebot: 'PUBLIC_ID',
  apiHost: 'https://bot.meuservidor.com',
  prefilledVariables: {
    "channel": "web",
    "name": "{{USER_NAME}}",
    "email": "{{USER_EMAIL}}",
    "page": window.location.pathname
  }
});
</script>
```

### 5. Open the bot with a custom button

```html
<button onclick="Typebot.open()">Contact us</button>
<button onclick="Typebot.close()">Close</button>
<button onclick="Typebot.toggle()">Open/Close</button>

<script type="module">
import Typebot from 'https://bot.meuservidor.com/api/v1/typebots/PUBLIC_ID/embed.js';
await Typebot.init({
  typebot: 'PUBLIC_ID',
  apiHost: 'https://bot.meuservidor.com',
  hideButton: true  // Hides the default launcher
});
</script>
```

## Email (via n8n)

### How it works

```
Client sends email to atendimento@meuservidor.com
     |
n8n (IMAP) detects new email
     |
n8n extracts email body
     |
n8n calls Typebot API (startChat / continueChat)
     |
Typebot processes and returns response
     |
n8n sends response via SMTP
     |
Client receives reply via email
```

### 1. In n8n: Email Receiver Workflow

**IMAP node**:
- Credentials: Configure IMAP (Gmail, Outlook, or own server)
- Folder: INBOX
- Options: Only unseen = true

**Function node**: Extract email data
```javascript
const email = $input.first().json;
const from = email.from?.[0]?.address || '';
const subject = email.subject || '';
const body = email.textPlain || email.textHtml || '';

// Clean HTML
const cleanBody = body.replace(/<[^>]*>/g, '').trim();

return {
  email: from,
  subject: subject,
  body: cleanBody,
  messageId: email.messageId,
  date: email.date
};
```

**HTTP Request node**: Typebot API

```
POST https://bot.meuservidor.com/api/v1/typebots/PUBLIC_ID/startChat
```

Body:
```json
{
  "message": "Email from {{ $json.email }}: {{ $json.body }}",
  "variables": {
    "channel": "email",
    "client_email": "{{ $json.email }}",
    "subject": "{{ $json.subject }}"
  }
}
```

**IF node**: If there is a response, send email

**SMTP node**:
- To: `{{ $json.email }}`
- Subject: `Re: {{ $json.subject }}`
- Text: `{{ $json.response }}`

**IMAP node**: Mark as read (Mark as Read = true)

### 2. Workflow: Email Sender (auto reply)

For quick responses without Typebot (e.g., "We received your email"), use a **Switch node**:

```javascript
const body = $json.body.toLowerCase();

if (body.includes('hours') || body.includes('schedule')) {
  return { type: 'faq' };
} else if (body.includes('agent') || body.includes('human')) {
  return { type: 'human' };
} else {
  return { type: 'typebot' };
}
```

### 3. Routing to Chatwoot

For emails that need a human agent, create a ticket in Chatwoot via API:

```bash
curl -X POST https://atendimento.meuservidor.com/api/v1/accounts/1/conversations \
  -H "Content-Type: application/json" \
  -H "api_access_token: YOUR_TOKEN" \
  -d '{
    "source_id": "client-email@example.com",
    "inbox_id": 2,
    "contact_id": 1,
    "additional_attributes": {
      "mail_subject": "I need help",
      "mail_body": "Problem description..."
    }
  }'
```

## Channel summary

| Channel | Middleware | Integration type | Maintenance |
|-------|-----------|-------------------|------------|
| WhatsApp | evolution-go | Direct webhook | Medium |
| Telegram | n8n | Telegram Webhook -> n8n | Medium |
| Facebook | n8n | Meta Webhook -> n8n | High (Meta policies) |
| Instagram | n8n | Meta Webhook -> n8n | High (Meta policies) |
| Web | - | Typebot embed (JS) | Low |
| Email | n8n | IMAP -> n8n -> SMTP | Low |

## Next step

[Unified Flow](./04-unified-flow.md) - Build the central flow that serves all channels.
