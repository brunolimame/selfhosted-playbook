# 03-05 - Parallels Desktop

## Visao geral

Parallels Desktop e um hypervisor de VM para macOS. Excelente desempenho e integracao no Apple Silicon (M1/M2/M3/M4).

## 1. Instalar o Parallels Desktop

- Baixe em https://www.parallels.com
- Software pago (ha versao de teste)
- Compativel com Macs Intel e Apple Silicon

## 2. Baixar o ISO do Ubuntu Server

- https://ubuntu.com/download/server
- **Apple Silicon**: Escolha **"Ubuntu Server for ARM"**
- **Intel**: Escolha **"Ubuntu Server (AMD64)"**

## 3. Criar a VM

1. Abra o Parallels Desktop
2. Clique em **File > New**
3. Selecione **"Install from a file or DVD"** e escolha o ISO
4. O Parallels detectara automaticamente o Ubuntu Server
5. Nome da VM: `ubuntu-server`
6. Selecione **"Customize settings before installation"**
7. Configure:
   - **General**: Name = `ubuntu-server`
   - **Hardware > CPU & Memory**: 4096 MB RAM, 2 CPUs
   - **Hardware > Hard Disk**: 40 GB
   - **Hardware > Network**: Bridged (o padrao e Shared Network, altere para Bridged)
8. Clique em **Create**

## 4. Iniciar a VM

Inicie a VM e prossiga para a [Instalacao do Ubuntu Server](./02-ubuntu-server.md).

## 5. Notas especificas do Parallels

- **Parallels Tools** serao sugeridos, mas sao opcionais para servidor CLI
- No **Apple Silicon**, a VM roda nativamente com desempenho quase nativo
- A rede deve estar em **modo Bridged** para a VM ter seu proprio IP na rede
- **Snapshots**: disponiveis no menu VM > Manage Snapshots

## Proximo passo

[Instalar Ubuntu Server na VM](./02-ubuntu-server.md)
