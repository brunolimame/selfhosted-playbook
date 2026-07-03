# 03 - Maquina Virtual (VM)

Guia para criar e configurar a maquina virtual que rodara o servidor.

## Etapas

| Passo | Descricao |
|-------|-----------|
| [01 - VirtualBox](./01-virtualbox.md) | Instalacao e configuracao da VM |
| [02 - Ubuntu Server](./02-ubuntu-server.md) | Instalacao do sistema operacional |
| [03 - Rede](./03-rede.md) | Configuracao de rede da VM |

## Visao geral

A VM rodara Ubuntu Server LTS (sem interface grafica) dentro do VirtualBox. Toda a interacao com ela sera via SSH.

## Especificacoes sugeridas

| Configuracao | Valor |
|--------------|-------|
| Nome | `ubuntu-server` |
| Tipo | Linux / Ubuntu (64-bit) |
| RAM | 4096 MB |
| CPU | 2 nucleos |
| Disco | 40 GB (dinamico) |
| Rede | Placa em modo Bridge |

## Rede em modo Bridge

A placa de rede da VM deve estar em **modo Bridge**. Isso faz com que a VM receba um IP da mesma rede do seu roteador, comportando-se como um dispositivo independente na rede local.

Isso e importante porque:
- A VM tera seu proprio IP na rede local
- Voce podera acessa-la via SSH pelo IP
- O Cloudflare Tunnel (e outros metodos) funcionarao corretamente

## Proximo passo

[Instalar VirtualBox e criar a VM](./01-virtualbox.md)
