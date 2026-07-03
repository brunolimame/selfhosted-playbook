# 10-02 - Facebook Messenger e Instagram

## Cómo funciona

Usamos la **Meta Graph API** conectada a **n8n** como middleware, siguiendo el mismo patrón de Telegram:

```
Usuario en Facebook Messenger / Instagram
     |
Meta envía webhook a n8n
     |
n8n llama a Typebot API (startChat / continueChat)
     |
Typebot procesa el flujo y devuelve respuesta
     |
n8n envía respuesta via Meta Send API
     |
Usuario recibe en Messenger / Instagram
```

## Prerrequisitos

- n8n instalado y funcionando ([documentación](../09-aplicacoes/02-n8n.md))
- Typebot instalado y funcionando ([documentación](../09-aplicacoes/04-typebot.md))
- Una **Página en Facebook** (para Messenger)
- Una **Cuenta Business en Instagram** (para Instagram DM)
- Una **Cuenta de Desarrollador en Meta**

## 1. Crear aplicación en Meta Developer

1. Acceda a https://developers.facebook.com
2. Haga clic en **My Apps > Create App**
3. Seleccione **Business > Next**
4. Nombre: `Mi Chatbot`
5. Añada el producto **Messenger**
6. Configure:

### Configurar página de Facebook

1. En **Settings > Advanced > Token de Página**
2. Seleccione su Página de Facebook
3. Genere un **Page Access Token**
4. Guarde el token - lo va a necesitar

### Configurar Instagram

1. En **Products > Messenger > Instagram**
2. Conecte su **Cuenta Business de Instagram**
3. Siga el flujo de permisos

## 2. Configurar Webhook en Meta

1. En **Products > Messenger > Settings > Webhooks**
2. URL de retorno: `https://n8n.meuservidor.com/webhook/meta-webhook`
3. Token de verificación: `mi-token-de-verificacion-meta`
4. Campos para suscribir:
   - `messages`
   - `messaging_postbacks`
   - `message_deliveries`
   - `message_reads`
5. Haga clic en **Verify and Save**

## 3. En n8n: crear workflow para Meta

### Workflow: Meta Webhook Receiver

**Webhook node** (dos modos):

**GET** (verificación de Meta):
```javascript
// Meta envía GET con hub.verify_token
// Devolver hub.challenge si el token coincide

const query = $input.first().json.query;
if (query['hub.verify_token'] === 'mi-token-de-verificacion-meta') {
  return query['hub.challenge'];
}
throw new Error('Token inválido');
```

**POST** (mensajes recibidos):
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

**HTTP Request node**: StartChat / ContinueChat con Typebot (mismo patrón de [Telegram](./01-telegram.md#2-en-n8n-crear-workflow-para-recibir-mensajes))

**HTTP Request node**: Enviar respuesta al usuario

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

### Variables de entorno

```env
META_PAGE_ACCESS_TOKEN=EAAx...
META_VERIFY_TOKEN=mi-token-de-verificacion-meta
TYPEBOT_URL=https://bot.meuservidor.com
TYPEBOT_PUBLIC_ID=su-typebot-public-id
```

## 4. Suscribir página al webhook

En el panel de Meta Developer:
1. **Webhooks > Edit Subscription**
2. Seleccione su página de Facebook
3. Haga clic en **Subscribe**

## 5. Enviar mensajes proactivos (opcional)

Para enviar mensajes sin que el usuario inicie (ej: recordatorio de cita):

```bash
curl -X POST "https://graph.facebook.com/v19.0/ME/messages" \
  -H "Authorization: Bearer EAAx..." \
  -H "Content-Type: application/json" \
  -d '{
    "recipient": {"id": "USUARIO_PSID"},
    "message": {"text": "¡Hola! Recordatorio: su cita es mañana a las 14h."},
    "messaging_type": "MESSAGE_TAG",
    "tag": "CONFIRMED_EVENT_UPDATE"
  }'
```

**Nota**: Los mensajes proactivos requieren etiquetas específicas (CONFIRMED_EVENT_UPDATE, POST_PURCHASE_UPDATE, etc.).

## 6. Sesiones unificadas con Typebot

Use los mismos flujos de Typebot para Telegram, Facebook y WhatsApp. n8n enruta según el canal:

En Typebot, cree una variable `canal` y rellénela en el HTTP Request:

```json
{
  "message": "{{ $json.text }}",
  "variables": {
    "canal": "facebook",
    "usuario_id": "{{ $json.senderId }}"
  }
}
```

## 7. Tratar medios (imágenes, audios, documentos)

### En n8n, detectar medios:

```javascript
const message = $input.first().json.body?.entry?.[0]?.messaging?.[0]?.message;

if (message?.attachments) {
  const attachment = message.attachments[0];
  const mediaUrl = attachment.payload?.url;

  // Descargar y enviar a Typebot como URL
  // O almacenar en MinIO y enviar URL
  return {
    senderId: message.sender?.id,
    text: '[Medio recibido]',
    mediaUrl: mediaUrl,
    mediaType: attachment.type
  };
}
```

## Próximo paso

[Web Widget y Email](./03-web-email.md) - Canales Web y Email automatizados.
