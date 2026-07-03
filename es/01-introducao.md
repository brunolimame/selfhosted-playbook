# 01 - Introduccion

## Objetivo

Crear un servidor casero accesible publicamente, ejecutando aplicaciones gestionadas por Coolify, sin depender de IP fija o servicios cloud costosos.

## Arquitectura

```
+------------------+       +------------------+       +------------------+
|  Usuario final   | ----> |  vm.doc.local    | ----> |  Cloudflare DNS  |
|  (navegador)     |       |  (HTTPS)         |       |  + Edge Network  |
+------------------+       +------------------+       +------------------+
                                                               |
                                                     (tunel seguro)
                                                               |
                                                     +----------+--------+
                                                     |  cloudflared     |
                                                     |  (agente local)  |
                                                     +----------+--------+
                                                               |
                                                     +----------+--------+
                                                     |  VM Ubuntu Server |
                                                     |  +------------+   |
                                                     |  | Coolify    |   |
                                                     |  | (puerto 8000)|  |
                                                     |  +------------+   |
                                                     |  | Docker     |   |
                                                     |  +------------+   |
                                                     +-------------------+
```

## Flujo de peticiones

1. Usuario accede a `https://vm.doc.local`
2. DNS de Cloudflare resuelve hacia el edge de Cloudflare
3. Cloudflare reenvia la peticion por el tunel seguro hasta el `cloudflared`
4. `cloudflared` entrega la peticion a Coolify (puerto 8000)
5. Coolify gestiona el enrutamiento hacia la aplicacion correcta
6. La respuesta sigue el camino inverso

## Por que este enfoque?

| Caracteristica | Beneficio |
|----------------|-----------|
| Sin IP fija | Cloudflare Tunnel crea conexion outbound, funciona con IP dinamica |
| Sin abrir puertos | Ningun puerto necesita ser expuesto en el router |
| SSL/TLS automatico | Cloudflare proporciona certificado HTTPS gratuito |
| Bajo costo | Solo el costo del dominio (Cloudflare Tunnel es gratuito) |
| Coolify | Interfaz Web para gestionar aplicaciones sin complicacion |

## Proximo paso

[Pre-requisitos](./02-pre-requisitos.md) - Verifique lo que necesita antes de comenzar.
