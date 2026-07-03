# 03-03 - Configuracion de Red de la VM

## 1. Configurar IP fija (estatica)

Por defecto la VM usa DHCP. Para evitar que la IP cambie (lo que romperia el Cloudflare Tunnel), configure una IP estatica.

### 1.1 Descubra la informacion de la red

En la VM, ejecute:

```bash
ip route show default
ip a
```

Anote:
- **Interfaz**: Ej: `enp0s3`
- **Gateway**: Ej: `192.168.1.1` (ruta predeterminada)
- **IP actual**: Ej: `192.168.1.100`
- **DNS**: Ej: `8.8.8.8` o la IP del router

### 1.2 Editar el netplan

Ubuntu Server usa netplan para configurar la red. Edite el archivo de configuracion:

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

O, si el archivo tiene otro nombre (ej: `50-cloud-init.yaml`):

```bash
sudo ls /etc/netplan/
```

Sustituya el contenido por algo como:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8
```

Ajuste los valores segun su red local.

### 1.3 Aplicar la configuracion

```bash
sudo netplan apply
```

### 1.4 Verificar

```bash
ip a
ping -c 3 8.8.8.8
ping -c 3 google.com
```

## 2. Configurar DHCP reservation (opcional)

Si prefiere mantener DHCP pero garantizar la misma IP siempre, configure **DHCP Reservation** en el router:

1. Acceda al panel del router (generalmente `192.168.1.1`)
2. Localice la seccion de **DHCP Reservation** o **Static DHCP**
3. Anada la MAC address de la VM con la IP deseada
4. Guarde y reinicie la red de la VM

Para descubrir la MAC address de la VM:

```bash
ip link show enp0s3
```

Busque `link/ether xx:xx:xx:xx:xx:xx`.

## 3. Configurar hostname

El hostname ya fue definido como `ubuntu-vm` durante la instalacion. Para confirmar:

```bash
hostnamectl
```

Si necesita cambiarlo:

```bash
sudo hostnamectl set-hostname ubuntu-vm
```

Y anada en `/etc/hosts`:

```bash
echo "127.0.1.1 ubuntu-vm" | sudo tee -a /etc/hosts
```

## 4. Probar conectividad completa

```bash
# Prueba interna
ping -c 2 127.0.0.1

# Prueba gateway
ping -c 2 192.168.1.1

# Prueba internet
ping -c 2 1.1.1.1

# Prueba DNS
ping -c 2 google.com
```

Todo OK? Esta listo para el siguiente paso.

## 5. Apagar la VM de forma segura

Siempre apague la VM desde la terminal:

```bash
sudo shutdown now
```

No cierre la ventana de VirtualBox directamente.

## Proximo paso

[Dominio y DNS](../04-dominio.md) - Configure su dominio en Cloudflare.
