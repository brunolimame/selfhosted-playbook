# Documentacao - Indice

Bem-vindo a documentacao do projeto Servidor Remoto.

## Navegacao

| # | Secao | Descricao |
|---|-------|-----------|
| 01 | [Introducao](./01-introducao.md) | Visao geral, arquitetura e objetivo |
| 02 | [Pre-requisitos](./02-pre-requisitos.md) | Hardware, software e contas necessarias |
| 03 | [Maquina Virtual (VM)](./03-vm/README.md) | Criacao da VM no VirtualBox com Ubuntu Server |
| 04 | [Dominio e DNS](./04-dominio.md) | Registro de dominio e configuracao Cloudflare DNS |
| 05 | [Exposicao Publica](./05-exposicao-publica/README.md) | Metodos para expor a VM na internet |
| 06 | [Coolify](./06-coolify/README.md) | Instalacao e gerenciamento de aplicacoes |
| 07 | [Seguranca](./07-seguranca.md) | Boas praticas de seguranca |
| 08 | [Manutencao](./08-manutencao.md) | Backup, atualizacoes e monitoramento |
| 09 | [Aplicacoes](./09-aplicacoes/README.md) | Odysseus, n8n, Evolution Go e mais |
| 10 | [Chatbot Multicanal](./10-chatbot-multicanal/README.md) | Bot para WhatsApp, Telegram, Facebook, Web, Email |
| 11 | [Proximas Sugestoes](./11-sugestoes.md) | Aplicacoes recomendadas (monitoria, CI/CD, backup) |
| -- | [Ferramentas Dev](./ferramentas-dev.md) | Webhook inspectors, API clients, tunnels, mocks |

## Convencoes usadas nesta documentacao

- **`vm.doc.local`**: dominio de exemplo. Substitua pelo seu dominio real.
- **`ubuntu-vm`**: hostname da VM. Pode ser alterado durante a instalacao.
- Blocos de comando: assumem que o sistema operacional do host ja foi identificado.

## Sobre

Este projeto documenta passo a passo como transformar um computador local (ou VM) em um servidor publicamente acessivel, gerenciado pelo Coolify, sem custo mensal alem do dominio.
