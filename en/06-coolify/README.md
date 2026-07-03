# 06 - Coolify

## What is Coolify?

[Coolify](https://coolify.io) is an open-source application management platform (self-hosted PaaS). Similar to Vercel, Netlify, or Heroku, but you control everything.

With Coolify you can:
- Deploy applications from Git repositories (GitHub, GitLab, Bitbucket)
- Manage databases (PostgreSQL, MySQL, Redis, MongoDB)
- Host static sites, APIs, Node.js, Python, PHP applications, etc.
- Configure domains and SSL automatically
- Manage environment variables
- View logs in real-time
- Deploy with one click

## Architecture with Coolify

```
Coolify (Docker container)
    |
    +-- Application 1 (Docker container)
    |      Internal port: 3000
    |      Domain: app1.meuservidor.com
    |
    +-- Application 2 (Docker container)
    |      Internal port: 8000
    |      Domain: app2.meuservidor.com
    |
    +-- Database (Docker container)
           Internal port: 5432
```

## Navigation

| Page | Description |
|--------|-----------|
| [01 - Installation](./01-installation.md) | Install Docker and Coolify on the VM |
| [02 - Applications](./02-applications.md) | First deploy, configurations |

## Next step

[Install Coolify](./01-installation.md)
