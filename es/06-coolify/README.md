# 06 - Coolify

## ¿Qué es Coolify?

[Coolify](https://coolify.io) es una plataforma open-source de gestión de aplicaciones (PaaS auto-alojada). Similar a Vercel, Netlify o Heroku, pero usted controla todo.

Con Coolify puede:
- Hacer deploy de aplicaciones desde repositorios Git (GitHub, GitLab, Bitbucket)
- Gestionar bases de datos (PostgreSQL, MySQL, Redis, MongoDB)
- Alojar sitios estáticos, APIs, aplicaciones Node.js, Python, PHP, etc.
- Configurar dominios y SSL automáticamente
- Gestionar variables de entorno
- Ver logs en tiempo real
- Hacer deploy con un clic

## Arquitectura con Coolify

```
Coolify (Docker container)
    |
    +-- Aplicación 1 (Docker container)
    |      Puerto interno: 3000
    |      Dominio: app1.meuservidor.com
    |
    +-- Aplicación 2 (Docker container)
    |      Puerto interno: 8000
    |      Dominio: app2.meuservidor.com
    |
    +-- Base de Datos (Docker container)
           Puerto interno: 5432
```

## Navegación

| Página | Descripción |
|--------|-------------|
| [01 - Instalación](./01-instalacao.md) | Instalar Docker y Coolify en la VM |
| [02 - Aplicaciones](./02-aplicacoes.md) | Primer deploy, configuraciones |

## Próximo paso

[Instalar Coolify](./01-instalacao.md)
