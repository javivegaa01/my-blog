---
title: "Curso IPv6"
date: 2022-03-21T15:51:20+01:00
draft: true
---
Apuntes del curso ipv6 de la plataforma OpenWebinars impartido por el profesor Alberto Molina.

Si estas leyendo este post para informarte de IPv6 deberias tener claro como funciona IPv4 te dejo los apuntes de otro curso sobre IPv4 [Curso Redes TCP/IP](curso_tcp_ip.md)

# **1. Introducción**

## **1.1. ¿Por qué IPv6?**

La principal razón es que el protocolo IPv4 está agotado, no existen direcciones IPv4 públicas disponibles.

Es necesario un nuevo protocolo de capa 3 con direcciones mayores de 32 bits, este protocolo está planteado de la forma que con IPv4 tenemos 4.300 millones de direcciones pero con un bit más tendríamos el doble y con 2 bit mñas tendríamos más de 16.000 millones. Es por eso que en IPv6 existen 2^128 direcciones ip, por lo que la limitación de IPs públicas disponibles no deberia ser un inconveniente.

## **1.2. Caracteristicas de IPv6**

* Direcciones de 128 bits
* Autoconfiguración
* Segmentos de subred de tamaño fijo. En IPv6 no hay diferencia de máscara de red. En IPv4 por ejemplo podiamos encontrar /8, /16, /24. En IPv6 todas son /64 , es decir, los primeros 64 bits de esos128 siempre indican la red.

## **1.3. Consecuencias de IPv6**

No es posible prescindir de IPv4 en los proximos años. Lo que se pretende son mecanismos en los cuales convivan los dos protocolos, vamos a trabajar de forma habitual en lo que se conoce como doble pila.

En IPv6 no existe NAT propiamente por lo que IPv6 plantea una conexión extremo a extremo, es decir, no vamos a tener direccionamiento privado en una red local y un router conectado con una ip pública sino que tendremos direccionamientos globales a todos los niveles, esto significa que cualquier máquina conectada a IPv6 es o puede ser accesible desde el exterior.

# **2. Direcciones IPv6**

## **2.1. Representación**

Una dirección IPv6 consta de 128 bits.

La representación seria 'dirección_IPv6/prefijo'. El prefijo indica el número de bits que hacen referencia a la red.

Se agrupan en 8 grupos de 16 bits separados por:

* Notación hexadeciaml (mayúsculas y minúsculas).
* No se ponen ceros a la izquierda.
* Varios grupos de ceros pueden sustituirse pro ':'. Es decir si tuvieramos el primer bit 1, luego 126 bits 0 y otro bit 1 se representaría '1::1'.
* No existe el termino máscara de red,sino el termino prefijo

Ejemplo de dirección IPv6: 'fe80::e13f:f893:e0f1:26fs/64'

También podría escribirse: 'fe80:0000:0000:0000:e13f:f893:e0f1:26fs/64'

## **2.2. Ámbito de una dirección**

No se utilizan los términnos dirección pública y dirección privada tan habituales en IPv4. En IPv6 los equipos van a ser enrutables extremo a extremo, esto quiere decir que la ip que va a tener una máquina va a ser alcanzable desde el exterior y única en el mundo, este termino se conocer como ip global.

En este curso vamos a distinguir dos tipos de dirección por ámbito:

* Direcciones de enlace local.
* Direcciones globales.

Las direcciones de enlace local se utilizan para conectar a nivel local y realizar una serie de acciones que permiten configuraciones con otra ip global y con el exterior.

En IPv6 existe un direccionamiento tan grande que se intenta que incluso las ip de ámbito local sean únicas en el mundo. Los equipos en muchas ocasiones tienen más de una dirección IPv6, por que tiene direcciones de diferentes ámbitos.

### **2.2.1. Direcciones de enlace local**

Son las que se encuentran en el ámbito local. Estas direcciones las asigna el sistema operativo cuando se levanta la interfaz de red. Esta dirección es utilizada para comunicarse en la red local utilizando el protocol Network Discovey (ND). El prefijo que se utiliza es fe80::/10.

### **2.2.2. Direcciones globales**

Son análogas a las direcciones públicas de IPv4 y por tanto, son únicas en el mundo y son asignadas de forma jeráquica por la IANA, y las proporciona nuestro ISP. El prefijo que se utiliza es 2000::/3.

## **2.3. Esquemas de entrega**

![ipv6](/images/apuntes/curso_ipv6/esquema-entrega.png/)

* Unicast: conexión nodo a nodo.
* Anycast: parecido al unicast pero conecta con el nodo más cerca en tramos de enrutamiento.
* Multicast: entrega de uno a varios de un mismo grupo.
* No hay broadcast en IPv6. Se ha sustituido por usos de multicast ya que broadcast proporciona tráfico inncesario en una red.

### **2.3.1. Unicast y Anycast**

Esquema habitual en direcciones de enlace:

* 64 bits de red para enrutamiento.
* 64 bits para la interfaz de red.

Esquema habitual en direcciones globales:

* 48 bits de red para enrutamiento.
* 16 bits de subred.
* 64 bits para la interfaz de red.

Los 64 bits de red: fe80::/10 y relleno con 54 ceros los 64 bits de interfaz de red.

### **2.3.2. Multicast**

Prefijo ff00::/8.

De uso común:

* ff02::1 : Todos los nodos de la red local.
* ff02::2 : Todos los routers de la red local.
* ff02::1:ff00:0000/104 : Nodo solicitado. Cada interfaz de red se agregar a un grupo multicast con los últimos 3 octetos de su dirección de enlace local.
* Para Ethernet existen las direcciones 33:33.

## **2.4. Direcciones ip reservadas**

| Bloque         | Utilización                       |
|----------------|-----------------------------------|
| ::1/128        | Loopback                          |
| 2000::/3       | Direcciones globales (de momento) |
| ff00::/8       | Multicast                         |
| ::/128         | Dirección no especificada         |
| 2001::/32      | Teredo                            |
| 2001:db8::/32  | Documentación                     |
| 2002::/16      | 6to4                              |
| fc00::/7       | Unique Local Address              |

## **2.5. Direcciones de enlace local**

El sistema operativo asigna una dirección IPv6 única a cada interfaz de red al levanlarla. 

Los primeros 64 bits son fe80:0000:0000:0000 .

Para los 64 bits de la interfaz de red:

* El mecanismo utilizado es EUI-64, se basa en la utilización de la dirección física  MAC.
* Por defecto en windows se genera aleatoriamente.

### **2.5.1. EUI-64**
Extended Unique Identifier se utiliza para las direcciones MAC. 

En IPv6 las direcciones MAC son de 64 bits, los 3 primeros octetos se utilizan para determinar el fabricante de la tarjeta y los 3 últimos para determinar la tarjeta concreta.

![ipv6](/images/apuntes/curso_ipv6/mac1.png)

En EUI-64 estos bloques de 3 octetos se separan y se incluye en medio dos octetos que son FF y FE.

![ipv6](/images/apuntes/curso_ipv6/mac2.png)

### **2.5.2. Direcciones de enlace local en windows**

Se utiliza por defecto un número aleatorio para los 64 bits de la interfaz de red pero se puede deshabilitar para utilizar EUI-64. Para deshabilitarlo desde powershell:

<pre><code class="shell">
Get-NetIPv6Protocol | Select-Object RandomizeIdentifiers
Set-NetIPv6Protocol -RandomizeIdentifiers Disabled 
</code></pre>

### **Ejercicio: Uso básico de direcciones de enlace local**

El escenario de este ejercicio consta de tres máquinas: 

* Dos linux: mi anfitriona y una máquina virtual.
* Una windows: una máquina virtual.

Conectadas a la misma red.

![ipv6](/images/apuntes/curso_ipv6/ej1-escenario.png)

Como hemos visto las direcciones de enlace local se asignan automáticamente al levantar las interfaces, por lo que en cuanto tengamos dos maquinas conectadas en red , están conectadas a través de la dirección de enlace:

Máquina Linux:

![ipv6](/images/apuntes/curso_ipv6/ej1-mlinux.png)

Máquina Windows:

![ipv6](/images/apuntes/curso_ipv6/ej1-mwindows.png)

Para hacer ping desde una máquina linux utilizamos el comando ping6.

![ipv6](/images/apuntes/curso_ipv6/ej1-ping6.png)

**Nota: La opción -c es para enviar un número especifico de peticiones**

Para hacer ping desde windows utilizamos ping.

![ipv6](/images/apuntes/curso_ipv6/ej1-ping.png)

Es importante destacar que las direcciones de enlace local no son para utilizarlas como sutituto de IPv4 , es decir, deberia de utilizarse para terminar de configurar la red y conseguir una ip global.

### **Ejemplo: Direccionamiento global**

Queremos que TODOS los equipos de nuestra red tengan una dirección IPv6 global y que estén en la misma red. Nuestro proveedor de Internet IPv6 nos debe dar:

* Una dirección IPv6 global para nuestro router:
    * Dirección IP: 2001:db8:1::1a78:d055:216d:3a78/64
    * Dirección IP gateway: 2001:db8:1::1
* Un rango de direcciones para todos los equipos: 2001:db8:bbb1::/48 ← 16 bits de subred 

Configuración del router:

* eth0: 2001:db8:1::1a78:d055:216d:3a78/64
* eth1: 2001:db8:bbb1::1/64 ← Subred 0000
* Puerta de enlace: 2001:db8:1::1 

Configuración equipo subred 0000:

* eth0: 2001:db8:bbb1::a986:d0ff:fe9a:c67e/64
* Puerta de enlace: 2001:db8:bbb1::1

Configuración router subred 0001:
* eth0: 2001:db8:bbb1::2:10ff:fecb:31c4/64
* eth1: 2001:db8:bbb1:1::1/64
* Puerta de enlace: 2001:db8:bbb1::1

Configuración equipo subred 0001:
* eth0: 2001:db8:bbb1:1:2:aaff:fe23:a123/64
* Puerta de enlace: 2001:db8:bbb1:1::1 

![ipv6](/images/apuntes/curso_ipv6/ejemplo-global.png)

**Nota: No puedo realizar este ejemplo porque mi proovedor de internet no me ofrece direccionamiento IPv6 global.**

### **Configuracion estática IPv6 en Windows**

Accedemos a Panel de Control > Redes e Internet > Conexiones de Red > Click derecho Propiedades y seleccionamos el protocolo IPv6.

![ipv6](/images/apuntes/curso_ipv6/conf-estatica-win.png)

### **Configuracion estática IPv6 en Linux**

En linux utilizamos el comando ip.

Con ip -6 a podemos añadir, borrar, vamos a añadir una ip:

<pre><code class="shell">
ip -6 a add fe80::5054:ff:fea1:6a54/64 dev eth0
</code></pre>

![ipv6](/images/apuntes/curso_ipv6/ipa1.png)

Para la configuración permanente nos dirigimos a el fichero /etc/network/interfaces definos la configuración estática.

![ipv6](/images/apuntes/curso_ipv6/netin.png)

# **3. ICMPv6 y descubrimiento de vecinos**

## **3.1. ICMPv6**

ICMPv6 es la versión del protocolo ICMP para IPv6. Sus siglas significan Internet Control Message Protocol para IPv6.

Tiene el mismo formato de ICMP pero con más tipos de mensajes:

* Neighbor Discovery
* Secure Neighbor Discovery
* Multicast Listener Discovery
* Multicast Router Discovery 

## **3.2. Descubrimiento de vecinos**

Es el protocolo de descrubrimiento de vecinos o en inglés Network DIscobery es el que sustituye al protocolo ARP de IPv4.

Como ya sabemos en IPv6 no existe Broadcast por tanto a diferencia de ARP no utiliza peticiones Broadcast sino que utiliza peticiones Multicast y ICMPv6.

Usos de Network Discovery:

* Se utiliza para obtener la dirección de un máquina en la red.
* Se utiliza para obtener los parámetros de la red y así obtener una dirección global.
* También se utiliza para comprobar que no hay direcciones duplicadas.

Para averiguar la MAC se utilizan dos mecanismos:

* Neighbour Solicitation (NS):
    * Se envía la petición a la dirección multicast de nodo solicitado. Cuando una máquina levanta su interfaz de red lo que hace es que se agrega a un grupo multicast en los que pone los últimos 3 octetos de la dirección, con esto cada vex que haga una petición a la dirección multicast le va a llegar a la máquina.
    * En Ethernet se encapsula una trama destinada a la dirección multicast IPv6, esto se hace porque no tendría sentido que existiera broadcast en unos niveles y otros no.
* Neighbour Advertisement (NA):
    * Respuesta a NS usando direcciones unicast.

Para obtener los parámetros de la red:

* Se utilizan mensajes multicast con los routers de la red para proporcionar:
    * Prefijo de direccionamiento global.
    * Reglas de encaminamiento.
    * Parámetros de la red.
    * Servidores DNS.
* Además de NS y NA existen los mensajes RS (router solicitation), RA (router advertisement) y Redirect

# **4. Configuración de direcciones IPv6.**

## **4.1. SLAAC**

Slaac es un mecanismo cuyas siglas re refieren a Stateless Address Autoconfiguración, como indica su nombre los nodos de red se configuran automáticamente sin que el usuario tenga que realizar ninguna acción. No existe algo equivalente en IPv4.

Funcionamiento:

Al levntar la interfaz de red se configura la ip de enlace local, a través de ella se solicitan los parámetros de la red mediante un mensaje RS (multicast) y el router devuelve los parámetros de la red mediante un mensaje RA. Tras esto el nodo se autoconfigura con los parámetros que le han sido otorgados. Todo este proceso no queda registrado en el router (stateless).

Extensiones RA para DNS

* Las opciones de RA no incluıan inicialmente ́información de DNS. Ahora se incluyen:
    * RDNSS: Recursive DNS Server
    * DNSSL: Lista de dominios de búsqueda
* En los clientes debian hay que instalar rdnssd y
resolvconf

## **4.2. DHCPv6**

Es denominado Stateful address autoconfiguration protocol, es decir, aquí se queda un registro en el router ya que la máquina no obtiene la dirección IPv6 que quiere sino la que le da el servidor dhcp.

Para su funcionamiento utiliza multicast y mensajas ICMPv6 al no existir broadcast, en DHCPv4 veíamos como el proceso utilizaba la MAC de difusión en este caso como todos los nodos estan el grupo multicast pues se realiza el proceso a través de él.

A diferencia de DHCPv4 los puertos son:

* Servidor: 547/udp.
* Cliente: 546/udp.

Fases del mecanismo DHCPv6:

* SOLICIT: El cliente envia una peticón al puerto 547/udp, dirección IP multicast FF02::1:2.

* ADVERTISE: El servidor responde al puerto 546/udp y dirección de enlace local del cliente ofrece una dirección global.

* REQUEST: El cliente solicita la concesión de la IPv6 al servidor.

* CONFIRM: El servidor confirma la concesión.

# **5. Mecanismos de transición**

## **5.1. Coexistencia de IPv4 e IPv6**

Es normal que una red utilice ambas pilas (dual-stack). Lo que se plantea como una situación a medio plazo es que aunque poco a poco se incorporan equipos a IPv6, estos equipos sigan necesitando el protocolo IPv4 ya que una parte de Internet IPv4 no va a conectarse a Internet a IPv6.

Nuestro ISP nos debería proporcionar:

* Una IPv4 pública para el router y se configura a la red local con IPv4 privadas utilizando NAT

* Una IPv6 global para el router y un prefijo /48-64 para la red local. De manera que todos los equipos acceden y son accesibles directamente a Internet IPv6.

## **5.2. Mecanismos de transición**

Los utilizamos cuando no disponemos de IPv6 nativo.

Existen diferentes mecanismos, en funcion de:

* Se configura un solo nodo.
* Se configura una red local.
* Se dispone de un control sobre el router
* Se tiene IP pública dinámica o estática.

Un ejemplo de mecanismo de transicón sería que teniendo una conexión nativa exclusiva de IPv6 podamos acceder mediante túneles a equipos en IPv4 y mediante esa conexión podamos acceder de forma indirecta a IPv4.

Lista de mecanismos:

* Stateless IP/ICMP Translation (SIIT)
* Tunnel Broker (6in4)
* 6to4
* 6over4
* 6rd (IPv6 rapid develoment)
* ISATAP
* NAT64
* 464XLAT
* Dual-Stack Lite (DS-Lite)
* Teredo



