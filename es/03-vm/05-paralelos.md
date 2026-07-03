# 03-05 - Parallels Desktop

Parallels Desktop para macOS (Intel + Apple Silicon). Producto de pago.
- Descargar desde https://www.parallels.com
- Descargar la ISO correcta:
  * Apple Silicon: Ubuntu Server para ARM
  * Intel: Ubuntu Server AMD64
- Crear la VM en Parallels:
  * File > New > Instalar desde archivo
  * Seleccionar la ISO
  * Nombre: `ubuntu-server`
  * Personalizar: 4096 MB RAM, 2 CPU, 40 GB disco, Red: Puente (Bridged)
- Iniciar la VM
- Nota: Parallels Tools opcional, rendimiento nativo en Apple Silicon, Bridge necesario para IP propia, snapshots disponibles
- Siguiente paso: [Instalar Ubuntu Server en la VM](./02-ubuntu-server.md)
