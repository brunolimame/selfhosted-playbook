# 09-12 - Netdata

## O que e?

[Netdata](https://www.netdata.cloud) e um sistema de monitoramento em **tempo real** que coleta milhares de metricas por segundo e exibe em graficos interativos. Zero configuracao: ao iniciar, ele ja detecta e monitora automaticamente todos os servicos e containers.

Funcionalidades principais:
- Metricas a cada 1 segundo (nao a cada 60s como ferramentas tradicionais)
- Auto-descoberta de servicos: PostgreSQL, Redis, Nginx, Docker, e 200+ outros
- Dashboard web interativo com graficos zoom e arrastaveis
- Alertas inteligentes configurados automaticamente
- Consumo leve: ~100-200 MB RAM, ~1% CPU
- Suporte a metricas historicas (configuravel)

## Por que Netdata neste projeto?

O Uptime Kuma diz **se** o servico esta no ar. O Netdata diz **por que** ele caiu:

| Cenario | Uptime Kuma | Netdata |
|---------|-------------|---------|
| "Site fora do ar" | Alerta: down | - |
| "RAM acabou" | - | Grafico de memoria mostrando o pico |
| "CPU a 100%" | - | Qual processo consumiu |
| "Disco cheio" | - | Exatamente qual diretorio |
| "Porta fechou" | Alerta: timeout | Logs do sistema |

## Arquitetura

```
netdata (container)
    Porta: 19999 (dashboard)
    |
    +-- Acessa /proc, /sys do host (read-only)
    +-- Acessa /var/run/docker.sock (containers)
    +-- Monitora automaticamente: CPU, RAM, disco, rede, Docker, PostgreSQL, Redis...
```

## Pre-requisitos

- Docker instalado na VM
- No minimo 512 MB RAM livres (Netdata usa ~100-200 MB)
- Privilegios especiais no container para acessar metricas do sistema

## Instalacao

### 1. Acessar a VM

```bash
ssh ubuntu@192.168.1.100
```

### 2. Criar diretorio

```bash
mkdir -p ~/netdata
cd ~/netdata
```

### 3. Criar docker-compose.yml

```bash
nano docker-compose.yml
```

```yaml
services:
  netdata:
    image: netdata/netdata:stable
    container_name: netdata
    hostname: ubuntu-vm
    restart: unless-stopped
    pid: host
    network_mode: host
    cap_add:
      - SYS_PTRACE
      - SYS_ADMIN
    security_opt:
      - apparmor:unconfined
    volumes:
      - netdata_config:/etc/netdata
      - netdata_lib:/var/lib/netdata
      - netdata_cache:/var/cache/netdata
      - /:/host/root:ro,rslave
      - /etc/passwd:/host/etc/passwd:ro
      - /etc/group:/host/etc/group:ro
      - /etc/localtime:/etc/localtime:ro
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /etc/os-release:/host/etc/os-release:ro
      - /var/log:/host/var/log:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro

volumes:
  netdata_config:
  netdata_lib:
  netdata_cache:
```

> **Nota**: `network_mode: host` faz o Netdata usar a porta 19999 diretamente no IP da VM, sem mapeamento de porta.

### 4. Iniciar

```bash
docker compose up -d
```

### 5. Verificar

```bash
docker compose ps
docker compose logs -f
```

## Acessar

- **Local**: `http://192.168.1.100:19999`
- Nao precisa de login - o dashboard abre direto com todas as metricas.

## Configurar dominio no Cloudflare Tunnel

```bash
nano ~/.cloudflared/config.yml
```

```yaml
  - hostname: netdata.meuservidor.com
    service: http://localhost:19999
```

```bash
sudo systemctl restart cloudflared
```

## O que o Netdata monitora automaticamente

### Sistema
- CPU (por nucleo, por processo)
- RAM (usada, buffer, cache, swap)
- Disco (I/O, uso, inodes)
- Rede (interface por interface, protocolos)
- Processos (top 10 por CPU/RAM)

### Docker
- Cada container: CPU, RAM, rede, disco
- Logs centralizados

### Servicos (detectados automaticamente)
- **PostgreSQL**: queries, conexoes, locks, cache hit ratio
- **Redis**: hits, misses, memoria, conexoes
- **Nginx**: requisicoes, conexoes, erros (se tiver)
- **MySQL/MariaDB**: queries, threads, buffer pool

## Alertas uteis pre-configurados

O Netdata ja vem com alertas inteligentes:

| Alerta | Gatilho | Acao sugerida |
|--------|---------|---------------|
| RAM > 80% | Uso de memoria alto | Verificar containers com maior consumo |
| CPU > 90% | Processador sobrecarregado | Identificar processo no dashboard |
| Disco > 85% | Quase cheio | Rodar `docker system prune` |
| Swap > 50% | RAM insuficiente | Aumentar RAM da VM |
| PostgreSQL connections > 100 | Muitas conexoes | Verificar pooling ou app com leak |

Para configurar notificacoes:
1. Acesse `http://192.168.1.100:19999`
2. Clique em **Alertas > Notificacoes**
3. Adicione Telegram, Discord, Email ou Slack

## Personalizar retencao de dados

Por padrao o Netdata armazena ~2 horas de metricas em memoria. Para aumentar:

```bash
nano docker-compose.yml
```

Adicione no `environment`:

```yaml
    environment:
      - NETDATA_PAGE_CACHE_SIZE=32
      - NETDATA_DBENGINE_SIZE=256
```

Isso aumenta a retencao para varios dias.

Ou edite o arquivo de configuracao:

```bash
docker compose exec netdata /etc/netdata/edit-config netdata.conf
```

```ini
[global]
    page cache size = 32
    dbengine multihost disk space = 256
```

## Integracoes com outras ferramentas

### Grafana (se tiver)
Netdata pode exportar metricas para Prometheus, que o Grafana consome:

```bash
# No arquivo de configuracao
docker compose exec netdata /etc/netdata/edit-config go.d/prometheus.conf
```

### n8n
O n8n pode consultar a API do Netdata para tomar decisoes:

```bash
# Obter uso atual de CPU
curl -s http://192.168.1.100:19999/api/v1/data?chart=system.cpu | jq '.result[0].value[1]'
```

## Manutencao

### Atualizar

```bash
cd ~/netdata
docker compose pull
docker compose up -d
```

### Verificar espaco usado

```bash
docker run --rm -v netdata_cache:/source alpine du -sh /source
```

### Logs

```bash
docker compose logs -f --tail 100
```

## Troubleshooting

| Erro | Causa | Solucao |
|------|-------|---------|
| Dashboard nao carrega | Porta nao acessivel | `docker compose ps` para ver se subiu |
| "Permission denied" | Faltam privilegios | Verificar `cap_add` e `security_opt` |
| Metricas do Docker vazias | Socket nao montado | Verificar `/var/run/docker.sock` no volumes |
| Consumo alto de RAM | DBENGINE muito grande | Reduzir `dbengine multihost disk space` |
| Graficos sem dados | Acabou de iniciar | Aguardar 30s para primeira coleta |

## Proximo passo

Voltar para [Indice de Aplicacoes](./README.md).
