# 02 - Pre-requisitos

## Hardware

| Componente | Minimo | Recomendado |
|------------|--------|-------------|
| Memoria RAM (host) | 8 GB | 16 GB+ |
| RAM para VM | 2 GB | 4 GB |
| Disco (host) | 30 GB livres | 80 GB+ |
| Disco para VM | 20 GB | 40 GB+ |
| Processador | 2 nucleos | 4+ nucleos |

## Software

### VirtualBox (gratuito)
- **Windows**: Baixe de https://www.virtualbox.org/wiki/Downloads (Windows hosts)
- **Linux**: `sudo apt install virtualbox` (Ubuntu/Debian)
- **macOS**: Baixe de https://www.virtualbox.org/wiki/Downloads (OS X hosts)

### Ubuntu Server ISO
- Baixe a ultima versao LTS: https://ubuntu.com/download/server
- Exemplo: `ubuntu-24.04-live-server-amd64.iso`

### SSH Client
- **Windows**: PowerShell ou [Windows Terminal](https://github.com/microsoft/terminal) (SSH nativo desde Windows 10 1809)
- **Linux/macOS**: SSH nativo no terminal

## Contas

### Cloudflare (gratuita)
- Crie uma conta em https://dash.cloudflare.com/sign-up
- Necessaria para DNS e Cloudflare Tunnel

### Registro de dominio
- Domino proprio (ex: `meudominio.com.br`)
- Recomendados: Cloudflare Registrar, Namecheap, Registro.br (.com.br)
- Custo tipico: R$ 30-80/ano

## Rede

- Acesso administrativo ao roteador (para configurar rede da VM)
- Conexao com internet estavel
- (Opcional) Capacidade de configurar DHCP reservation na rede local

## Verificacao rapida

Antes de comecar, execute no terminal do seu host:

**Windows (PowerShell)**:
```powershell
# Verificar se o Hyper-V esta desabilitado (pode conflitar com VirtualBox)
Get-WindowsOptionalFeature -FeatureName Microsoft-Hyper-V-All
```

**Linux/macOS**:
```bash
# Verificar suporte a virtualizacao
egrep -c '(vmx|svm)' /proc/cpuinfo
```

## Proximo passo

[Criacao da VM](./03-vm/README.md) - Instale o VirtualBox e crie a maquina virtual.
