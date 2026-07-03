# Documentacion - Indice

Bienvenido a la documentacion del proyecto Servidor Remoto.

## Navegacion

| # | Seccion | Descripcion |
|---|---------|-------------|
| 01 | [Introduccion](./01-introduccion.md) | Vision general, arquitectura y objetivo |
| 02 | [Pre-requisitos](./02-pre-requisitos.md) | Hardware, software y cuentas necesarias |
| 03 | [Maquina Virtual (VM)](./03-vm/README.md) | Creacion de la VM en VirtualBox con Ubuntu Server |
| 04 | [Dominio y DNS](./04-dominio.md) | Registro de dominio y configuracion Cloudflare DNS |
| 05 | [Exposicion Publica](./05-exposicion-publica/README.md) | Metodos para exponer la VM en internet |
| 06 | [Coolify](./06-coolify/README.md) | Instalacion y gestion de aplicaciones |
| 07 | [Seguridad](./07-seguridad.md) | Buenas practicas de seguridad |
| 08 | [Mantenimiento](./08-mantenimiento.md) | Backup, actualizaciones y monitoreo |
| 09 | [Aplicaciones](./09-aplicaciones/README.md) | Odysseus, n8n, Evolution Go y mas |
| 10 | [Chatbot Multicanal](./10-chatbot-multicanal/README.md) | Bot para WhatsApp, Telegram, Facebook, Web, Email |
| 11 | [Proximas Sugerencias](./11-sugerencias.md) | Aplicaciones recomendadas (monitoreo, CI/CD, backup) |
| -- | [Herramientas Dev](./herramientas-dev.md) | Webhook inspectors, API clients, tunnels, mocks |

## Convenciones usadas en esta documentacion

- **`vm.doc.local`**: dominio de ejemplo. Sustituyalo por su dominio real.
- **`ubuntu-vm`**: hostname de la VM. Puede cambiarse durante la instalacion.
- Bloques de comando: asumen que el sistema operativo del host ya fue identificado.

## Acerca de

Este proyecto documenta paso a paso como transformar un computador local (o VM) en un servidor publicamente accesible, gestionado por Coolify, sin costo mensual adicional al del dominio.
