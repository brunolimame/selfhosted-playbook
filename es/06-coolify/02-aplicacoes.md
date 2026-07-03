# 06-02 - Primer Deploy

## 1. Conectar un repositorio Git

1. En el panel de Coolify, vaya a **Sources**
2. Haga clic en **Add Source**
3. Seleccione su plataforma: **GitHub**, **GitLab** o **Bitbucket**
4. Siga el flujo de autenticación (OAuth)
5. Seleccione los repositorios que desea disponibilizar

## 2. Deploy de una aplicación simple (Node.js)

### Ejemplo: API Hello World

Cree un repositorio en GitHub con este contenido:

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

### Crear el deploy en Coolify

1. Vaya a **Applications > Add Application**
2. Seleccione el repositorio creado
3. Nombre: `hello-world`
4. Build pack: `Node.js`
5. Puerto: `3000`
6. **Deploy**

Coolify va a:
1. Clonar el repositorio
2. Instalar dependencias
3. Buildear la aplicación
4. Iniciar el contenedor

## 3. Configurar dominio para la aplicación

Después del deploy, vaya a **Applications > hello-world > Domains**:

1. Haga clic en **Add Domain**
2. Escriba `hello.vm.meuservidor.com`
3. Haga clic en **Save**

Si Cloudflare Tunnel está configurado, necesita agregar este dominio al túnel:

```bash
# En la VM, edite el config.yml de cloudflared
nano ~/.cloudflared/config.yml
```

Agregue:

```yaml
  - hostname: hello.vm.meuservidor.com
    service: http://localhost:3000
```

Reinicie cloudflared:

```bash
sudo systemctl restart cloudflared
```

Acceda a: `https://hello.vm.meuservidor.com`

## 4. Deploy con Docker Compose

Coolify también soporta deploy vía Docker Compose directamente desde un repositorio.

Cree un archivo `docker-compose.yml` en el repositorio:

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

En Coolify, seleccione **Docker Compose** como build pack.

## 5. Variables de entorno

Para agregar variables de entorno:
1. Vaya a **Applications > [app] > Environment Variables**
2. Agregue clave/valor
3. Haga clic en **Save & Redeploy**

## 6. Logs

Para ver logs en tiempo real:
1. Vaya a **Applications > [app] > Logs**
2. Seleccione el contenedor
3. Vea logs de build y ejecución

## 7. Deploy mediante Webhook

Coolify genera URLs de webhook para deploy automático:
1. Vaya a **Applications > [app] > Webhooks**
2. Copie la URL `https://vm.meuservidor.com/api/v1/deploy?token=...`
3. Configure en GitHub/GitLab: **Settings > Webhooks** con esa URL
4. Ahora, cada push hace deploy automático

## 8. Ejemplos de aplicaciones soportadas

| Tipo | Build Pack | Puerto típico |
|------|-----------|---------------|
| Node.js / Express | Node.js | 3000 |
| Python / Flask | Python | 5000 |
| PHP / Laravel | PHP | 80 |
| React / Vite | Node.js (static) | 80 |
| Next.js | Node.js | 3000 |
| Dockerfile | Dockerfile | definido en el Dockerfile |
| Docker Compose | Docker Compose | definido en el compose |

## Próximo paso

[Seguridad](../07-seguranca.md) — Proteja su servidor.
