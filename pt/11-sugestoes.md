# 11 - Sugestoes de Proximas Aplicacoes

Com base no ecossistema ja documentado, aqui estao sugestoes do que adicionar em seguida, organizadas por categoria.

## Monitoramento e Observabilidade

### Uptime Kuma
Monitor de uptime com dashboard bonito, notificacoes e status page publica.

```
docker compose:
  - port: 3001
  - image: louislam/uptime-kuma
  - depende de: nada
```

Por que adicionar: Monitora todas as aplicacoes da VM (Coolify, Typebot, n8n, evolution-go) e envia alerta se algo cair.

### Netdata
Monitoramento em tempo real de CPU, RAM, disco, rede de toda a VM.

```
docker compose:
  - port: 19999
  - image: netdata/netdata
  - depende de: nada
```

Por que adicionar: Visibilidade completa dos recursos da VM, essencial para saber se esta faltando RAM ou CPU.

### Sentry (self-hosted)
Rastreamento de erros em aplicacoes. Capture excecoes do Typebot, n8n, Dify.

```
docker compose:
  - port: 9000
  - image: getsentry/sentry
  - depende de: postgres, redis
```

Por que adicionar: Saber quando uma aplicacao quebrou e por que, com stack trace completo.

## Banco de Dados

### pgAdmin
Interface web para administrar PostgreSQL. Util para todas as apps que usam Postgres.

```
docker compose:
  - port: 5050
  - image: dpage/pgadmin4
```

### Redis Commander
Interface web para administrar Redis (usado por Dify, Chatwoot, evolution-api).

```
docker compose:
  - port: 8081
  - image: rediscommander/redis-commander
```

## CI/CD e Git

### Gitea / Forgejo
Servidor Git auto-hospedado (como GitHub). Permite hospedar repositorios privados e integra com o Coolify.

```
docker compose:
  - port: 3000
  - image: gitea/gitea
  - depende de: postgres
```

Por que adicionar: Hospedar os repositorios das aplicacoes localmente, integrando com Coolify para deploy automatico.

### Woodpecker CI
Pipeline CI/CD leve que integra com Gitea. Roda testes e deploy automaticamente.

```
docker compose:
  - port: 8000
  - image: woodpeckerci/woodpecker-server
  - depende de: postgres, gitea
```

## Comunicacao

### Mattermost
Chat de equipe auto-hospedado (alternativa ao Slack). Integra com n8n para notificacoes.

```
docker compose:
  - port: 8065
  - image: mattermost/mattermost
  - depende de: postgres
```

## DNS e Proxy

### AdGuard Home
Bloqueio de propagandas e rastreadores em nivel de DNS para toda a rede.

```
docker compose:
  - port: 80/3000
  - image: adguard/adguardhome
```

### Nginx Proxy Manager
Interface web para gerenciar proxies reversos e certificados SSL. Alternativa mais simples ao Cloudflare Tunnel.

```
docker compose:
  - port: 80/81/443
  - image: jc21/nginx-proxy-manager
  - depende de: nada
```

## Backup

### Duplicati
Backup automatico com criptografia para nuvem (Google Drive, S3, etc.).

```
docker compose:
  - port: 8200
  - image: linuxserver/duplicati
```

### BorgBackup + Borgmatic
Backup eficiente com deduplicacao e compressao.

```bash
sudo apt install borgmatic
```

## Resumo por prioridade

| Prioridade | App | Motivo |
|-----------|-----|--------|
| Alta | **Uptime Kuma** | Saber se os servicos estao no ar |
| Alta | **Netdata** | Monitorar recursos da VM |
| Media | **Gitea** | Hospedar repositorios localmente |
| Media | **pgAdmin** | Gerenciar bancos PostgreSQL |
| Baixa | **AdGuard Home** | Bloquear propagandas na rede |
| Baixa | **Duplicati** | Backup para nuvem |

## Proximo passo

[Ferramentas de Desenvolvimento](./ferramentas-dev.md) - Ferramentas e servicos uteis para desenvolvimento.
