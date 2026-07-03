# 10-04 - Flujo Unificado

Ahora que cada canal está configurado, vamos a construir el **flujo central en Typebot** que atiende todos los canales con enrutamiento inteligente entre Typebot, Dify y Chatwoot.

## Vista general del flujo

```
INICIO
  |
  +-- Variable "canal" define el contexto
  |
  v
BIENVENIDA (personalizada por canal)
  |
  v
MENÚ PRINCIPAL
  |-- (1) Preguntas Frecuentes --> Dify (RAG)
  |-- (2) Hablar con Agente --> Chatwoot
  |-- (3) Agendar Servicio   --> Flujo de agendamiento
  |-- (4) Salir              --> Finaliza
  |
  v
EVALUACIÓN FINAL
```

## 1. Variables de entrada

En Typebot, cree en el primer bloque las variables:

| Variable | Descripción | Ejemplo |
|----------|-----------|---------|
| `canal` | Canal de origen | `whatsapp`, `telegram`, `facebook`, `instagram`, `web`, `email` |
| `usuario_id` | ID del usuario en el canal | `5511999999999@s.whatsapp.net` |
| `nome` | Nombre del usuario | `Juan` |
| `email_cliente` | Email (si está disponible) | `juan@email.com` |

## 2. Bienvenida condicional por canal

Use un **Switch block**:

```
Caso canal = "telegram":
  "¡Hola {nome}! Bienvenido a la atención por Telegram."

Caso canal = "whatsapp":
  "¡Hola {nome}! Bienvenido a la atención por WhatsApp."

Caso canal = "web":
  "¡Hola! ¿Cómo podemos ayudar?"

Caso canal = "email":
  "Hemos recibido su mensaje. Le responderé en breve."

Caso por defecto:
  "¡Hola {nome}! Sea bienvenido(a)."
```

## 3. Menú principal

Cree un bloque **Choice**:

```
[1] Preguntas Frecuentes -> llama a Dify (HTTP Request)
[2] Hablar con Agente -> llama a Chatwoot (HTTP Request)
[3] Agendar Servicio -> flujo de agendamiento
[4] Finalizar
```

## 4. Integración con Dify (FAQ inteligente)

**Bloque HTTP Request**:

```
POST https://ia.meuservidor.com/v1/chat-messages
Headers:
  Authorization: Bearer app-xxxxx
  Content-Type: application/json
Body:
  {
    "inputs": {},
    "query": "{{ mensagem_do_usuario }}",
    "response_mode": "blocking",
    "user": "{{ usuario_id }}"
  }
```

**Bloque Set Variable**:
```
respuesta_dify = {{ http_request.answer }}
```

**Bloque Text**:
```
{{ respuesta_dify }}

Escriba su siguiente pregunta o elija:
1. Volver al menú
2. Hablar con agente
3. Finalizar
```

## 5. Integración con Chatwoot (transferencia a humano)

**Bloque HTTP Request** (crear conversación):

```
POST https://atendimento.meuservidor.com/api/v1/accounts/1/conversations
Headers:
  api_access_token: SU_TOKEN_CHATWOOT
  Content-Type: application/json
Body:
  {
    "source_id": "{{ usuario_id }}",
    "inbox_id": 1,
    "contact_id": 1,
    "status": "pending",
    "additional_attributes": {
      "canal": "{{ canal }}",
      "nome": "{{ nome }}"
    }
  }
```

**Bloque Text**:
```
Un agente le atenderá en breve. ¡Gracias por su paciencia!
```

## 6. Flujo de agendamiento (ejemplo)

1. **Text**: "¿Qué servicio desea agendar?"
   - Consulta
   - Soporte Técnico
   - Otro

2. **Date Input**: "¿Cuál es la mejor fecha?"

3. **Text**: "¿Qué horario prefiere?"

4. **HTTP Request** (n8n):
   ```
   POST https://n8n.meuservidor.com/webhook/agendamiento
   Body: { "servico": "...", "data": "...", "cliente": "{{ nome }}" }
   ```

5. **Text**: "¡Agendamiento confirmado! Enviaremos un recordatorio."

## 7. Router central en n8n

Cree un workflow que recibe de todos los canales y enruta:

```javascript
// Function node: Router Central
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

## 8. Tratamiento de errores

### Timeout de Dify
```
Lo siento, estoy teniendo dificultades. ¿Puede reformular?
O escriba "agente" para hablar con un humano.
```

### Sesión expirada en Typebot
```javascript
if (response.status === 404) {
  const newSession = await startNewSession(typebotId);
  await saveSessionId(userId, newSession.sessionId);
  // Reenviar el mensaje
}
```

## 9. Logs y métricas

Log cada interacción via webhook de n8n:

```javascript
await $http.post('https://n8n.meuservidor.com/webhook/log', {
  timestamp: new Date(),
  channel: $json.channel,
  usuario: $json.usuario_id,
  mensagem: $json.text,
  resposta: $json.reply,
  tempo_ms: responseTime,
  rota: 'dify' | 'chatwoot' | 'typebot'
});
```

### Métricas para monitorear
- Volumen por canal
- Tasa de resolución automática vs. humana
- Tiempo medio de respuesta
- Horarios de pico
- Palabras clave más frecuentes

## 10. Estructura completa de Typebot

```
[1] VARIABLES: canal, usuario_id, nome, email_cliente

[2] SWITCH: canal
    -> telegram: "¡Hola {nome}! Bienvenido a Telegram"
    -> whatsapp: "¡Hola {nome}! Bienvenido a WhatsApp"
    -> web: "¡Hola! ¿Cómo ayudar?"
    -> email: "Hemos recibido su email."
    -> defecto: "¡Hola {nome}!"

[3] CHOICE: "Elija una opción"
    -> "Preguntas Frecuentes"
         [4] HTTP -> Dify
         [5] TEXT -> respuesta
         [6] CHOICE -> "¿Algo más?"
               -> Sí: vuelve [3]
               -> No: finaliza
    -> "Hablar con Agente"
         [7] HTTP -> Chatwoot
         [8] TEXT: "Transfiriendo..."
    -> "Agendar Servicio"
         [9] TEXT: "¿Qué servicio?"
         [10] DATE: "¿Qué fecha?"
         [11] TEXT: "¿Qué horario?"
         [12] HTTP -> n8n
         [13] TEXT: "¡Confirmado!"
    -> "Finalizar"
         [14] TEXT: "¡Gracias!"
```

## 11. Checklist de implementación

- [ ] Typebot con flujo multicanal configurado
- [ ] n8n con workflows: Telegram, Facebook, Email
- [ ] evolution-go conectado a Typebot
- [ ] Dify con base de conocimiento cargada
- [ ] Chatwoot con inbox de Evolution configurado
- [ ] MinIO funcionando
- [ ] Cloudflare Tunnel con dominios:
  - `bot.meuservidor.com` (Typebot)
  - `n8n.meuservidor.com` (n8n)
  - `ia.meuservidor.com` (Dify)
  - `atendimento.meuservidor.com` (Chatwoot)
- [ ] Webhook de Telegram registrado
- [ ] Webhook de Meta configurado
- [ ] Email IMAP/SMTP probado

## Próximo paso

[Seguridad](../07-seguranca.md) - Proteja todo el ecosistema.
