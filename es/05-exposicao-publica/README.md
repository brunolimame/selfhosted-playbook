# 05 - Exposición Pública

Esta sección cubre diferentes métodos para exponer su VM en Internet, permitiendo el acceso a Coolify y sus aplicaciones mediante dominio.

## Comparación de los métodos

| Método | ¿Abrir puertos? | ¿IP fija? | Seguridad | Costo | Complejidad |
|--------|-----------------|-----------|-----------|-------|-------------|
| [Cloudflare Tunnel](./01-cloudflare-tunnel.md) | No | No | Alta | Gratuito | Baja |
| [DDNS + Port Forwarding](./02-ddns-port-forwarding.md) | Sí | Dinámico (DDNS) | Media | Gratuito | Media |
| [Tailscale Funnel](./03-tailscale-funnel.md) | No | No | Alta | Gratuito (hasta 3 users) | Baja |
| [ngrok](./04-ngrok.md) | No | No | Alta | Gratuito (limitado) | Baja |

## Criterios de elección

- **¿No quiere abrir puertos en el router?** Use Cloudflare Tunnel, Tailscale Funnel o ngrok
- **¿Quiere máxima seguridad?** Cloudflare Tunnel o Tailscale Funnel
- **¿Quiere solución 100% gratuita sin límites?** Cloudflare Tunnel (gratuito) o DDNS
- **¿Necesita acceso solo para usted y su equipo?** Tailscale Funnel
- **¿Solución más simple?** ngrok (pero con límites en el gratis)

## Recomendación

**Cloudflare Tunnel** es el método recomendado para este proyecto porque:
- No requiere abrir puertos en el router
- Funciona con IP dinámica (conexión outbound)
- SSL/TLS automático vía Cloudflare
- Gratuito sin límites de datos (excepto video streaming en el plan Free)
- Se integra perfectamente con el Cloudflare DNS ya configurado

## Próximo paso

Elija un método:
- [Cloudflare Tunnel](./01-cloudflare-tunnel.md) (recomendado)
- [DDNS + Port Forwarding](./02-ddns-port-forwarding.md)
- [Tailscale Funnel](./03-tailscale-funnel.md)
- [ngrok](./04-ngrok.md)
