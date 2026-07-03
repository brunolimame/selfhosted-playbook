# 05 - Exposicao Publica

Esta secao cobre diferentes metodos para expor sua VM na internet, permitindo acesso ao Coolify e suas aplicacoes via dominio.

## Comparacao dos metodos

| Metodo | Abrir portas? | IP fixo? | Seguranca | Custo | Complexidade |
|--------|--------------|----------|-----------|-------|--------------|
| [Cloudflare Tunnel](./01-cloudflare-tunnel.md) | Nao | Nao | Alta | Gratuito | Baixa |
| [DDNS + Port Forwarding](./02-ddns-port-forwarding.md) | Sim | Dinamico (DDNS) | Media | Gratuito | Media |
| [Tailscale Funnel](./03-tailscale-funnel.md) | Nao | Nao | Alta | Gratuito (ate 3 users) | Baixa |
| [ngrok](./04-ngrok.md) | Nao | Nao | Alta | Gratuito (limitado) | Baixa |

## Criterios de escolha

- **Nao quer abrir portas no roteador?** Use Cloudflare Tunnel, Tailscale Funnel ou ngrok
- **Quer seguranca maxima?** Cloudflare Tunnel ou Tailscale Funnel
- **Quer solucao 100% gratuita sem limites?** Cloudflare Tunnel (gratuito) ou DDNS
- **Precisa de acesso仅 para voce e equipe?** Tailscale Funnel
- **Solucao mais simples?** ngrok (mas com limites no gratis)

## Recomendacao

**Cloudflare Tunnel** e o metodo recomendado para este projeto porque:
- Nao requer abrir portas no roteador
- Funciona com IP dinamico (conexao outbound)
- SSL/TLS automatico via Cloudflare
- Gratuito sem limites de dados (exceto video streaming no plano Free)
- Integra-se perfeitamente com o Cloudflare DNS ja configurado

## Proximo passo

Escolha um metodo:
- [Cloudflare Tunnel](./01-cloudflare-tunnel.md) (recomendado)
- [DDNS + Port Forwarding](./02-ddns-port-forwarding.md)
- [Tailscale Funnel](./03-tailscale-funnel.md)
- [ngrok](./04-ngrok.md)
