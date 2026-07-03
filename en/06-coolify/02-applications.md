# 06-02 - First Deploy

## 1. Connect a Git repository

1. In the Coolify dashboard, go to **Sources**
2. Click **Add Source**
3. Select your platform: **GitHub**, **GitLab**, or **Bitbucket**
4. Follow the authentication flow (OAuth)
5. Select the repositories you want to make available

## 2. Deploy a simple application (Node.js)

### Example: Hello World API

Create a repository on GitHub with this content:

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

### Create the deploy in Coolify

1. Go to **Applications > Add Application**
2. Select the created repository
3. Name: `hello-world`
4. Build pack: `Node.js`
5. Port: `3000`
6. **Deploy**

Coolify will:
1. Clone the repository
2. Install dependencies
3. Build the application
4. Start the container

## 3. Configure domain for the application

After deploy, go to **Applications > hello-world > Domains**:

1. Click **Add Domain**
2. Enter `hello.vm.meuservidor.com`
3. Click **Save**

If Cloudflare Tunnel is configured, you need to add this domain to the tunnel:

```bash
# On the VM, edit cloudflared config.yml
nano ~/.cloudflared/config.yml
```

Add:

```yaml
  - hostname: hello.vm.meuservidor.com
    service: http://localhost:3000
```

Restart cloudflared:

```bash
sudo systemctl restart cloudflared
```

Access: `https://hello.vm.meuservidor.com`

## 4. Deploy with Docker Compose

Coolify also supports deployment via Docker Compose directly from a repository.

Create a `docker-compose.yml` file in the repository:

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

In Coolify, select **Docker Compose** as the build pack.

## 5. Environment variables

To add environment variables:
1. Go to **Applications > [app] > Environment Variables**
2. Add key/value
3. Click **Save & Redeploy**

## 6. Logs

To view logs in real-time:
1. Go to **Applications > [app] > Logs**
2. Select the container
3. View build and runtime logs

## 7. Deploy via Webhook

Coolify generates webhook URLs for automatic deployment:
1. Go to **Applications > [app] > Webhooks**
2. Copy the URL `https://vm.meuservidor.com/api/v1/deploy?token=...`
3. Configure in GitHub/GitLab: **Settings > Webhooks** with this URL
4. Now, every push triggers automatic deployment

## 8. Examples of supported applications

| Type | Build Pack | Typical port |
|------|-----------|--------------|
| Node.js / Express | Node.js | 3000 |
| Python / Flask | Python | 5000 |
| PHP / Laravel | PHP | 80 |
| React / Vite | Node.js (static) | 80 |
| Next.js | Node.js | 3000 |
| Dockerfile | Dockerfile | defined in Dockerfile |
| Docker Compose | Docker Compose | defined in compose |

## Next step

[Security](../07-security.md) - Secure your server.
