# 02 - Pre-requisitos

## Hardware

| Componente | Minimo | Recomendado |
|------------|--------|-------------|
| Memoria RAM (host) | 8 GB | 16 GB+ |
| RAM para VM | 2 GB | 4 GB |
| Disco (host) | 30 GB libres | 80 GB+ |
| Disco para VM | 20 GB | 40 GB+ |
| Procesador | 2 nucleos | 4+ nucleos |

## Software

### VirtualBox (gratuito)
- **Windows**: Descargue de https://www.virtualbox.org/wiki/Downloads (Windows hosts)
- **Linux**: `sudo apt install virtualbox` (Ubuntu/Debian)
- **macOS**: Descargue de https://www.virtualbox.org/wiki/Downloads (OS X hosts)

### Ubuntu Server ISO
- Descargue la ultima version LTS: https://ubuntu.com/download/server
- Ejemplo: `ubuntu-24.04-live-server-amd64.iso`

### SSH Client
- **Windows**: PowerShell o [Windows Terminal](https://github.com/microsoft/terminal) (SSH nativo desde Windows 10 1809)
- **Linux/macOS**: SSH nativo en la terminal

## Cuentas

### Cloudflare (gratuita)
- Cree una cuenta en https://dash.cloudflare.com/sign-up
- Necesaria para DNS y Cloudflare Tunnel

### Registro de dominio
- Dominio propio (ej: `mudominio.com`)
- Recomendados: Cloudflare Registrar, Namecheap, Registro.br (.com.br)
- Costo tipico: R$ 30-80/año

## Red

- Acceso administrativo al router (para configurar red de la VM)
- Conexion a internet estable
- (Opcional) Capacidad de configurar DHCP reservation en la red local

## Verificacion rapida

Antes de comenzar, ejecute en la terminal de su host:

**Windows (PowerShell)**:
```powershell
# Verificar si Hyper-V esta deshabilitado (puede conflictuar con VirtualBox)
Get-WindowsOptionalFeature -FeatureName Microsoft-Hyper-V-All
```

**Linux/macOS**:
```bash
# Verificar soporte de virtualizacion
egrep -c '(vmx|svm)' /proc/cpuinfo
```

## Proximo paso

[Creacion de la VM](./03-vm/README.md) - Instale VirtualBox y cree la maquina virtual.
