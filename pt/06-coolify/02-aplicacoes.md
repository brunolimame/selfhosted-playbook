# 06-02 - Primeiro Deploy

## 1. Conectar um repositorio Git

1. No painel do Coolify, va em **Sources**
2. Clique em **Add Source**
3. Selecione sua plataforma: **GitHub**, **GitLab** ou **Bitbucket**
4. Siga o fluxo de autenticacao (OAuth)
5. Selecione os repositorios que deseja disponibilizar

## 2. Deploy de uma aplicacao simples (Node.js)

### Exemplo: API Hello World

Crie um repositorio no GitHub com este conteudo:

```javascript
// index.js
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.json({ message: 'Hello from Coolify!', host: req.hostname });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

```json
{
  "name": "hello-coolify",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.18.0"
  }
}
```

### Criar o deploy no Coolify

1. Va em **Applications > Add Application**
2. Selecione o repositorio criado
3. Nome: `hello-world`
4. Build pack: `Node.js`
5. Porta: `3000`
6. **Deploy**

O Coolify vai:
1. Clonar o repositorio
2. Instalar dependencias
3. Buildar a aplicacao
4. Iniciar o container

## 3. Configurar dominio para a aplicacao

Apos o deploy, va em **Applications > hello-world > Domains**:

1. Clique em **Add Domain**
2. Digite `hello.vm.meuservidor.com`
3. Clique em **Save**

Se o Cloudflare Tunnel estiver configurado, voce precisa adicionar este dominio ao tunel:

```bash
# Na VM, edite o config.yml do cloudflared
nano ~/.cloudflared/config.yml
```

Adicione:

```yaml
  - hostname: hello.vm.meuservidor.com
    service: http://localhost:3000
```

Reinicie o cloudflared:

```bash
sudo systemctl restart cloudflared
```

Acesse: `https://hello.vm.meuservidor.com`

## 4. Deploy com Docker Compose

O Coolify tambem suporta deploy via Docker Compose diretamente de um repositorio.

Crie um arquivo `docker-compose.yml` no repositorio:

```yaml
version: '3.8'
services:
  app:
    image: nginx:alpine
    ports:
      - "80"
    environment:
      - NGINX_HOST=hello.vm.meuservidor.com
```

No Coolify, selecione **Docker Compose** como build pack.

## 5. Variaveis de ambiente

Para adicionar variaveis de ambiente:
1. Va em **Applications > [app] > Environment Variables**
2. Adicione chave/valor
3. Clique em **Save & Redeploy**

## 6. Logs

Para ver logs em tempo real:
1. Va em **Applications > [app] > Logs**
2. Selecione o container
3. Veja logs de build e execucao

## 7. Deploy via Webhook

O Coolify gera URLs de webhook para deploy automatico:
1. Va em **Applications > [app] > Webhooks**
2. Copie a URL `https://vm.meuservidor.com/api/v1/deploy?token=...`
3. Configure no GitHub/GitLab: **Settings > Webhooks** com essa URL
4. Agora, cada push faz deploy automatico

## 8. Exemplos de aplicacoes suportadas

| Tipo | Build Pack | Porta tipica |
|------|-----------|--------------|
| Node.js / Express | Node.js | 3000 |
| Python / Flask | Python | 5000 |
| PHP / Laravel | PHP | 80 |
| React / Vite | Node.js (static) | 80 |
| Next.js | Node.js | 3000 |
| Dockerfile | Dockerfile | definida no Dockerfile |
| Docker Compose | Docker Compose | definida no compose |

## Proximo passo

[Seguranca](../07-seguranca.md) - Proteja seu servidor.
