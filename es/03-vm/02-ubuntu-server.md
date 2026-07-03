# 03-02 - Instalacion de Ubuntu Server

## 1. Iniciar la instalacion

1. En VirtualBox, seleccione la VM `ubuntu-server`
2. Haga clic en **Iniciar**
3. La VM iniciara desde la ISO de Ubuntu Server
4. En el menu inicial, seleccione **Try or Install Ubuntu Server** y espere

## 2. Pasos de la instalacion

### 2.1 Idioma
- Seleccione `English` (el idioma del sistema; configure locale es_ES despues si lo desea)
- Enter para continuar

### 2.2 Layout de teclado
- Seleccione `Spanish` o `Spanish (Latin American)`
- Enter para continuar

### 2.3 Network connections
- El instalador debe detectar automaticamente la placa en modo Bridge
- Anote la **IP** mostrada (ej: `192.168.1.100`). Usara esta IP para acceder via SSH durante la configuracion
- Seleccione **Done** y Enter

### 2.4 Configure Proxy
- Deje en blanco
- **Done**

### 2.5 Configure Ubuntu Archive Mirror
- Deje el predeterminado (`http://archive.ubuntu.com/ubuntu`)
- **Done**

### 2.6 Guided storage configuration
- Seleccione **Use An Entire Disk** (usar disco completo)
- Seleccione el disco (ej: `VBOX_HARDISK 40.0 GB`)
- **Done**

### 2.7 Storage configuration confirmation
- Confirme con **Continue**
- Esto particionara el disco automaticamente

### 2.8 Profile setup
| Campo | Valor sugerido |
|-------|----------------|
| Your name | `Ubuntu VM` |
| Your server's name | `ubuntu-vm` |
| Pick a username | `ubuntu` |
| Password | Elija una contrasena segura |
| Confirm password | Repita la contrasena |

> Importante: Guarde bien este usuario y contrasena. Se usaran para SSH y sudo.

### 2.9 Ubuntu Pro
- Seleccione **Skip for now** (no necesita Ubuntu Pro)
- **Continue**

### 2.10 SSH Setup
- Marque la opcion **Install OpenSSH server** (con Enter)
- Seleccione **Done**

### 2.11 Feature Server Snaps
- No seleccione ningun snap
- **Done**

### 2.12 Instalacion
- El instalador copiara los archivos
- Al final, seleccione **Reboot Now**

### 2.13 Expulsar la ISO
- Cuando VirtualBox pida remover el medio, haga clic en **Devices > Optical Drives > Remove disk from virtual drive**
- O presione Enter para remover automaticamente

## 3. Primer inicio de sesion

La VM reiniciara. Inicie sesion con el usuario y contrasena definidos.

```bash
ubuntu-vm login: ubuntu
Password: [su contrasena]
```

## 4. Actualizar el sistema

Una vez conectado, actualice los paquetes:

```bash
sudo apt update && sudo apt upgrade -y
```

## 5. Verificar IP

Confirme la IP de la VM:

```bash
ip a
```

La IP estara en la interfaz `enp0s3` o similar (generalmente `192.168.x.x`).

## 6. Probar SSH (desde su host)

En la terminal de su **host** (no de la VM), pruebe la conexion SSH:

**Windows (PowerShell)**:
```powershell
ssh ubuntu@192.168.1.100
```

**Linux/macOS**:
```bash
ssh ubuntu@192.168.1.100
```

Sustituya `192.168.1.100` por la IP de la VM.

A partir de ahora, toda configuracion puede hacerse via SSH, sin necesidad de la ventana de VirtualBox.

## Proximo paso

[Configuracion de Red de la VM](./03-red.md) - Configure IP fija y hostname.
