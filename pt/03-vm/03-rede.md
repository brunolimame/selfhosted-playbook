# 03-03 - Configuracao de Rede da VM

## 1. Configurar IP fixo (estatico)

Por padrao a VM usa DHCP. Para evitar que o IP mude (o que quebraria o Cloudflare Tunnel), configure um IP estatico.

### 1.1 Descubra as informacoes da rede

Na VM, execute:

```bash
ip route show default
ip a
```

Anote:
- **Interface**: Ex: `enp0s3`
- **Gateway**: Ex: `192.168.1.1` (rota padrao)
- **IP atual**: Ex: `192.168.1.100`
- **DNS**: Ex: `8.8.8.8` ou o IP do roteador

### 1.2 Editar o netplan

O Ubuntu Server usa netplan para configurar rede. Edite o arquivo de configuracao:

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

Ou, se o arquivo tiver outro nome (ex: `50-cloud-init.yaml`):

```bash
sudo ls /etc/netplan/
```

Substitua o conteudo por algo como:

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

Ajuste os valores conforme sua rede local.

### 1.3 Aplicar a configuracao

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

Se preferir manter DHCP mas garantir o mesmo IP sempre, configure **DHCP Reservation** no roteador:

1. Acesse o painel do roteador (geralmente `192.168.1.1`)
2. Localize a secao de **DHCP Reservation** ou **Static DHCP**
3. Adicione o MAC address da VM com o IP desejado
4. Salve e reinicie a rede da VM

Para descobrir o MAC address da VM:

```bash
ip link show enp0s3
```

Procure por `link/ether xx:xx:xx:xx:xx:xx`.

## 3. Configurar hostname

O hostname ja foi definido como `ubuntu-vm` durante a instalacao. Para confirmar:

```bash
hostnamectl
```

Se precisar alterar:

```bash
sudo hostnamectl set-hostname ubuntu-vm
```

E adicione no `/etc/hosts`:

```bash
echo "127.0.1.1 ubuntu-vm" | sudo tee -a /etc/hosts
```

## 4. Testar conectividade completa

```bash
# Teste interno
ping -c 2 127.0.0.1

# Teste gateway
ping -c 2 192.168.1.1

# Teste internet
ping -c 2 1.1.1.1

# Teste DNS
ping -c 2 google.com
```

Tudo OK? Voce esta pronto para o proximo passo.

## 5. Desligar a VM com seguranca

Sempre desligue a VM pelo terminal:

```bash
sudo shutdown now
```

Nao feche a janela do VirtualBox diretamente.

## Proximo passo

[Dominio e DNS](../04-dominio.md) - Configure seu dominio no Cloudflare.
