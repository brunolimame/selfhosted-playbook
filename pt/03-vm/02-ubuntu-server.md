# 03-02 - Instalacao do Ubuntu Server

## 1. Iniciar a instalacao

1. No VirtualBox, selecione a VM `ubuntu-server`
2. Clique em **Iniciar**
3. A VM inicializara a partir da ISO do Ubuntu Server
4. No menu inicial, selecione **Try or Install Ubuntu Server** e aguarde

## 2. Passos da instalacao

### 2.1 Idioma
- Selecione `English` (o idioma do sistema; configure locale pt_BR depois se desejar)
- Enter para continuar

### 2.2 Layout de teclado
- Selecione `Portuguese (Brazil)` ou `Portuguese`
- Enter para continuar

### 2.3 Network connections
- O instalador deve detectar automaticamente a placa em modo Bridge
- Anote o **IP** exibido (ex: `192.168.1.100`). Voce usara este IP para acessar via SSH durante a configuracao
- Selecione **Done** e Enter

### 2.4 Configure Proxy
- Deixe em branco
- **Done**

### 2.5 Configure Ubuntu Archive Mirror
- Deixe o padrao (`http://archive.ubuntu.com/ubuntu`)
- **Done**

### 2.6 Guided storage configuration
- Selecione **Use An Entire Disk** (usar disco inteiro)
- Selecione o disco (ex: `VBOX_HARDISK 40.0 GB`)
- **Done**

### 2.7 Storage configuration confirmation
- Confirme com **Continue**
- Isso vai particionar o disco automaticamente

### 2.8 Profile setup
| Campo | Valor sugerido |
|-------|---------------|
| Your name | `Ubuntu VM` |
| Your server's name | `ubuntu-vm` |
| Pick a username | `ubuntu` |
| Password | Escolha uma senha forte |
| Confirm password | Repita a senha |

> Importante: Guarde bem este usuario e senha. Sera usado para SSH e sudo.

### 2.9 Ubuntu Pro
- Selecione **Skip for now** (nao precisa do Ubuntu Pro)
- **Continue**

### 2.10 SSH Setup
- Marque a opcao **Install OpenSSH server** (com Enter)
- Selecione **Done**

### 2.11 Feature Server Snaps
- Nao selecione nenhum snap
- **Done**

### 2.12 Instalacao
- O instalador copiara os arquivos
- Ao final, selecione **Reboot Now**

### 2.13 Ejetar a ISO
- Quando o VirtualBox pedir para remover a midia, clique em **Devices > Optical Drives > Remove disk from virtual drive**
- Ou pressione Enter para remover automaticamente

## 3. Primeiro login

A VM reiniciara. Faca login com o usuario e senha definidos.

```bash
ubuntu-vm login: ubuntu
Password: [sua senha]
```

## 4. Atualizar o sistema

Assim que logar, atualize os pacotes:

```bash
sudo apt update && sudo apt upgrade -y
```

## 5. Verificar IP

Confirme o IP da VM:

```bash
ip a
```

O IP estara na interface `enp0s3` ou similar (geralmente `192.168.x.x`).

## 6. Testar SSH (do seu host)

No terminal do seu **host** (nao da VM), teste a conexao SSH:

**Windows (PowerShell)**:
```powershell
ssh ubuntu@192.168.1.100
```

**Linux/macOS**:
```bash
ssh ubuntu@192.168.1.100
```

Substitua `192.168.1.100` pelo IP da VM.

A partir de agora, toda configuracao pode ser feita via SSH, sem precisar da janela do VirtualBox.

## Proximo passo

[Configuracao de Rede da VM](./03-rede.md) - Configure IP fixo e hostname.
