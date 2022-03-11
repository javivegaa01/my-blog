---
title: "Instalación desatendida"
date: 2021-10-12T16:22:33+02:00
draft: true
---

# *Creación de un sistema automatizado de instalación.*

## *Instalación basada en fichero de configuración pressed.*

### *1.Introducción*

Los desarrolladores de Debian han creado un sistema que permite realizar instalaciones de forma automática o desatendida partiendo de un fichero de configuración (preseed).
Esta forma de instalación trae una gran ventaja que es un mecanismo para responder a preguntas realizadas durante la instalación sin tener que introducir manualmente las respuestas mientras ésta se ejecuta.

### *2.Pasos para realizar una instalación desatendida correctamente.*

#### *2.1.Descargar la iso.*

Desde la página oficial de debian descargamos la última versión de la iso con la arquitectura que deseemos.

#### *2.2.Modificar la iso.*

Podemos realizar varios métodos de preconfiguración, en este caso vamos a utilizar la preconfiguración por imagen de arranque.
Para modificar la iso vamos a crear una carpeta dentro del directorio /mnt debido a que tanto /mnt o /media son los directorios donde se establecen generalmente los puntos de montaje.

<pre><code class="shell">
vega@debian:/mnt$ cd
vega@debian:~$ cd /mnt/
vega@debian:/mnt$ sudo mkdir iso
vega@debian:/mnt$ ls
iso
vega@debian:/mnt$ cd ../home/vega/Descargas/
vega@debian:~/Descargas$ sudo mount -o loop debian-11.0.0-amd64-netinst.iso /mnt/iso/
vega@debian:~/Descargas$ cd /mnt/iso/
vega@debian:/mnt/iso$ ls
autorun.inf  css     dists  EFI       g2ldr      install      isolinux    pics  README.html          README.mirrors.txt  README.txt  tools
boot         debian  doc    firmware  g2ldr.mbr  install.amd  md5sum.txt  pool  README.mirrors.html  README.source       setup.exe   win32-loader.ini
vega@debian:/mnt/iso$ 
</code></pre>

Una vez montado la imagen en el directorio vamos a crear una imagen desatendida para ello no nos hacen falta todos los archivos que acabamos de ver, empezamos por,crear la carpeta, donde se van a alojar a mover todos los directorios y archivos necesarios, y crear un enlace simbólico que que apuntará al resto.

<pre><code class="shell">
vega@debian:/mnt$ sudo mkdir iso-preseed
vega@debian:/mnt$ cd iso-preseed/
vega@debian:/mnt/iso-preseed$ sudo cp -pr /mnt/iso/install.amd install.amd
vega@debian:/mnt/iso-preseed$ sudo cp -pr /mnt/iso/dists dists
vega@debian:/mnt/iso-preseed$ sudo cp -pr /mnt/iso/pool pool
vega@debian:/mnt/iso-preseed$ sudo cp -pr /mnt/iso/.disk .disk
vega@debian:/mnt/iso-preseed$ sudo cp -pr /mnt/iso/isolinux/ isolinux
vega@debian:/mnt/iso-preseed$ sudo ln -s .debian
</code></pre>

##### *2.2.1.Creación de archivo preseed*

Ahora pasaremos a crear el archivo de respuestas. Lo creamos en nuestro equipo en la ruta iso-preseed/respuestas con el nombre de preseed.cfg.
Recordemos que la finalidad de este tipo de instalación es  responder a preguntas realizadas durante la instalación sin tener que introducir manualmente las respuestas y por tanto estas hay que indicarlas en este fichero.
El fichero tiene la siguiente sintaxis. (el archivo que muestro a continuación está ya configurado para mi instalación)

<pre><code class="shell">
# Configuración de localización para el idioma y país.
d-i debian-installer/locale string es_ES.UTF8
d-i debian-installer/language string spanish
d-i debian-installer/country string Spain
d-i debian-installer/locale string es_ES.UTF-8

# Mapa del teclado
d-i keyboard-configuration/toggle select No toggling
d-i keymap select es

#Configuración de la red
# netcfg escogerá la interfaz que tiene enlace si puede. Esto hace que no
# muestre la lista si hay más de uno.
d-i netcfg/choose_interface select auto

#Configuración de los repositorios
d-i mirror/country string manual
d-i mirror/http/hostname string ftp.es.debian.org
d-i mirror/http/directory string /debian
d-i mirror/http/proxy string

#Configuración de reloj y zona horaria
d-i clock-setup/utc boolean true

#Particionado de discos
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true

#Usuarios y contraseñas
d-i passwd/root-password-crypted password usjiu68uCKNng
popularity-contest popularity-contest/participate boolean false
d-i passwd/user-fullname string usuario
d-i passwd/username string usuario
d-i passwd/user-password-crypted password usjiu68uCKNng

#Entorno grafico
tasksel tasksel/first multiselect

#Seleccion de paquetes.
d-i pkgsel/include string openssh-server
d-i apt-setup/cdrom/set-first boolean false
d-i apt-setup/cdrom/set-next boolean false
d-i apt-setup/cdrom/set-failed boolean false

#Grub
d-i grub-installer/only_debian boolean true
d-i grub-installer/with_other_os boolean true
d-i grub-installer/bootdev  string default
</code></pre>

Estos son todos los parámetros que añadido/modificado.
* Configuración de localización para el idioma y país.
* Mapa del teclado
* Configuración de la red
* Configuración de los repositorios
* Configuración de reloj y zona horaria
* Particionado de discos
* Usuarios y contraseñas
* Seleccion de paquetes
* Entorno gráfico
* Cargador de arranque 

Y la contraseña tanto del root como del usuario la he encriptado con el comando openssl passwd.

<pre><code class="shell">
root@debian:~# openssl passwd -crypt -salt usuario
Password: 
usjiu68uCKNng
</code></pre>

#### *2.3.Generar la iso*

Ahora vamos a preparar la imagen. Para ello nos desplazaremos hasta la carpeta isolinux que copiamos a iso-preseed en un paso anterior y editamos el archivo txt.cfg para añadirle una nueva línea:

<pre><code class="shell">
vega@debian:/mnt/iso-preseed/respuestas$ cd ../isolinux/
vega@debian:/mnt/iso-preseed/isolinux$ sudo nano txt.cfg 

default install
label install
        menu label ^Install
        kernel /install.amd/vmlinuz
        append vga=788 initrd=/install.amd/initrd.gz -- quiet
label unattended-gnome
 menu label ^Instalación Debian Desatendida Preseed Vega
 kernel /install.amd/gtk/vmlinuz
 append vga=788 initrd=/install.amd/gtk/initrd.gz preseed/file=/cdrom/respuestas/preseed.cfg locale=es_ES console-setup/ask_detect=false keyboard-configuration/xkb-keymap=es --
</code></pre>

Debido a que hemos realizado cambios en los contenidos de la carpeta islinux, deberemos volver a generar el archivo de sumas de verificación que irá ubicado en la raíz del CD. Ejecutaremos el comando (como root):

<pre><code class="shell">
vega@debian:/mnt/iso-preseed/isolinux$ su -
Contraseña: 
root@debian:~# cd /mnt/iso-preseed/isolinux/
root@debian:/mnt/iso-preseed/isolinux# md5sum `find ! -name "md5sum.txt" ! -path "./isolinux/*" -follow -type f` > md5sum.txt
</code></pre>

Para generar la iso solo falta instalar el paquete genisoimage y ejecutar el comando.

<pre><code class="shell">
apt-get update && apt-get install genisoimage
</code></pre>

<pre><code class="shell">
root@debian:/mnt# genisoimage -o cd-preseed.iso -l -r -J -no-emul-boot -boot-load-size 4 -boot-info-table -b isolinux/isolinux.bin -c isolinux/boot.cat iso-preseed
I: -input-charset not specified, using utf-8 (detected in locale settings)
Size of boot image is 4 sectors -> No emulation
  2.31% done, estimate finish Tue Sep 21 12:53:03 2021
  4.62% done, estimate finish Tue Sep 21 12:53:03 2021
  6.93% done, estimate finish Tue Sep 21 12:53:03 2021
  9.24% done, estimate finish Tue Sep 21 12:53:03 2021
 11.54% done, estimate finish Tue Sep 21 12:53:03 2021
 13.85% done, estimate finish Tue Sep 21 12:53:03 2021
 16.16% done, estimate finish Tue Sep 21 12:53:03 2021
 18.47% done, estimate finish Tue Sep 21 12:53:03 2021
 20.78% done, estimate finish Tue Sep 21 12:53:03 2021
 23.08% done, estimate finish Tue Sep 21 12:53:03 2021
 25.39% done, estimate finish Tue Sep 21 12:53:03 2021
 27.70% done, estimate finish Tue Sep 21 12:53:03 2021
 30.01% done, estimate finish Tue Sep 21 12:53:03 2021
 32.31% done, estimate finish Tue Sep 21 12:53:03 2021
 34.62% done, estimate finish Tue Sep 21 12:53:03 2021
 36.93% done, estimate finish Tue Sep 21 12:53:03 2021
 39.24% done, estimate finish Tue Sep 21 12:53:03 2021
 41.54% done, estimate finish Tue Sep 21 12:53:03 2021
 43.85% done, estimate finish Tue Sep 21 12:53:03 2021
 46.16% done, estimate finish Tue Sep 21 12:53:03 2021
 48.47% done, estimate finish Tue Sep 21 12:53:03 2021
 50.78% done, estimate finish Tue Sep 21 12:53:03 2021
 53.09% done, estimate finish Tue Sep 21 12:53:03 2021
 55.39% done, estimate finish Tue Sep 21 12:53:03 2021
 57.70% done, estimate finish Tue Sep 21 12:53:03 2021
 60.00% done, estimate finish Tue Sep 21 12:53:03 2021
 62.32% done, estimate finish Tue Sep 21 12:53:03 2021
 64.62% done, estimate finish Tue Sep 21 12:53:03 2021
 66.93% done, estimate finish Tue Sep 21 12:53:03 2021
 69.24% done, estimate finish Tue Sep 21 12:53:03 2021
 71.55% done, estimate finish Tue Sep 21 12:53:03 2021
 73.85% done, estimate finish Tue Sep 21 12:53:03 2021
 76.17% done, estimate finish Tue Sep 21 12:53:03 2021
 78.47% done, estimate finish Tue Sep 21 12:53:03 2021
 80.78% done, estimate finish Tue Sep 21 12:53:03 2021
 83.09% done, estimate finish Tue Sep 21 12:53:03 2021
 85.39% done, estimate finish Tue Sep 21 12:53:03 2021
 87.70% done, estimate finish Tue Sep 21 12:53:03 2021
 90.01% done, estimate finish Tue Sep 21 12:53:03 2021
 92.31% done, estimate finish Tue Sep 21 12:53:03 2021
 94.63% done, estimate finish Tue Sep 21 12:53:03 2021
 96.93% done, estimate finish Tue Sep 21 12:53:03 2021
 99.24% done, estimate finish Tue Sep 21 12:53:03 2021
Total translation table size: 2048
Total rockridge attributes bytes: 174993
Total directory bytes: 950272
Path table size(bytes): 7504
Max brk space used 19e000
216652 extents written (423 MB)
</code></pre>

*Prueba desde la interfaz virt-manager de Kvm*

Seleccionamos crear una nueva maquina mediante medio de instalación local.

![pxe](/images/instalacion_desatendida/maquina1.png/)

!cdrom.png!

Elegimos la iso que hemos modificado y le ponemos un nombre para identificarla.

!iso.png!

!nombre.png!

Como podemos observar aparece un apartado nuevo en el menú de instalación llamado Instalación desatendida preseed Vega, esa es la opción que tenemos que seleccionar.

!opción.png!

!inicio.gif!

!fin1.gif!

## *Instalación basada en PXE/TFTP*

### *Introducción*

Preboot eXecution Environment, es un entorno para arrancar e instalar el sistema operativo en computadoras a través de una red, de manera independiente de los dispositivos de almacenamiento de datos disponibles o de los sistemas operativos instalados,pero nos hacen falta dos elementos esenciales, el primero de ellos, un protocolo que nos permita servir los ficheros necesarios para la instalación a la máquina cliente. Ahí es donde entra el juego el protocolo TFTP (Trivial File Transfer Protocol), que es un protocolo de transferencia muy simple semejante a una versión básica de FTP.
Además del protocolo TFTP, necesitaremos hacer uso de un servidor DHCP que permita asignar direcciones IP dentro de la red a los clientes, ya que es un requisito básico para que los clientes se puedan comunicar con el servidor.

### *Configuración de la máquina que albergará el servidor*

<pre><code class="shell">
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
	config.vm.synced_folder ".", "/vagrant", disabled: true
	config.vm.define :spxe do |spxe|
		spxe.vm.box = "debian/buster64" 
		spxe.vm.hostname = "spxe"
		spxe.vm.network :private_network,
				:libvirt__network_name => "red_privada_pxe",
				:libvirt__dhcp_enabled => false,
				:ip => "192.168.150.10",
				:libvirt__forward_mode => "none"
		spxe.vm.provider :libvirt do |s|
			s.memory = 1024
			s.cpus = 1
			s.nested = true
			s.volume_cache = 'none'
		end
	end
end
</code></pre>

Hostname: spxe
Red: Creamos una interfaz de red privada, que tendrá un direccionamiento estático, ya que se trata del servidor.
Ram: 1024 GB

### *Instalación y configuración de TFTP y DHCP*

Para tener ambos servicios existe un paquete que nos facilita. dnsmasq

<pre><code class="shell">
vagrant@spxe:~$ sudo su -
root@spxe:~# apt-get update && apt-get upgrade
root@spxe:~# apt-get install dnsmasq
root@spxe:~# systemctl status dnsmasq.service | egrep Active
   Active: active (running) since Sat 2021-10-02 08:15:02 UTC; 27s ago
</code></pre>

Una vez instalado nos dirigimos al fichero /etc/dnsmasq.conf y lo editamos para que realice las funciones deseadas.

<pre><code class="shell">
#Servidor DHCP
#establecemos rango,mascara de red y duración de la concesión 
dhcp-range=192.168.150.50,192.168.150.100,255.255.255.0,12h

#Establecemos el fichero que los clientes van a usar para arrancar a través de la red
dhcp-boot=pxelinux.0

#Servidor TFTP
enable-tftp

#Especificamos el directorio que contendrá los ficheros para servir
tftp-root=/srv/tftp
</code></pre>

Para cargar la nueva configuración reiniciamos el servicio.

<pre><code class="shell">
root@spxe:~# systemctl restart dnsmasq.service 
</code></pre>

### *Configuración de los ficheros de instalación*

Una vez que el servicio dnsmasq está funcionando correctamente, es hora de descargar los ficheros necesarios en /srv/tftp.

<pre><code class="shell">
root@spxe:~# cd /srv/tftp/
root@spxe:/srv/tftp# 
</code></pre>

Lo primero será descargar los ficheros necesarios (wget), en el enlace que he dejado a continuación solo sería necesarios cambiar la distribución en mi caso bullseye.

<pre><code class="shell">
root@spxe:/srv/tftp# wget http://ftp.debian.org/debian/dists/bullseye/main/installer-amd64/current/images/netboot/netboot.tar.gz
--2021-10-02 08:43:34--  http://ftp.debian.org/debian/dists/bullseye/main/installer-amd64/current/images/netboot/netboot.tar.gz
Resolving ftp.debian.org (ftp.debian.org)... 199.232.182.132, 2a04:4e42:6d::644
Connecting to ftp.debian.org (ftp.debian.org)|199.232.182.132|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 38285766 (37M) [application/x-gzip]
Saving to: ‘netboot.tar.gz’

netboot.tar.gz      100%[===================>]  36.51M  68.2MB/s    in 0.5s    

2021-10-02 08:43:35 (68.2 MB/s) - ‘netboot.tar.gz’ saved [38285766/38285766]
</code></pre>

De esta manera tenemos los ficheros comprimidos en el directorio, pero para configurarlos hay que descomprimirlos.

<pre><code class="shell">
root@spxe:/srv/tftp# ls
netboot.tar.gz
root@spxe:/srv/tftp# tar -zxf netboot.tar.gz
root@spxe:/srv/tftp# ls
debian-installer  ldlinux.c32  pxelinux.0  pxelinux.cfg  version.info
root@spxe:/srv/tftp# 
</code></pre>

*tar*
* z: Utiliza gzip para descomprimir el fichero.
* x: Indica a tar que desempaquete el fichero.
* f: Indica a tar que el siguiente argumento es el nombre del fichero .tar.gz.

Ya existe el fichero pxelinux.0 que los clientes van a usar para arrancar a través de la red.

Y por último queda proporcionar al cliente conexión al exterior para descargar la paquetería durante la instalación, por tanto, tendremos que configurar NAT en el servidor, de manera que la máquina cliente sea capaz de salir al exterior a través del servidor. Para realizar esto último activare el bit de forward, permitiendo así que los paquetes puedan pasar de una interfaz a otra en el servidor, y con iptables estableceré las reglas en el corta fuegos.

<pre><code class="shell">
echo 1 > /proc/sys/net/ipv4/ip_forward

iptables -t nat -A POSTROUTING -o eth1 -j SNAT --to-source 192.168.121.30
</code></pre>

*Prueba*

He creado una máquina para probar el servidor.

!pxe1.png!

Para que arranque por red, nos dirigimos a Opciones de arranca y priorizamos el arranque por red.

!pxe2.png!

También conectamos la máquina a la red privada que creamos anteriormente.

!pxe3.png!

Arranque.

!pxe.gif!

## *Instalación desatendida desde servidor pxe*

Simplemente habría que colocar el fichero preseed dentro del servidor pxe.
Con un servidor web (apache2)  instalado, colocaremos el fichero preseed dentro del directorio /var/www/html ya que es el directorio por defecto de apache2.

Para instalar apache2:

<pre><code class="shell">
apt-get install apache2
</code></pre>


<pre><code class="shell">
root@spxe:/var/www/html# ls preseed.cfg 
preseed.cfg
root@spxe:/var/www/html# 
</code></pre>

Ahora nos dirigimos al directorio donde estaban guardados los ficheros de arranque y modificamos el fichero txt.cfg,quedando de la siguiente manera:

<pre><code class="shell">
root@spxe:/srv/tftp/debian-installer/amd64/boot-screens# ls txt.cfg 
txt.cfg
root@spxe:/srv/tftp/debian-installer/amd64/boot-screens# nano txt.cfg
</code></pre>

<pre><code class="shell">
default install
label install
        menu label ^Install
        kernel debian-installer/amd64/linux
        append vga=788 initrd=debian-installer/amd64/initrd.gz -- quiet
label unattended-gnome
 menu label ^Instalación Debian Desatendida Preseed Vega
 kernel debian-installer/amd64/linux
 append vga=788 hostname=spider domain='' initrd=debian-installer/amd64/initrd.gz url=http://192.168.150.10/preseed.cfg locale=es_ES console-setup/ask_detect=false keyboard-configuration/xkb-keymap=es --

## con el comando url= indicamos el alojamiento nuestro archivo preseed.cfg
## con el comando debian-installer/locale=es_ES escogemos el idioma español para el instalador.
## con el comando console-setup/ask_detect=false indicamos que no queremos la autodetección del teclado.
## con el comando keyboard-configuration/layoutcode=es elegimos la configuracion de teclado como español
</code></pre>

Por tanto el menú de arranque cambiará y nos proporcionará la opción de Instalación desatendida.

!pxe_preseed.gif!

Otro modo de arranque , es en el menú presionar esc y se nos abrirá una prompt en el que tendremos que escribir una instrucción de la siguiente forma:

<pre><code class="shell">
auto url = ip del servidor/fichero
</code></pre>
 
!autourl.gif!

## *Bibliografia*

Iso: https://www.debian.org/distrib/index.es.html

Doc Preseed: https://www.debian.org/releases/stable/armel/apbs02.es.html#preseed-loading

Kvm: https://wiki.debian.org/es/KVM

Pxe: https://es.wikipedia.org/wiki/Preboot_Execution_Environment