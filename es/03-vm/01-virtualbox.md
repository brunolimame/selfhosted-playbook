# 03-01 - Instalacion de VirtualBox y Creacion de la VM

## 1. Instalar VirtualBox

### Windows
1. Acceda a https://www.virtualbox.org/wiki/Downloads
2. Descargue `VirtualBox-x.x.x-xxx-Win.exe`
3. Ejecute el instalador como Administrador
4. Mantenga las opciones predeterminadas y avance
5. Si aparece un aviso sobre red, confirme la instalacion de los drivers

> **Importante**: Desactive Hyper-V si esta activo (WSL, Docker Desktop pueden conflictuar). Ejecute en PowerShell como admin:
> ```powershell
> Disable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
> ```
> Y reinicie el computador.

### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install -y virtualbox virtualbox-ext-pack
```

### macOS
1. Descargue el `VirtualBox-x.x.x-xxx-OSX.dmg`
2. Abra el archivo y arrastre VirtualBox a la carpeta Applications
3. Si macOS lo bloquea, vaya a `Preferencias del Sistema > Seguridad y Privacidad` y permitalo

## 2. Descargar la ISO de Ubuntu Server

Descargue la version LTS mas reciente:
```
https://ubuntu.com/download/server
```

Archivo: `ubuntu-24.04-live-server-amd64.iso` (ejemplo)

## 3. Crear la Maquina Virtual

Abra VirtualBox y haga clic en **Nuevo**.

### Configuraciones de la VM

| Campo | Valor |
|-------|-------|
| Nombre | `ubuntu-server` |
| Carpeta de la maquina | Dejar predeterminado |
| Imagen ISO | `ubuntu-24.04-live-server-amd64.iso` (seleccionar ahora) |
| Tipo | Linux |
| Version | Ubuntu (64-bit) |
| RAM | 4096 MB |
| CPU | 2 CPUs |
| Disco duro | Crear disco virtual ahora |
| Tamano del disco | 40 GB |
| Tipo de disco | VDI (VirtualBox Disk Image) |
| Almacenamiento | Dinamicamente asignado |

En la pantalla de creacion, haga clic en **Finalizar**.

## 4. Ajustar configuraciones de la VM

Seleccione la VM `ubuntu-server` y haga clic en **Configuracion**:

### Red
- Pestana **Red**
- **Placa 1**: Habilitada
- **Conectado a**: `Placa en modo Bridge`
- **Nombre**: Seleccione su placa de red fisica (Wi-Fi o Ethernet)

### Procesador
- Pestana **Sistema** > **Procesador**
- Sliders: 2 CPUs, 100% de ejecucion

### Video
- Pestana **Video**
- Memoria de video: 16 MB (minimo necesario para servidor)
- Aceleracion 3D: desactivada

Haga clic en **OK**.

## 5. Verificacion

Su VM `ubuntu-server` debe aparecer en la lista de VirtualBox con la configuracion:
- Apagada
- Ubuntu (64-bit)
- 4096 MB RAM
- Bridge network

## Proximo paso

[Instalar Ubuntu Server en la VM](./02-ubuntu-server.md) - Inicie la VM y siga el instalador.
