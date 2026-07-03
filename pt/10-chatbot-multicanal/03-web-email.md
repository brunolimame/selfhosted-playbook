# 10-03 - Web Widget e Email

## Web Widget (Typebot Embed)

Diferente do Telegram e Facebook, o Typebot tem suporte **nativo** para web embed. Nao precisa de n8n como middleware.

### 1. Publicar o Typebot

1. Acesse `https://bot.meuservidor.com`
2. Abra o fluxo desejado
3. Clique em **Share** (ou **Publish**)
4. Selecione **Embed**

### 2. Incorporar no site

Copie o codigo e cole no `<body>` do seu site:

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

### 3. Personalizar o launcher

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

### 4. Preencher variaveis automaticamente

```html
<script type="module">
import Typebot from 'https://bot.meuservidor.com/api/v1/typebots/PUBLIC_ID/embed.js';
Typebot.init({
  typebot: 'PUBLIC_ID',
  apiHost: 'https://bot.meuservidor.com',
  prefilledVariables: {
    "canal": "web",
    "nome": "{{NOME_USUARIO}}",
    "email": "{{EMAIL_USUARIO}}",
    "pagina": window.location.pathname
  }
});
</script>
```

### 5. Abrir o bot com um botao customizado

```html
<button onclick="Typebot.open()">Fale conosco</button>
<button onclick="Typebot.close()">Fechar</button>
<button onclick="Typebot.toggle()">Abrir/Fechar</button>

<script type="module">
import Typebot from 'https://bot.meuservidor.com/api/v1/typebots/PUBLIC_ID/embed.js';
await Typebot.init({
  typebot: 'PUBLIC_ID',
  apiHost: 'https://bot.meuservidor.com',
  hideButton: true  // Esconde o launcher padrao
});
</script>
```

## Email (via n8n)

### Como funciona

```
Cliente envia email para atendimento@meuservidor.com
     |
n8n (IMAP) detecta novo email
     |
n8n extrai corpo do email
     |
n8n chama Typebot API (startChat / continueChat)
     |
Typebot processa e retorna resposta
     |
n8n envia resposta via SMTP
     |
Cliente recebe resposta no email
```

### 1. No n8n: Email Receiver Workflow

**IMAP node**:
- Credentials: Configurar IMAP (Gmail, Outlook, ou servidor proprio)
- Folder: INBOX
- Options: Only unseen = true

**Function node**: Extrair dados do email
```javascript
const email = $input.first().json;
const from = email.from?.[0]?.address || '';
const subject = email.subject || '';
const body = email.textPlain || email.textHtml || '';

// Limpar HTML
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

**IF node**: Se tiver resposta, enviar email

**SMTP node**:
- To: `{{ $json.email }}`
- Subject: `Re: {{ $json.subject }}`
- Text: `{{ $json.response }}`

**IMAP node**: Marcar como lido (Mark as Read = true)

### 2. Workflow: Email Sender (resposta automatica)

Para respostas rapidas sem Typebot (ex: "Recebemos seu email"), use um **Switch node**:

```javascript
const body = $json.body.toLowerCase();

if (body.includes('horario') || body.includes('funcionamento')) {
  return { tipo: 'faq' };
} else if (body.includes('atendente') || body.includes('humano')) {
  return { tipo: 'humano' };
} else {
  return { tipo: 'typebot' };
}
```

### 3. Roteamento para Chatwoot

Para emails que precisam de atendente humano, crie um ticket no Chatwoot via API:

```bash
curl -X POST https://atendimento.meuservidor.com/api/v1/accounts/1/conversations \
  -H "Content-Type: application/json" \
  -H "api_access_token: SEU_TOKEN" \
  -d '{
    "source_id": "email-cliente@exemplo.com",
    "inbox_id": 2,
    "contact_id": 1,
    "additional_attributes": {
      "mail_subject": "Preciso de ajuda",
      "mail_body": "Descricao do problema..."
    }
  }'
```

## Resumo dos canais

| Canal | Middleware | Tipo de integracao | Manutencao |
|-------|-----------|-------------------|------------|
| WhatsApp | evolution-go | Webhook direto | Media |
| Telegram | n8n | Webhook Telegram -> n8n | Media |
| Facebook | n8n | Webhook Meta -> n8n | Alta (Meta politicas) |
| Instagram | n8n | Webhook Meta -> n8n | Alta (Meta politicas) |
| Web | - | Typebot embed (JS) | Baixa |
| Email | n8n | IMAP -> n8n -> SMTP | Baixa |

## Proximo passo

[Fluxo Unificado](./04-flow-unificado.md) - Construa o fluxo central que atende todos os canais.
