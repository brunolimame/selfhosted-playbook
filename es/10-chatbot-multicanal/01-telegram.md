# 10-01 - Telegram

## Cómo funciona

Telegram no tiene integración nativa directa con Typebot. Usamos **n8n como middleware**:

```
Usuario en Telegram
     |
Bot API (api.telegram.org)
     |
Webhook de n8n (recibe el mensaje)
     |
n8n llama a Typebot API (startChat / continueChat)
     |
Typebot procesa el flujo y devuelve respuesta
     |
n8n envía respuesta via Bot API (sendMessage)
     |
Usuario recibe en Telegram
```

## Prerrequisitos

- n8n instalado y funcionando ([documentación](../09-aplicacoes/02-n8n.md))
- Typebot instalado y funcionando ([documentación](../09-aplicacoes/04-typebot.md))
- Cuenta en Telegram

## 1. Crear el bot en Telegram (BotFather)

1. Abra Telegram y busque **@BotFather**
2. Envíe `/newbot`
3. Defina:
   - **Name**: `Mi Atención` (nombre visible)
   - **Username**: `mi_atencion_bot` (termina con `bot`)
4. BotFather devolverá el **token de la API**. Guárdelo:

```
1234567890:ABCdefGHIjklmNOPqrSTUvWXyz-ABCDEF
```

5. (Opcional) Configure comandos:
   ```
   /setcommands
   ```
   Envíe:
   ```
   start - Iniciar atención
   menu - Ver opciones
   contacto - Hablar con agente
   ayuda - Ayuda
   ```

## 2. En n8n: crear workflow para recibir mensajes

### Workflow: Telegram Webhook Receiver

**Trigger node**: Webhook
- Path: `/telegram-webhook`
- Method: POST

**Function node**: Extraer datos
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

**IF node**: Verificar si tiene texto
- Condition: `{{ $json.text.length > 0 }}`

**HTTP Request node**: Start/Continue chat con Typebot

URL (start - primer mensaje):
```
POST {{ $env.TYPEBOT_URL }}/api/v1/typebots/{{ $env.TYPEBOT_PUBLIC_ID }}/startChat
```

URL (continue - mensajes siguientes):
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

**Function node**: Procesar respuesta de Typebot
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

**HTTP Request node**: Enviar respuesta a Telegram
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

**Webhook Response node**: Responder 200 a Telegram
```json
{ "status": "ok" }
```

### Variables de entorno en n8n

Añada en el `.env` de n8n o en las credenciales del workflow:

```env
TELEGRAM_BOT_TOKEN=1234567890:ABCdefGHIjklmNOPqrSTUvWXyz-ABCDEF
TYPEBOT_URL=https://bot.meuservidor.com
TYPEBOT_PUBLIC_ID=su-typebot-public-id
```

## 3. Registrar el webhook en Telegram

Reemplace la URL y el token:

```bash
curl -X POST "https://api.telegram.org/botSU_TOKEN/setWebhook" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://n8n.meuservidor.com/webhook/telegram-webhook",
    "allowed_updates": ["message", "callback_query"]
  }'
```

Verificar si el webhook fue registrado:

```bash
curl "https://api.telegram.org/botSU_TOKEN/getWebhookInfo"
```

## 4. Gestionar sesiones en Typebot

n8n necesita almacenar el `sessionId` de Typebot para cada usuario de Telegram.

### Base de sesiones via n8n (in-memory)

En n8n, cree una **IF node** para verificar si existe sessionId:

```
{{ $items("HTTP Request - Start Chat").length > 0 }}
```

O use una base de datos ligera (SQLite via n8n) para persistir `chatId` -> `sessionId`.

## 5. Crear el flujo en Typebot

### Estructura básica del chatbot

```
Bloque: "¡Bienvenido a la atención!"
    Opciones:
      1. "Hablar con agente" ──> Activa Chatwoot
      2. "Preguntas frecuentes"  ──> Pregunta a Dify
      3. "Agendar servicio"    ──> Flujo de agendamiento

Bloque: "Escriba su pregunta"
    HTTP Request -> Dify API
    Muestra respuesta

Bloque: "Transfiriendo a un agente..."
    HTTP Request -> n8n (que crea ticket en Chatwoot)
```

### Configurar variables para Telegram

En Typebot, use variables para personalizar:

- `{{ canal }}` = `telegram`
- `{{ usuario_id }}` = `{{ $json.chatId }}`
- `{{ nome }}` = `{{ $json.username }}`

## 6. Probar

1. Abra Telegram y encuentre su bot
2. Envíe `/start`
3. El bot debe responder con el mensaje de bienvenida
4. Envíe una pregunta y vea el flujo funcionar

## Troubleshooting

| Error | Causa | Solución |
|------|-------|---------|
| `404 Not Found` en el webhook | URL del webhook incorrecta | Verificar `getWebhookInfo` |
| Bot no responde | Webhook no registrado | Ejecutar `setWebhook` nuevamente |
| `chat_id` inválido | Estructura del mensaje cambió | Loguear el body bruto en n8n |
| Sesión expirada | Typebot cerró sesión | Iniciar nueva sesión (startChat) |
| Timeout | n8n o Typebot lentos | Verificar logs de n8n |

## Próximo paso

[Facebook e Instagram](./02-facebook-instagram.md) - Añada más canales.
