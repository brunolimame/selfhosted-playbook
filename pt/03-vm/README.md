# 03 - Maquina Virtual (VM)

Guia para criar e configurar a maquina virtual que rodara o servidor.

## Escolha seu hypervisor

| Hypervisor | Plataforma | Custo | Documentacao |
|------------|-----------|-------|--------------|
| VirtualBox | Windows, Linux, macOS | Gratuito | [Guia](./01-virtualbox.md) |
| VMware Workstation Pro | Windows, Linux | Gratuito (uso pessoal) | [Guia](./04-vmware.md) |
| Parallels Desktop | macOS (Intel + Apple Silicon) | Pago | [Guia](./05-paralelos.md) |

> **Nota**: Os passos de instalacao do Ubuntu Server e configuracao de rede sao os mesmos para qualquer hypervisor.

## Etapas

| Passo | Descricao |
|-------|-----------|
| [01 - VirtualBox](./01-virtualbox.md) | Instalacao e configuracao da VM (VirtualBox) |
| [02 - Ubuntu Server](./02-ubuntu-server.md) | Instalacao do sistema operacional |
| [03 - Rede](./03-rede.md) | Configuracao de rede da VM |

## Especificacoes sugeridas (qualquer hypervisor)

| Configuracao | Valor |
|--------------|-------|
| Nome | `ubuntu-server` |
| SO | Ubuntu Server LTS (64-bit) |
| RAM | 4096 MB |
| CPU | 2 nucleos |
| Disco | 40 GB |
| Rede | Placa em modo Bridge |

## Rede em modo Bridge

A placa de rede da VM deve estar em **modo Bridge**. Isso faz com que a VM receba um IP da mesma rede do seu roteador, comportando-se como um dispositivo independente na rede local.

Isso e importante porque:
- A VM tera seu proprio IP na rede local
- Voce podera acessa-la via SSH pelo IP
- O Cloudflare Tunnel (e outros metodos) funcionarao corretamente

## Proximo passo

Escolha seu hypervisor e crie a VM:
- [VirtualBox](./01-virtualbox.md) (Windows, Linux, macOS - gratuito)
- [VMware Workstation Pro](./04-vmware.md) (Windows, Linux - gratuito para uso pessoal)
- [Parallels Desktop](./05-paralelos.md) (macOS - pago)
