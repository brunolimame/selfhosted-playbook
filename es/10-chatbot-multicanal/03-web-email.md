# 10-03 - Web Widget y Email

## Web Widget (Typebot Embed)

Diferente de Telegram y Facebook, Typebot tiene soporte **nativo** para web embed. No necesita n8n como middleware.

### 1. Publicar el Typebot

1. Acceda a `https://bot.meuservidor.com`
2. Abra el flujo deseado
3. Haga clic en **Share** (o **Publish**)
4. Seleccione **Embed**

### 2. Incorporar en el sitio

Copie el código y péguelo en el `<body>` de su sitio:

```html
<script type="module">
import Typebot from 'https://bot.meuservidor.com/api/v1/typebots/PUBLIC_ID/embed.js';
Typebot.init({
  typebot: 'PUBLIC_ID',
  apiHost: 'https://bot.meuservidor.com',
  prefilledVariables: {
    "canal": "web",
    "nome": "Visitante"
  }
});
</script>
```

### 3. Personalizar el launcher

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

### 4. Rellenar variables automáticamente

```html
<script type="module">
import Typebot from 'https://bot.meuservidor.com/api/v1/typebots/PUBLIC_ID/embed.js';
Typebot.init({
  typebot: 'PUBLIC_ID',
  apiHost: 'https://bot.meuservidor.com',
  prefilledVariables: {
    "canal": "web",
    "nome": "{{NOMBRE_USUARIO}}",
    "email": "{{EMAIL_USUARIO}}",
    "pagina": window.location.pathname
  }
});
</script>
```

### 5. Abrir el bot con un botón personalizado

```html
<button onclick="Typebot.open()">Hable con nosotros</button>
<button onclick="Typebot.close()">Cerrar</button>
<button onclick="Typebot.toggle()">Abrir/Cerrar</button>

<script type="module">
import Typebot from 'https://bot.meuservidor.com/api/v1/typebots/PUBLIC_ID/embed.js';
await Typebot.init({
  typebot: 'PUBLIC_ID',
  apiHost: 'https://bot.meuservidor.com',
  hideButton: true  // Oculta el launcher por defecto
});
</script>
```

## Email (via n8n)

### Cómo funciona

```
Cliente envía email a atencion@meuservidor.com
     |
n8n (IMAP) detecta nuevo email
     |
n8n extrae cuerpo del email
     |
n8n llama a Typebot API (startChat / continueChat)
     |
Typebot procesa y devuelve respuesta
     |
n8n envía respuesta via SMTP
     |
Cliente recibe respuesta en el email
```

### 1. En n8n: Email Receiver Workflow

**IMAP node**:
- Credentials: Configurar IMAP (Gmail, Outlook, o servidor propio)
- Folder: INBOX
- Options: Only unseen = true

**Function node**: Extraer datos del email
```javascript
const email = $input.first().json;
const from = email.from?.[0]?.address || '';
const subject = email.subject || '';
const body = email.textPlain || email.textHtml || '';

// Limpiar HTML
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
  "message": "Email de {{ $json.email }}: {{ $json.body }}",
  "variables": {
    "canal": "email",
    "email_cliente": "{{ $json.email }}",
    "assunto": "{{ $json.subject }}"
  }
}
```

**IF node**: Si tiene respuesta, enviar email

**SMTP node**:
- To: `{{ $json.email }}`
- Subject: `Re: {{ $json.subject }}`
- Text: `{{ $json.response }}`

**IMAP node**: Marcar como leído (Mark as Read = true)

### 2. Workflow: Email Sender (respuesta automática)

Para respuestas rápidas sin Typebot (ej: "Hemos recibido su email"), use un **Switch node**:

```javascript
const body = $json.body.toLowerCase();

if (body.includes('horario') || body.includes('horarios')) {
  return { tipo: 'faq' };
} else if (body.includes('agente') || body.includes('humano')) {
  return { tipo: 'humano' };
} else {
  return { tipo: 'typebot' };
}
```

### 3. Enrutamiento a Chatwoot

Para emails que necesitan un agente humano, cree un ticket en Chatwoot via API:

```bash
curl -X POST https://atendimento.meuservidor.com/api/v1/accounts/1/conversations \
  -H "Content-Type: application/json" \
  -H "api_access_token: SU_TOKEN" \
  -d '{
    "source_id": "email-cliente@ejemplo.com",
    "inbox_id": 2,
    "contact_id": 1,
    "additional_attributes": {
      "mail_subject": "Necesito ayuda",
      "mail_body": "Descripción del problema..."
    }
  }'
```

## Resumen de canales

| Canal | Middleware | Tipo de integración | Mantenimiento |
|-------|-----------|-------------------|------------|
| WhatsApp | evolution-go | Webhook directo | Medio |
| Telegram | n8n | Webhook Telegram -> n8n | Medio |
| Facebook | n8n | Webhook Meta -> n8n | Alto (políticas Meta) |
| Instagram | n8n | Webhook Meta -> n8n | Alto (políticas Meta) |
| Web | - | Typebot embed (JS) | Bajo |
| Email | n8n | IMAP -> n8n -> SMTP | Bajo |

## Próximo paso

[Flujo Unificado](./04-flow-unificado.md) - Construya el flujo central que atiende todos los canales.
