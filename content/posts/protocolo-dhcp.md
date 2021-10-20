---
title: "Protocolo Dhcp"
date: 2021-10-20T10:56:48+02:00
draft: true
---

# Protocolo DHCP

![dhcp](/images/dhcp/dhcp.png/)

## ¿Qué es el protocolo DHCP?
El Protocolo de configuración dinámica de host (DHCP) es un protocolo cliente/servidor que proporciona automáticamente un host de Protocolo de Internet (IP) con su dirección IP y otra información de configuración relacionada, como la máscara de subred y la puerta de enlace predeterminada.El servicio DHCP está activo en un servidor donde se centraliza la gestión de la direcciones IP de la red. Hoy en día, muchos sistemas operativos incluyen este servicio dada su importancia.

### Ventajas y desventajas de DHCP

#### Ventajas
* Fácil configuración
* Facilita el seguimiento y supervisión de las direcciones IP, ya que son controladas por el servidor

#### Desventajas
* Cuando existe una falla en el servidor DHCP, los dispositivos renovarán su dirección IP, interrumpiendo la conexión e impidiendo su funcionamiento.

Una vez hablado un poco de que es dhcp vamos a crear un escenario, en el que vamos a configurar un servidor dhcp junto con unos clientes para mostrar lo que hemos comentado.

## Preparación del escenario

El escenario lo vamos a crear en libvirt/kvm y tendrá la siguiente forma:

* **Máquina Servidor**: Tiene tres tarjetas de red: una que le da acceso a internet (NAT o pública) y dos redes privadas (muy aisladas.
* **Máquina nodo_lan1**: Un cliente linux conectado a la primera red privada.
* **Máquina nodo_lan2**: Un cliente linux conectado a la segunda red privada.

![dhcp](/images/dhcp/escenario.png/)

### Instalación del escenario (libvirt/kvm)

Si no tienes instalado kvm, es muy sencillo, aquí te dejo un link 
* [es/KVM](https://wiki.debian.org/es/KVM)

Si nunca has trabajado con kvm mediante la línea de comandos , te dejo un par de links que te pueden interesar

* [Virtualización con Kvm](https://manuais.iessanclemente.net/index.php/Virtualizaci%C3%B3n_en_GNU/Linux_con_KVM)
* [Networking](https://wiki.libvirt.org/page/Networking)

## Creación de las redes

Antes de pasar a la terminal voy a comentar un poco los tipos de redes principales que hay en kvm.

* **Redes privadas basadas en NAT**: Se creará un dispositivo switch virtual que está conectado a todas las máquinas virtuales que estén conectadas a dicha red.
* **Red privada aislada (Isolated)**: En este caso las máquinas virtuales no tienen acceso al exterior. La definición es la misma que las redes NAT.
* **Red privada muy aislada (Veryisolated)**: Es una red a la que se conectan las máquinas virtuales pero que no está conectada el host. En este caso tendremos que utilizar direccionamiento estático para configurar la red de las máquinas virtuales.
* **Redes públicas conectadas a un bridge externo**: La máquina virtual estará conectado a un bridge que ha sido defenido en el hosts, es decir, la máquina virtual estará en la misma red que el host.

Comenzamos con la creación.
Lo primero de todo será crear las redes, que estarán definidas en ficheros .xml .

**Red 1: Pública**

```
<network>
    <name>Red1_Public</name>
    <forward mode="bridge"/>
    <bridge name="br0"/>
</network>
```

**Red 2: VeryIsolated**

```
<network>
    <name>Red2_Veryisolated</name>
    <bridge name="virbr35"/>
</network>
```

**Red 3: VeryIsolated**

```
<network>
  <name>Red3_Veryisolated</name>
  <bridge name="virbr40"/>
</network>
```

Para asegurarnos que no tenemos errores de síntaxis podemos ejecutar el siguiente comando:

```
virt-xml-validate 'nombre_fichero.xml'
```

Sí no tenemos ningún error podemos continuar.
Una vez tenemos lo ficheros, definimos e iniciamos las redes.

```
virsh -c qemu:///system net-define --file rpubdhcp.xml
virsh -c qemu:///system net-define --file rpridhcp.xml
virsh -c qemu:///system net-define --file rpridhcp2.xml

vega@spider:~$ virsh -c qemu:///system net-start --network Red1_Public 
La red Red1_Public se ha iniciado

vega@spider:~$ virsh -c qemu:///system net-start --network Red2_Veryisolated 
La red Red2_Veryisolated se ha iniciado

vega@spider:~$ virsh -c qemu:///system net-start --network Red3_Veryisolated 
La red Red3_Veryisolated se ha iniciado

```

## Instalación de las máquinas virtuales

Para instalar las máquinas utilizamos virt-install. Con este comando definiremos:
* Red se conecta la máquina : --network network=
* Cantidad de memoria Ram : --memory 
* Número de cpus : --vcpus
* Espacio de disco : --disk
* Nombre de la máquina : --name
* Sistema operativo que tendrá : --cdrom

**Instalación del Servidor**

```
virt-install --connect qemu:///system --cdrom ~/Descargas/debian-11.0.0-amd64-netinst.iso \
--memory 1024 \
--vcpus 1 \
--disk size=10 \
--network network=Red2_Veryisolated \
--name Nodo_Lan1
```

**Instalación de Nodo_Lan1 y Nodo_Lan2**

```
virt-install --connect qemu:///system --cdrom ~/Descargas/debian-11.0.0-amd64-netinst.iso \
--memory 1024 \
--vcpus 1 \
--disk size=10 \
--network network=Red3_Veryisolated \
--name Nodo_Lan2
```

En estas máquinas al estar conectadas a redes aisladas, tendremos que posponer la configuración de red.

![install](/images/dhcp/nointernet.png/)

Con esto ya tenemos creado el escenario.

```
vega@spider:~$ virsh -c qemu:///system list --all
 Id   Nombre              Estado
-----------------------------------
 -    Nodo_Lan1           apagado
 -    Nodo_Lan2           apagado
 -    Servidor            apagado
```
## Configuración de las máquinas.

### Servidor

En el servidor nos dirigimos al fichero ```/etc/network/interfaces``` y configuramos las redes veryisolated de forma estatica.

```
user-server@servidor:~$ cat /etc/network/interfaces

#ens4
auto ens4
iface ens4 inet static
	address 192.168.100.1
	netmask 255.255.255.0

#ens5
auto ens5
iface ens5 inet static
	address 192.168.200.1
	netmask 255.255.255.0
```

Para guardar los cambios ejecutamos 

```
systemctl restart systemctl restart networking.service && reboot
```

### Clientes

En el fichero ```/etc/network/interfaces``` de los clientes se establece que obtengan ip mediante dhcp.

**Nodo_Lan1**

![interfaces](/images/dhcp/interfaces_nodo1.png/)

**Nodo_Lan2**

![interfaces](/images/dhcp/interfaces_nodo2.png/)

Para guardar los cambios ejecutamos 

```
systemctl restart systemctl restart networking.service && reboot
```

## Servidor isc-dhcp-server

Para llevar a cabo la configuración utilizaremos el paquete ```isc-dhcp-server``` .

### ¿Que es isc-dhcp-server?

Internet Software Consortium. Es un paquete que incluye el servidor y el cliente llamado dhclient. Por tanto dhclient es el  Internet Software Consortium DHCP Client.Es el que instalan la mayoría de las distribuciones Linux, Originalmente escrito por Ted Lemon.

### Instalación de isc-dhcp-server

```
apt update && apt install isc-dhcp-server -y
```

Cuando instalamos el servidor por primera se produce un error, ya que no está configurado.

### Configuración

Para configurar isc-dhcp-server debemos modificar dos ficheros.

Ficheros de configuración de isc-dhcp-server
* /etc/default/isc-dhcp-server : para indicar la interfaz de escucha del DHCP.
* /etc/dhcp/dhcpd.conf : para indicar las redes que va a servir.

**Fichero /etc/default/isc-dhcp-server**
```
root@servidor:~# cat /etc/default/isc-dhcp-server | egrep 'INTERFACES' 
INTERFACESv4="ens4 ens5" 
```

**Fichero /etc/dhcp/dhcpd.conf**

``` 
root@servidor:~# cat /etc/dhcp/dhcpd.conf 

#Configuration nodo1
subnet 192.168.100.0 netmask 255.255.255.0 {
 range 192.168.100.10 192.168.100.80;
 option domain-name-servers 1.1.1.1, 1.0.0.1;
 option routers 192.168.100.1;
 default-lease-time 43200;
 max-lease-time 43200;
}

#Configuration nodo2
subnet 192.168.200.0 netmask 255.255.255.0 {
 range 192.168.200.30 192.168.200.50;
 option domain-name-servers 1.1.1.1, 1.0.0.1;
 option routers 192.168.200.1;
 default-lease-time 3600;
 max-lease-time 3600;
}
```

En el fichero dhcpd.conf he definido dos configuraciones, una para cada cliente.