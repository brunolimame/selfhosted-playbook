# 01 - Introducao

## Objetivo

Criar um servidor caseiro acessivel publicamente, rodando aplicacoes gerenciadas pelo Coolify, sem depender de IP fixo ou servicos cloud caros.

## Arquitetura

```
+------------------+       +------------------+       +------------------+
|  Usuario final   | ----> |  vm.doc.local    | ----> |  Cloudflare DNS  |
|  (navegador)     |       |  (HTTPS)         |       |  + Edge Network  |
+------------------+       +------------------+       +------------------+
                                                               |
                                                    (tunel seguro)
                                                               |
                                                    +----------+--------+
                                                    |  cloudflared     |
                                                    |  (agente local)  |
                                                    +----------+--------+
                                                               |
                                                    +----------+--------+
                                                    |  VM Ubuntu Server |
                                                    |  +------------+   |
                                                    |  | Coolify    |   |
                                                    |  | (porta 8000)|  |
                                                    |  +------------+   |
                                                    |  | Docker     |   |
                                                    |  +------------+   |
                                                    +-------------------+
```

## Fluxo de requisicoes

1. Usuario acessa `https://vm.doc.local`
2. DNS do Cloudflare resolve para o edge do Cloudflare
3. Cloudflare encaminha a requisicao pelo tunel seguro ate o `cloudflared`
4. `cloudflared` entrega a requisicao ao Coolify (porta 8000)
5. Coolify gerencia o roteamento para a aplicacao correta
6. A resposta segue o caminho inverso

## Por que esta abordagem?

| Caracteristica | Beneficio |
|----------------|-----------|
| Sem IP fixo | Cloudflare Tunnel cria conexao outbound, funciona com IP dinamico |
| Sem abrir portas | Nenhuma porta precisa ser exposta no roteador |
| SSL/TLS automatico | Cloudflare fornece certificado HTTPS gratuito |
| Custo baixo | Apenas o custo do dominio (Cloudflare Tunnel e gratuito) |
| Coolify | Interface Web para gerenciar aplicacoes sem complicacao |

## Proximo passo

[Pre-requisitos](./02-pre-requisitos.md) - Verifique o que voce precisa antes de comecar.
