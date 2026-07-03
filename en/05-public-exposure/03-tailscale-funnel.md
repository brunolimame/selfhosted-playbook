# 05-03 - Tailscale Funnel

## Overview

[Tailscale](https://tailscale.com) creates a private network (WireGuard) between your devices. **Funnel** is a feature that allows you to expose services from your Tailscale network publicly on the internet, using a `*.ts.net` subdomain.

```
Internet -> vm.meuservidor.com -> Cloudflare DNS
                                        |
                            Tailscale Funnel (443)
                                        |
                               VM (Tailscale)
                                        |
                               Coolify (:8000)
```

**Difference from Cloudflare Tunnel**: With Tailscale Funnel, users need to access via a `*.ts.net` subdomain (unless you configure your domain to point there).

## 1. Install Tailscale on the VM

Access the VM via SSH:

```bash
ssh ubuntu@192.168.1.100
```

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

## 2. Authenticate

```bash
sudo tailscale up
```

This will display a URL. Copy and open it in your browser, log in with your Google/Microsoft/GitHub/Apple account.

After authenticating, check the VM's Tailscale IP:

```bash
tailscale ip -4
```

Example: `100.x.x.x`

## 3. Install Tailscale on your host (optional)

To access the VM via the Tailscale network (without relying on the local IP):

- **Windows**: Download from https://tailscale.com/download
- **Linux**: `curl -fsSL https://tailscale.com/install.sh | sh`
- **macOS**: Download from the App Store or website

Log into the same account and run `tailscale up`.

Now you can access the VM via Tailscale IP:
```bash
ssh ubuntu@100.x.x.x
```

## 4. Enable Funnel

Funnel exposes a local service publicly via `https://<machine-name>.<yourdomain>.ts.net`.

```bash
sudo tailscale funnel --bg 8000
```

This exposes port 8000 (Coolify) publicly at:
```
https://ubuntu-vm.ts.net
```

To use a custom name (before `ts.net`):
```bash
sudo tailscale funnel --bg 8000 --set-path=/
```

## 5. Use a custom domain with Tailscale Funnel

To use `vm.meuservidor.com` with Tailscale Funnel, you need an additional reverse proxy or use **Tailscale Serve** with custom HTTPS.

### Option: Cloudflare CNAME to Tailscale

In Cloudflare DNS, create a CNAME record:

| Type | Name | Content |
|------|------|----------|
| CNAME | `vm` | `ubuntu-vm.ts.net` |

With proxy **DNS Only** (orange cloud off).

This makes `vm.meuservidor.com` point to `ubuntu-vm.ts.net`.

> **Note**: Cloudflare Tunnel (the recommended method) is more flexible for custom domains.

## 6. Manage Funnel

```bash
# Check status
tailscale funnel status

# Stop the funnel
tailscale funnel off
```

## Advantages of Tailscale Funnel

- Very simple setup (2 commands)
- End-to-end encryption (WireGuard + HTTPS)
- No need to open ports
- Private network access between devices
- Free for up to 3 users

## Limitations

- Public URL is `*.ts.net` (or requires CNAME configuration)
- Depends on Tailscale infrastructure
- 3-user limit on the free plan
- Tailscale is an additional layer between the user and the service

## Next step

[Coolify - Installation](../06-coolify/README.md)
