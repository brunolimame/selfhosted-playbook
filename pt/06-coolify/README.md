# 06 - Coolify

## O que e o Coolify?

[Coolify](https://coolify.io) e uma plataforma open-source de gerenciamento de aplicacoes (PaaS auto-hospedada). Similar ao Vercel, Netlify ou Heroku, mas voce controla tudo.

Com o Coolify voce pode:
- Fazer deploy de aplicacoes a partir de repositorios Git (GitHub, GitLab, Bitbucket)
- Gerenciar bancos de dados (PostgreSQL, MySQL, Redis, MongoDB)
- Hospedar sites estaticos, APIs, aplicacoes Node.js, Python, PHP, etc.
- Configurar dominios e SSL automaticamente
- Gerenciar variaveis de ambiente
- Ver logs em tempo real
- Fazer deploy com um clique

## Arquitetura com Coolify

```
Coolify (Docker container)
    |
    +-- Aplicacao 1 (Docker container)
    |      Porta interna: 3000
    |      Dominio: app1.meuservidor.com
    |
    +-- Aplicacao 2 (Docker container)
    |      Porta interna: 8000
    |      Dominio: app2.meuservidor.com
    |
    +-- Banco de Dados (Docker container)
           Porta interna: 5432
```

## Navegacao

| Pagina | Descricao |
|--------|-----------|
| [01 - Instalacao](./01-instalacao.md) | Instalar Docker e Coolify na VM |
| [02 - Aplicacoes](./02-aplicacoes.md) | Primeiro deploy, configuracoes |

## Proximo passo

[Instalar o Coolify](./01-instalacao.md)
