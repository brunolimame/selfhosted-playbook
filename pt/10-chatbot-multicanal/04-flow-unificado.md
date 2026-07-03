# 10-04 - Fluxo Unificado

Agora que cada canal esta configurado, vamos construir o **fluxo central no Typebot** que atende todos os canais com roteamento inteligente entre Typebot, Dify e Chatwoot.

## Visao geral do fluxo

```
INICIO
  |
  +-- Variavel "canal" define o contexto
  |
  v
BOAS-VINDAS (personalizada por canal)
  |
  v
MENU PRINCIPAL
  |-- (1) Duvidas Frequentes --> Dify (RAG)
  |-- (2) Falar com Atendente --> Chatwoot
  |-- (3) Agendar Servico   --> Fluxo de agendamento
  |-- (4) Sair              --> Encerra
  |
  v
AVALIACAO FINAL
```

## 1. Variaveis de entrada

No Typebot, crie no primeiro bloco as variaveis:

| Variavel | Descricao | Exemplo |
|----------|-----------|---------|
| `canal` | Canal de origem | `whatsapp`, `telegram`, `facebook`, `instagram`, `web`, `email` |
| `usuario_id` | ID do usuario no canal | `5511999999999@s.whatsapp.net` |
| `nome` | Nome do usuario | `Joao` |
| `email_cliente` | Email (se disponivel) | `joao@email.com` |

## 2. Boas-vindas condicional por canal

Use um **Switch block**:

```
Caso canal = "telegram":
  "Ola {nome}! Bem-vindo ao atendimento pelo Telegram."

Caso canal = "whatsapp":
  "Ola {nome}! Bem-vindo ao atendimento pelo WhatsApp."

Caso canal = "web":
  "Ola! Como podemos ajudar?"

Caso canal = "email":
  "Recebemos sua mensagem. Responderei em breve."

Caso padrao:
  "Ola {nome}! Seja bem-vindo(a)."
```

## 3. Menu principal

Crie um bloco **Choice**:

```
[1] Duvidas Frequentes -> chama Dify (HTTP Request)
[2] Falar com Atendente -> chama Chatwoot (HTTP Request)
[3] Agendar Servico -> fluxo de agendamento
[4] Encerrar
```

## 4. Integracao com Dify (FAQ inteligente)

**Bloco HTTP Request**:

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

**Bloco Set Variable**:
```
resposta_dify = {{ http_request.answer }}
```

**Bloco Text**:
```
{{ resposta_dify }}

Digite sua proxima pergunta ou escolha:
1. Voltar ao menu
2. Falar com atendente
3. Encerrar
```

## 5. Integracao com Chatwoot (transferencia para humano)

**Bloco HTTP Request** (criar conversa):

```
POST https://atendimento.meuservidor.com/api/v1/accounts/1/conversations
Headers:
  api_access_token: SEU_TOKEN_CHATWOOT
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

**Bloco Text**:
```
Voce sera atendido por um agente em breve. Agradecemos a paciencia!
```

## 6. Fluxo de agendamento (exemplo)

1. **Text**: "Qual servico deseja agendar?"
   - Consulta
   - Suporte Tecnico
   - Outro

2. **Date Input**: "Qual a melhor data?"

3. **Text**: "Qual horario prefere?"

4. **HTTP Request** (n8n):
   ```
   POST https://n8n.meuservidor.com/webhook/agendamento
   Body: { "servico": "...", "data": "...", "cliente": "{{ nome }}" }
   ```

5. **Text**: "Agendamento confirmado! Enviaremos um lembrete."

## 7. Roteador central no n8n

Crie um workflow que recebe de todos os canais e roteia:

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

## 8. Tratamento de erros

### Timeout do Dify
```
Desculpe, estou com dificuldades. Pode reformular?
Ou digite "atendente" para falar com um humano.
```

### Sessao expirada no Typebot
```javascript
if (response.status === 404) {
  const newSession = await startNewSession(typebotId);
  await saveSessionId(userId, newSession.sessionId);
  // Reenviar a mensagem
}
```

## 9. Logs e metricas

Log cada interacao via webhook do n8n:

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

### Metricas para acompanhar
- Volume por canal
- Taxa de resolucao automatica vs. humana
- Tempo medio de resposta
- Horarios de pico
- Palavras-chave mais frequentes

## 10. Estrutura completa do Typebot

```
[1] VARIAVEIS: canal, usuario_id, nome, email_cliente

[2] SWITCH: canal
    -> telegram: "Ola {nome}! Bem-vindo ao Telegram"
    -> whatsapp: "Ola {nome}! Bem-vindo ao WhatsApp"
    -> web: "Ola! Como ajudar?"
    -> email: "Recebemos seu email."
    -> padrao: "Ola {nome}!"

[3] CHOICE: "Escolha uma opcao"
    -> "Duvidas Frequentes"
         [4] HTTP -> Dify
         [5] TEXT -> resposta
         [6] CHOICE -> "Mais algo?"
              -> Sim: volta [3]
              -> Nao: encerra
    -> "Falar com Atendente"
         [7] HTTP -> Chatwoot
         [8] TEXT: "Transferindo..."
    -> "Agendar Servico"
         [9] TEXT: "Qual servico?"
         [10] DATE: "Qual data?"
         [11] TEXT: "Qual horario?"
         [12] HTTP -> n8n
         [13] TEXT: "Confirmado!"
    -> "Encerrar"
         [14] TEXT: "Obrigado!"
```

## 11. Checklist de implantacao

- [ ] Typebot com fluxo multicanal configurado
- [ ] n8n com workflows: Telegram, Facebook, Email
- [ ] evolution-go conectado ao Typebot
- [ ] Dify com base de conhecimento carregada
- [ ] Chatwoot com inbox do Evolution configurado
- [ ] MinIO rodando
- [ ] Cloudflare Tunnel com dominios:
  - `bot.meuservidor.com` (Typebot)
  - `n8n.meuservidor.com` (n8n)
  - `ia.meuservidor.com` (Dify)
  - `atendimento.meuservidor.com` (Chatwoot)
- [ ] Webhook do Telegram registrado
- [ ] Webhook do Meta configurado
- [ ] Email IMAP/SMTP testado

## Proximo passo

[Seguranca](../07-seguranca.md) - Proteja todo o ecossistema.
