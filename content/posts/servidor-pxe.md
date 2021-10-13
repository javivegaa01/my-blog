---
title: "Servidor Pxe"
date: 2021-10-12T16:22:33+02:00
draft: true
---

# Servidor PXE
## Introducción
Preboot eXecution Environment, es un entorno para arrancar e instalar el sistema operativo en computadoras a través de una red, de manera independiente de los dispositivos de almacenamiento de datos disponibles o de los sistemas operativos instalados,pero nos hacen falta dos elementos esenciales, el primero de ellos, un protocolo que nos permita servir los ficheros necesarios para la instalación a la máquina cliente. Ahí es donde entra el juego el protocolo TFTP (Trivial File Transfer Protocol), que es un protocolo de transferencia muy simple semejante a una versión básica de FTP.
Además del protocolo TFTP, necesitaremos hacer uso de un servidor DHCP que permita asignar direcciones IP dentro de la red a los clientes, ya que es un requisito básico para que los clientes se puedan comunicar con el servidor.

## Soy la prueba de que el script funciona
