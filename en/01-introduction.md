# 01 - Introduction

## Goal

Create a publicly accessible home server, running applications managed by Coolify, without relying on a fixed IP or expensive cloud services.

## Architecture

```
+------------------+       +------------------+       +------------------+
|  End user        | ----> |  vm.doc.local    | ----> |  Cloudflare DNS  |
|  (browser)       |       |  (HTTPS)         |       |  + Edge Network  |
+------------------+       +------------------+       +------------------+
                                                               |
                                                     (secure tunnel)
                                                               |
                                                     +----------+--------+
                                                     |  cloudflared     |
                                                     |  (local agent)   |
                                                     +----------+--------+
                                                               |
                                                     +----------+--------+
                                                     |  VM Ubuntu Server |
                                                     |  +------------+   |
                                                     |  | Coolify    |   |
                                                     |  | (port 8000)|   |
                                                     |  +------------+   |
                                                     |  | Docker     |   |
                                                     |  +------------+   |
                                                     +-------------------+
```

## Request flow

1. User accesses `https://vm.doc.local`
2. Cloudflare DNS resolves to Cloudflare edge
3. Cloudflare forwards the request through the secure tunnel to `cloudflared`
4. `cloudflared` delivers the request to Coolify (port 8000)
5. Coolify manages routing to the correct application
6. The response follows the reverse path

## Why this approach?

| Feature | Benefit |
|---------|---------|
| No fixed IP | Cloudflare Tunnel creates outbound connection, works with dynamic IP |
| No port forwarding | No ports need to be exposed on the router |
| Automatic SSL/TLS | Cloudflare provides free HTTPS certificate |
| Low cost | Only the domain cost (Cloudflare Tunnel is free) |
| Coolify | Web interface to manage applications without hassle |

## Next step

[Prerequisites](./02-prerequisites.md) - Check what you need before starting.
