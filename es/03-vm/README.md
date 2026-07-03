# 03 - Maquina Virtual (VM)

Guia para crear y configurar la maquina virtual que ejecutara el servidor.

## Etapas

| Paso | Descripcion |
|------|-------------|
| [01 - VirtualBox](./01-virtualbox.md) | Instalacion y configuracion de la VM |
| [02 - Ubuntu Server](./02-ubuntu-server.md) | Instalacion del sistema operativo |
| [03 - Red](./03-red.md) | Configuracion de red de la VM |

## Vision general

La VM ejecutara Ubuntu Server LTS (sin interfaz grafica) dentro de VirtualBox. Toda la interaccion con ella sera via SSH.

## Especificaciones sugeridas

| Configuracion | Valor |
|---------------|-------|
| Nombre | `ubuntu-server` |
| Tipo | Linux / Ubuntu (64-bit) |
| RAM | 4096 MB |
| CPU | 2 nucleos |
| Disco | 40 GB (dinamico) |
| Red | Placa en modo Bridge |

## Red en modo Bridge

La placa de red de la VM debe estar en **modo Bridge**. Esto hace que la VM reciba una IP de la misma red de su router, comportandose como un dispositivo independiente en la red local.

Esto es importante porque:
- La VM tendra su propia IP en la red local
- Podra acceder a ella via SSH por la IP
- El Cloudflare Tunnel (y otros metodos) funcionaran correctamente

## Proximo paso

[Instalar VirtualBox y crear la VM](./01-virtualbox.md)
