# 04 - Dominio y DNS

## 1. Elegir y registrar un dominio

Necesita un dominio propio. Ejemplos:

| Tipo | Ejemplo | Donde registrar |
|------|---------|-----------------|
| .com | `miservidor.com` | Namecheap, GoDaddy, Cloudflare Registrar |
| .com.br | `miservidor.com.br` | Registro.br |
| .dev | `miservidor.dev` | Namecheap, Cloudflare Registrar |

> **Consejo**: Registrar por Cloudflare Registrar cuesta precios de costo (sin margen de ganancia). Si ya tiene un dominio en otro registrador, puede moverlo a Cloudflare.

## 2. Crear cuenta en Cloudflare

1. Acceda a https://dash.cloudflare.com/sign-up
2. Cree una cuenta gratuita
3. Confirme el email

## 3. Anadir el dominio a Cloudflare

1. Despues del inicio de sesion, haga clic en **Add a Site**
2. Escriba su dominio (ej: `miservidor.com`)
3. Seleccione el plan **Free** (gratuito)
4. Cloudflare escanea los registros DNS existentes (puede tardar unos segundos)
5. Avance

## 4. Cambiar nameservers

Cloudflare mostrara dos nameservers como:
```
dns1.ns.cloudflare.com
dns2.ns.cloudflare.com
```

1. Acceda al panel de su registrador de dominio
2. Localice la opcion de cambiar **Nameservers** (DNS Server)
3. Sustituya los nameservers actuales por los de Cloudflare
4. Guarde

La propagacion puede tardar desde unos minutos hasta 48 horas (generalmente < 1 hora).

## 5. Verificar estado

En el panel de Cloudflare, el estado cambiara de **Pending** a **Active** cuando el cambio se haya propagado.

## 6. Crear registros DNS (provisionales)

Mientras configura el tunel, cree un registro DNS temporal para prueba:

En el panel de Cloudflare, vaya a **DNS > Records** y anada:

**Registro A** (para prueba local):
| Tipo | Nombre | Contenido | Proxy |
|------|--------|-----------|-------|
| A | `vm` | `192.168.1.100` | DNS only (desactivado) |

Esto crea `vm.miservidor.com` apuntando a la IP local de la VM.

> **Nota**: Este registro funcionara solo en la red local. La configuracion final usando Cloudflare Tunnel (u otro metodo) sustituira este registro.

## 7. Configurar SSL/TLS

En el panel de Cloudflare:
1. Vaya a **SSL/TLS > Overview**
2. Seleccione **Full (strict)** para mayor seguridad
3. En **Origin Server > Create Certificate**, genere un certificado autofirmado (opcional, pero recomendado)

## 8. Esperar propagacion de DNS

Verifique si el DNS ya se propagó:

```bash
# En su host (fuera de la VM)
nslookup vm.miservidor.com
# o
ping vm.miservidor.com
```

## Proximo paso

[Exposicion Publica](./05-exposicion-publica/README.md) - Elija como exponer su VM en internet.
