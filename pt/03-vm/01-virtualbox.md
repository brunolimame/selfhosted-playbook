# 03-01 - Instalacao do VirtualBox e Criacao da VM

## 1. Instalar o VirtualBox

### Windows
1. Acesse https://www.virtualbox.org/wiki/Downloads
2. Baixe `VirtualBox-x.x.x-xxx-Win.exe`
3. Execute o instalador como Administrador
4. Mantenha as opcoes padrao e avance
5. Se aparecer aviso sobre rede, confirme a instalacao dos drivers

> **Importante**: Desative o Hyper-V se estiver ativo (WSL, Docker Desktop podem conflitar). Execute no PowerShell como admin:
> ```powershell
> Disable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
> ```
> E reinicie o computador.

### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install -y virtualbox virtualbox-ext-pack
```

### macOS
1. Baixe o `VirtualBox-x.x.x-xxx-OSX.dmg`
2. Abra o arquivo e arraste o VirtualBox para a pasta Applications
3. Se o macOS bloquear, va em `Preferencias do Sistema > Seguranca e Privacidade` e permita

## 2. Baixar o Ubuntu Server ISO

Baixe a versao LTS mais recente:
```
https://ubuntu.com/download/server
```

Arquivo: `ubuntu-24.04-live-server-amd64.iso` (exemplo)

## 3. Criar a Maquina Virtual

Abra o VirtualBox e clique em **Novo**.

### Configuracoes da VM

| Campo | Valor |
|-------|-------|
| Nome | `ubuntu-server` |
| Pasta da maquina | Deixar padrao |
| Imagem ISO | `ubuntu-24.04-live-server-amd64.iso` (selecionar agora) |
| Tipo | Linux |
| Versao | Ubuntu (64-bit) |
| RAM | 4096 MB |
| CPU | 2 CPUs |
| Disco rigido | Criar disco virtual agora |
| Tamanho do disco | 40 GB |
| Tipo do disco | VDI (VirtualBox Disk Image) |
| Armazenamento | Dinamicamente alocado |

Na tela de criacao, clique em **Concluir**.

## 4. Ajustar configuracoes da VM

Selecione a VM `ubuntu-server` e clique em **Configuracoes**:

### Rede
- Aba **Rede**
- **Placa 1**: Habilitada
- **Conectado a**: `Placa em modo Bridge`
- **Nome**: Selecione sua placa de rede fisica (Wi-Fi ou Ethernet)

### Processador
- Aba **Sistema** > **Processador**
- Sliders: 2 CPUs, 100% de execucao

### Video
- Aba **Video**
- Memoria de video: 16 MB (minimo necessario para servidor)
- Aceleracao 3D: desligada

Clique em **OK**.

## 5. Verificacao

Sua VM `ubuntu-server` deve aparecer na lista do VirtualBox com a configuracao:
- Desligada
- Ubuntu (64-bit)
- 4096 MB RAM
- Bridge network

## Proximo passo

[Instalar Ubuntu Server na VM](./02-ubuntu-server.md) - Inicie a VM e siga o instalador.
