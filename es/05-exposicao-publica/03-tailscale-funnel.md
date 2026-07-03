# 05-03 - Tailscale Funnel

## Visión general

[Tailscale](https://tailscale.com) crea una red privada (WireGuard) entre sus dispositivos. **Funnel** es un recurso que permite exponer servicios de su red Tailscale públicamente en Internet, usando un subdominio `*.ts.net`.

```
Internet -> vm.meuservidor.com -> Cloudflare DNS
                                        |
                            Tailscale Funnel (443)
                                        |
                               VM (Tailscale)
                                        |
                               Coolify (:8000)
```

**Diferencia del Cloudflare Tunnel**: Con Tailscale Funnel, los usuarios necesitan acceder mediante un subdominio `*.ts.net` (a menos que configure su dominio para apuntar hacia allí).

## 1. Instalar Tailscale en la VM

Acceda a la VM vía SSH:

```bash
ssh ubuntu@192.168.1.100
```

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

## 2. Autenticarse

```bash
sudo tailscale up
```

Esto mostrará una URL. Cópiela y ábrala en el navegador, inicie sesión con su cuenta Google/Microsoft/GitHub/Apple.

Después de autenticarse, verifique la IP Tailscale de la VM:

```bash
tailscale ip -4
```

Ejemplo: `100.x.x.x`

## 3. Instalar Tailscale en su host (opcional)

Para acceder a la VM por la red Tailscale (sin depender de la IP local):

- **Windows**: Descargue de https://tailscale.com/download
- **Linux**: `curl -fsSL https://tailscale.com/install.sh | sh`
- **macOS**: Descargue de la App Store o sitio web

Ingrese a la misma cuenta y ejecute `tailscale up`.

Ahora puede acceder a la VM vía IP Tailscale:
```bash
ssh ubuntu@100.x.x.x
```

## 4. Habilitar Funnel

Funnel expone un servicio local públicamente mediante `https://<nombre-de-la-maquina>.<sudominio>.ts.net`.

```bash
sudo tailscale funnel --bg 8000
```

Esto expone el puerto 8000 (Coolify) públicamente en:
```
https://ubuntu-vm.ts.net
```

Para usar un nombre personalizado (antes de `ts.net`):
```bash
sudo tailscale funnel --bg 8000 --set-path=/
```

## 5. Usar dominio propio con Tailscale Funnel

Para usar `vm.meuservidor.com` con Tailscale Funnel, necesita un proxy inverso adicional o usar **Tailscale Serve** con HTTPS personalizado.

### Opción: CNAME de Cloudflare a Tailscale

En Cloudflare DNS, cree un registro CNAME:

| Tipo | Nombre | Contenido |
|------|--------|-----------|
| CNAME | `vm` | `ubuntu-vm.ts.net` |

Con proxy **DNS Only** (naranja apagada).

Esto hace que `vm.meuservidor.com` apunte a `ubuntu-vm.ts.net`.

> **Nota**: Cloudflare Tunnel (método recomendado) es más flexible para dominios propios.

## 6. Gestionar Funnel

```bash
# Ver estado
tailscale funnel status

# Detener el funnel
tailscale funnel off
```

## Ventajas de Tailscale Funnel

- Configuración muy simple (2 comandos)
- Cifrado punta-a-punta (WireGuard + HTTPS)
- No requiere abrir puertos
- Acceso a red privada entre dispositivos
- Gratuito hasta 3 usuarios

## Limitaciones

- URL pública es `*.ts.net` (o necesita configurar CNAME)
- Depende de la infraestructura Tailscale
- Límite de 3 usuarios en el plan gratuito
- Tailscale es una capa adicional entre el usuario y el servicio

## Próximo paso

[Coolify - Instalación](../06-coolify/README.md)
