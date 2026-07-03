# 05 - Public Exposure

This section covers different methods for exposing your VM to the internet, allowing access to Coolify and its applications via domain.

## Method Comparison

| Method | Open ports? | Fixed IP? | Security | Cost | Complexity |
|--------|--------------|----------|-----------|-------|--------------|
| [Cloudflare Tunnel](./01-cloudflare-tunnel.md) | No | No | High | Free | Low |
| [DDNS + Port Forwarding](./02-ddns-port-forwarding.md) | Yes | Dynamic (DDNS) | Medium | Free | Medium |
| [Tailscale Funnel](./03-tailscale-funnel.md) | No | No | High | Free (up to 3 users) | Low |
| [ngrok](./04-ngrok.md) | No | No | High | Free (limited) | Low |

## Selection Criteria

- **Don't want to open ports on the router?** Use Cloudflare Tunnel, Tailscale Funnel or ngrok
- **Want maximum security?** Cloudflare Tunnel or Tailscale Funnel
- **Want a 100% free solution without limits?** Cloudflare Tunnel (free) or DDNS
- **Need access only for you and your team?** Tailscale Funnel
- **Simplest solution?** ngrok (but with limits on the free tier)

## Recommendation

**Cloudflare Tunnel** is the recommended method for this project because:
- No need to open ports on the router
- Works with dynamic IP (outbound connection)
- Automatic SSL/TLS via Cloudflare
- Free with no data limits (except video streaming on the Free plan)
- Integrates perfectly with the already configured Cloudflare DNS

## Next step

Choose a method:
- [Cloudflare Tunnel](./01-cloudflare-tunnel.md) (recommended)
- [DDNS + Port Forwarding](./02-ddns-port-forwarding.md)
- [Tailscale Funnel](./03-tailscale-funnel.md)
- [ngrok](./04-ngrok.md)
