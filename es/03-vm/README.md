# 03 - Maquina Virtual (VM)

Guia para crear y configurar la maquina virtual que ejecutara el servidor.

## Elige tu hipervisor

| Hipervisor | Plataforma | Costo | Documentacion |
|------------|-----------|-------|---------------|
| VirtualBox | Windows, Linux, macOS | Gratuito | [Guia](./01-virtualbox.md) |
| VMware Workstation Pro | Windows, Linux | Gratuito (uso personal) | [Guia](./04-vmware.md) |
| Parallels Desktop | macOS (Intel + Apple Silicon) | Pago | [Guia](./05-paralelos.md) |

> **Nota**: Los pasos de instalacion de Ubuntu Server y configuracion de red son los mismos para cualquier hipervisor.

## Pasos

| Paso | Descripcion |
|------|-------------|
| [01 - VirtualBox](./01-virtualbox.md) | Instalacion y configuracion de la VM (VirtualBox) |
| [02 - Ubuntu Server](./02-ubuntu-server.md) | Instalacion del sistema operativo |
| [03 - Red](./03-red.md) | Configuracion de red de la VM |

## Especificaciones sugeridas (cualquier hipervisor)

| Configuracion | Valor |
|---------------|-------|
| Nombre | `ubuntu-server` |
| SO | Ubuntu Server LTS (64-bit) |
| RAM | 4096 MB |
| CPU | 2 nucleos |
| Disco | 40 GB |
| Red | Placa en modo Bridge |

## Red en modo Bridge

La placa de red de la VM debe estar en **modo Bridge**. Esto hace que la VM reciba una IP de la misma red de su router, comportandose como un dispositivo independiente en la red local.

Esto es importante porque:
- La VM tendra su propia IP en la red local
- Podra acceder a ella via SSH por la IP
- El Cloudflare Tunnel (y otros metodos) funcionaran correctamente

## Proximo paso

Elige tu hipervisor y crea la VM:
- [VirtualBox](./01-virtualbox.md) (Windows, Linux, macOS - gratuito)
- [VMware Workstation Pro](./04-vmware.md) (Windows, Linux - gratuito para uso personal)
- [Parallels Desktop](./05-paralelos.md) (macOS - pago)
