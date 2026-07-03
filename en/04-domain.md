# 04 - Domain and DNS

## 1. Choose and register a domain

You need your own domain. Examples:

| Type | Example | Where to register |
|------|---------|-------------------|
| .com | `myserver.com` | Namecheap, GoDaddy, Cloudflare Registrar |
| .com.br | `myserver.com.br` | Registro.br |
| .dev | `myserver.dev` | Namecheap, Cloudflare Registrar |

> **Tip**: Registering through Cloudflare Registrar costs at-cost prices (no markup). If you already have a domain at another registrar, you can move it to Cloudflare.

## 2. Create a Cloudflare account

1. Go to https://dash.cloudflare.com/sign-up
2. Create a free account
3. Confirm your email

## 3. Add the domain to Cloudflare

1. After login, click **Add a Site**
2. Enter your domain (ex: `myserver.com`)
3. Select the **Free** plan
4. Cloudflare scans existing DNS records (may take a few seconds)
5. Proceed

## 4. Change nameservers

Cloudflare will display two nameservers like:
```
dns1.ns.cloudflare.com
dns2.ns.cloudflare.com
```

1. Access your domain registrar's panel
2. Find the option to change **Nameservers** (DNS Server)
3. Replace the current nameservers with Cloudflare's
4. Save

Propagation can take from a few minutes to 48 hours (usually < 1 hour).

## 5. Check status

In the Cloudflare dashboard, the status will change from **Pending** to **Active** when the change has propagated.

## 6. Create DNS records (temporary)

While configuring the tunnel, create a temporary DNS record for testing:

In the Cloudflare dashboard, go to **DNS > Records** and add:

**A Record** (for local testing):
| Type | Name | Content | Proxy |
|------|------|---------|-------|
| A | `vm` | `192.168.1.100` | DNS only (disabled) |

This creates `vm.myserver.com` pointing to the VM's local IP.

> **Note**: This record will only work on the local network. The final configuration using Cloudflare Tunnel (or another method) will replace this record.

## 7. Configure SSL/TLS

In the Cloudflare dashboard:
1. Go to **SSL/TLS > Overview**
2. Select **Full (strict)** for maximum security
3. Under **Origin Server > Create Certificate**, generate a self-signed certificate (optional but recommended)

## 8. Wait for DNS propagation

Check if DNS has already propagated:

```bash
# On your host (outside the VM)
nslookup vm.myserver.com
# or
ping vm.myserver.com
```

## Next step

[Public Exposure](./05-public-exposure/README.md) - Choose how to expose your VM on the internet.
