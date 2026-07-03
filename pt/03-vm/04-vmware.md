# 03-04 - VMware Workstation

## Visao geral

VMware Workstation Pro agora e gratuito para uso pessoal. Alternativa ao VirtualBox para criacao de VMs.

## 1. Baixar e instalar o VMware Workstation Pro

- **Windows/Linux**: Baixe em https://www.vmware.com/products/workstation-pro.html
- Durante a instalacao, certifique-se de que a ponte VMnet esteja ativa para conectividade de rede

## 2. Baixar o ISO do Ubuntu Server

Mesmo ISO do guia do VirtualBox: https://ubuntu.com/download/server

## 3. Criar a VM no VMware

1. Abra o VMware Workstation
2. Clique em **"Create a New Virtual Machine"**
3. Selecione **"Typical (recommended)"**
4. Escolha **"Installer disc image file (iso)"** e selecione o ISO do Ubuntu Server
5. Guest OS: **Linux > Ubuntu 64-bit**
6. Nome da VM: `ubuntu-server`
7. Disco: **40 GB** (split into multiple files)
8. Customize Hardware:
   - Memory: **4096 MB**
   - Processors: **2 cores**
   - Network Adapter: **Bridged** (isto da a VM seu proprio IP na sua LAN)
   - Remova dispositivos desnecessarios (som, impressora, USB se nao forem necessarios)
9. **Finish**

## 4. Iniciar a VM

Inicie a VM e prossiga para a [Instalacao do Ubuntu Server](./02-ubuntu-server.md).

## 5. Notas especificas do VMware

- **VMware Tools** nao e essencial para o Ubuntu Server (ele fornece melhor integracao de mouse/video, que nao sao necessarios para CLI)
- **Rede Bridge** funciona diretamente no VMware, sem configuracao adicional
- **Snapshots** podem ser tirados no menu VM > Snapshot

## Proximo passo

[Instalar Ubuntu Server na VM](./02-ubuntu-server.md)
