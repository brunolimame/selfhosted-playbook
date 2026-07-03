# 03-04 - VMware Workstation

VMware Workstation Pro ahora es gratuito para uso personal. Alternativa a VirtualBox para Windows y Linux.
- Descargar desde https://www.vmware.com/products/workstation-pro.html
- Descargar la ISO de Ubuntu Server: https://ubuntu.com/download/server
- Crear la VM en VMware:
  * Tipica (recomendada)
  * Seleccionar la ISO de Ubuntu Server
  * Linux > Ubuntu 64-bit
  * Nombre: `ubuntu-server`
  * Disco: 40 GB (dividir en varios archivos)
  * Personalizar: 4096 MB RAM, 2 CPU, Adaptador de red: Puente (Bridged)
- Iniciar la VM, el instalador de Ubuntu arrancara
- Nota: VMware Tools opcional para CLI, la red en modo Bridge funciona de fabrica, snapshots disponibles
- Siguiente paso: [Instalar Ubuntu Server en la VM](./02-ubuntu-server.md)
