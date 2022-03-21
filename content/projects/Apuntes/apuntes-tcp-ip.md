---
title: "Apuntes curso Redes TCP/IP"
date: 2022-03-21T18:13:47+02:00
draft: true
---

# **CURSO REDES TCP/IP**

Apuntes del curso redes tcp/ip de OpenWebinars impartido por Alberto Molina.

## **1. Introducción y conceptos previos**

Conmutación: mecanismo que se utiliza para poner en contacto dos teléfonos que no están en contacto directamente.

Latencia: tiempo que tarda en llegar la información de un punto a otro.

Velocidad de transmisión: es la cantidad de información que la red es capaz de transmitir de un punto a otro. (bit por segundo).

Datagramas: cantidad de información conjunta que viaja por la red de ordenadores.

### **1.1. Protocolos de red**

Un protocolo de red es el conjunto de reglas que rigen el intercambio de información a través de una red de ordenadores.

Una aplicación no se encarga de aspectos de bajo nivel, es el sistema operativo el que se encarga.

Ejemplo: en una aplicación de mensajería sería esta la que le indica al sistema operativo dónde quiere mandar el mensaje, el sistema operativo del otro dispositivo termina abriendo el contenido que le ha llegado y se lo manda a la aplicación correspondiente.

Este mecanismo comentado en el ejemplo es un proceso complejo que utiliza un modelo de capas.

El modelo de capas es un conjunto de protocolos que están uno encima de otro y se intentan comunicar.

El encapsulamiento es un proceso que encarga de envolver las capas y que contiene como dato el contenido completo de la capa superior.

El modelo de capas se traduce en niveles de modelo TCP/IP.

![tcp-ip](/images/proyectos/apuntes/capas-tcp-ip.png/)


## **2. Direcciones IPv4**

### **2.1. ¿Qué es una dirección IPv4?**

Es un número que identifica una interfaz de red de un equipo dentro de la red. Es más no solo identidica al equipo sino también a la red en sí misma.

Todos los equipos conectados a una red tienen que tener una dirección IPv4 única. Estas direcciones son repartidas por la Internet Assigned Numbers Authority (IANA).

### **2.2. Dirección IP: Bits, NDP**

Las direcciones IPv4 son números de 32 bits, es decir, existen 2^32 direcciones ip en el mundo. Las direcciones IPv4 se han acabado, no existen nuevas direcciones IPv4 disponibles.

En esos 32 bits que componen la dirección se encuentran una serie de bits que indican la red en la que se encuentra y otra serie de bits que indican el número de la interfaz de red del equipo conectado.

No todas las direcciones ip identifican hosts de forma única, solo las que empiezan por 0,10 ó 110, es decir, las direcciones ip de uso normal son las que están entre 0.0.0.0 y la 223.255.255.255 .

### **2.3. Máscara de red. Notación CIDR**

La máscara de red es un número que indica de una dirección Ipv4, cuantos bits son de red y cuantos de hosts.

*Ejemplo: 255.255.255.0 para 192.168.1.1*

*11111111.11111111.11111111.00000000*  
*Los primeros 24 bits indican la red. Los 8 restantes el host dentro de la red. Es decir, red -> 192.168.1 , host -> 1 .*

La notación CIDR utiliza una '/' seguido de un número de bits. En caso del ejemplo anterior /24.

*Ejercicios*  
*Ejercicio 1: Determina cuales de los siguientes pares de hosts están en la misma red*

*8.8.8.8/8 y 8.7.8.8/8*  
*192.168.1.1/24 y 192.168.1.2/24*  
*80.59.1.152/22 y 80.59.14.234/22*

*Están en la misma red los hosts 192.168.1.1/24 y 192.168.1.2/24 ya que los primeros 24 bits que indican la red coinciden*

*Ejercicio 2: Determina eñ número de hosts que puede haber en una red /8, /12, /16, /28.*  

*/28 -> 255.255.240.0 -> 16 hosts.*  
*/16 -> 255.255.0.0 -> 65.536 hosts.*  
*/12 -> 255.240.0.0 ->1.048.576 hosts.*  
*/8  -> 255.0.0.0 -> 16.777.216 hosts.*

### **2.4. Esquemas de red**

![tcp-ip](/images/proyectos/apuntes/esquema-de-red.png/)

Unicast: proceso por el cuál se envía un paquete de un host a un host individual.  

Anycast: parecido al unicast pero como se pueden repetir se envia al host más cerca en tramós de enrutamiento.

Multicast: proceso por el cuál se envía un paquete de un host a un grupo seleccionado de hosts, posiblemente en redes distintas.  

Broadcast: proceso por el cuál se envía un paquete de host a todos los hosts de la red.

### **2.5. Direcciones ip reservadas**

| Bloque               | Rango                          | Utilización   |
|----------------------|--------------------------------|---------------|
| 10.0.0.0/8           | 10.0.0.0 - 10.255.255.255      | IPs privadas  |
| 100.64.0.0/10        | 100.64.0.0 - 100.127.255.255   | CGNAT         |
| 127.0.0.0/8          | 127.0.0.0 - 127.255.255.255    | Loopback      |
| 169.254.0.0/26       | 169.254.0.0 - 169.254.255.255  | Enlace local  |
| 172.16.0.0/12        | 172.16.0.0 - 172.31.255.255    | IPs privadas  |
| 192.0.2.0/24         | 192.0.2.0 - 192.0.2.255        | Documentación |
| 192.168.0.0/16       | 192.168.0.0 - 192.168.255.255  | IPs privadas  |
| 192.51.100.0/24      | 192.51.100.0 - 192.51.100.255  | Documentación |
| 203.0.113.0/24       | 203.0.113.0 - 203.0.113.255    | Documentación |
| 224.0.0.0/4          | 224.0.0.0 - 239.255.255.255    | Multicast     |
| 240.0.0.0/4          | 240.0.0.0 - 254.255.255.255    | Usos futuros  |
| 255.255.255.255/32   |                                | Broadcast     |

### **2.6. Direcciones ip públicas y privadas**

Internet cuando se desarrolló con direccionamiento IPv4 solo utilizaba direcciones ip públicas.  

Una dirección pública es una dirección que es alcanzable extremo a extremo.

*Ejemplo videoconferencia:*

![tcp-ip](/images/proyectos/apuntes/ejemplo-videoconferencia.png/)

*1.Actualmente para que el equipo A el equipo B establezcan una videoconferencia solo tienen que ponerse de acuerdo en que app escoger y la conexión es directa.*

*2.Antiguamente para que dos equipos tuvieran una videoconferencia tenian que estar conectados a un tercero, esto era posible ya qu elos equipos tenián por sí mismos una dirección ip pública.*

Actualmente todos tenemos un dispositivo router que hace de NAT para que las redes privadas tengan conexión a internet.

### **2.7. Enrutamiento básico. Puerta de enlace**

#### **2.7.1. Enrutamiento estático**

El enrutamiento o encaminamiento es el mecanismo utilizado para enviar una petición de un equipo a su destino.

Las reglas de encaminamiento o enrutamiento son el conjunto de reglas que especifican el camino que debe seguir en cada caso.

Existe también el enrutamiento dinámico utilizado por routers en redes complejas.

#### **2.7.2. Tipos de reglas de enrutamiento**

* Reglas de hosts: específicas para una ip. 10.0.22.1/32 vía 192.168.1.14 .
* Reglas de red: específicas para un bloque. 10.0.0.0/16 vía 192.168.0.1 .
* Encaminamiento por defecto: 0.0.0.0/0 vía 192.168.0.1. Se denomina puerta de enlace o gateway. 

#### **2.7.3. Prioridad en la tabla de encaminamiento**

* No importa el orden de escritura.
* Se aplican en primer lugar las reglas de host.
* En segundo lugar las reglas específicas de un bloque.
* Como última opción se utiliza la regla de enrutamiento por defecto.

### **2.8. Introducción a NAT**

Debido a que las direcciones IPv4 se han acabado, NAT es un recurso necesario hoy en día, ya que gracias a NAT se pueden realizar dos acciones concretas. Una de ellas es que los equipos conectados a una red con direccionamiento privado puedan acceder a internet con una ip pública. Este proceso es lo que se conoce como Source NAT (SNAT). La segunda acción que lleva a cabo NAT es la denominada Destination NAT. DNAT consigue que un equipo pueda acceder a un servicio ubicado en un equipo con dirección privada.

### **2.9. Introducción a DHCP**

DHCP  es un protocolo que nos permite la configuración dinámica de las máquinas.

Lleva a cabo un mecanismo que consiste en que una máquina se conecta a una red, y solicita a una ip.

Fases del mecanismo DCHP:

* DHCP DISCOVERY: La máquina envía una petición al broadcast. 'H:¿Hay algún DCHP en la red?'
* DHCP OFFER: Un servidor DHCP ofrece una ip. 'SER-DCHP: Esta ip está disponible'
* DHCP REQUEST: La máquina solicita la ip. 'H: Dame la ip'
* DHCP ACK: El servidor responde afirmativamente a la petición. 'SER-DCHP: Toma una ip'


## **3. Resolución de nombres**

Las IPs están muy bien para las máquinas pero las personas recordamos muy mal los números sobre todo si son largos. La solución es asociar nombres a las direcciones IP.

### **3.1. Resolución estática**

Si utilizamos resolución estática guardamos la asociación entre nombres e IPs en un fichero. Por ejemplo /etc/hosts.

En el funcionamiento de las máquinas la prioridad siempre es la resolución estática frente a la dinámica.

### **3.2. Resolución dinámica**

Hoy en día la resolución de nombre en internet se basa en un protocolo llamado DNS.

DNS tiene un sistema jerarquico, por ejemplo, www.google.es.

               .
             es
       google
    www

La resolución dinámica sirve a la hora de reconocer las direcciones ip de tu red doméstica, que cambian constantemente, a un nombre fijo.

## **4. Redes Ethernet**

Ethernet es el estándar de redes cableadas, surgió en 1983 y se ha ido modificando según las necesidades actuales.

### **4.1. Nivel físico**

El nivel fisico se encarga del envío de bits entre los nodos. Proporciona una interfaz estandarizada.

Esta capa física se ocupa de:

* Topología de red: bus, anillo, malla o estrella.
* Comunicación en serie o paralela.
* Modo de transmisión: Simplex, half duplex o full duplex.

### **4.2. Nivel de enlace**

Las funciones principales del nivel de enlace son:

* Emisor: transformar los bits a transmitir por el medio en una señal.
* Receptor: extraer de una señal del medio los bits transmitidos.
* Gestionar el acceso al medio si es compartido.
* Detectar errores de transmisión.
* Opcionalmente, corregir errores.

### **4.3. Dirección MAC**

La dirección MAC es una dirección de la tarjeta de red. Normalmente se ponen dentro de la configuración de la tarjeta de fabrica.

Formas de representar la MAC:

* MAC48. Un número de 48 bits escrito en 6 octetos en hexadecimal. Ejemplo: 'f4:aa:12:23:34:67'
* EUI64. Un número de 64 bits escrito en 8 octetos en hexadecimal, es una extensión de MAC48.
* MAC de difusión: 'FF:FF:FF:FF:FF:FF'. Se refiere a todos los nodos de la red.

### **4.4. Trama Ethernet**

Es un paquete de datos en una red Ethernet. Esquema de una trama ethernet:

![tcp-ip](/images/proyectos/apuntes/trama-ethernet.png/)

* Preamble: 7 octetos de sincronización 10101010 .
* SFD: Start Frame Delimiter, 1 octeto 10101011 .
* Dirección MAC destino.
* Dirección MAC origen.
* Carga o payload: Hasta 1500 octetos.
* FCS: Frame Check Sequence, es un CRC de 4 octetos.

### **4.5. ARP**

Es un protocolo cuyas significan protocolo de resolución de direcciones, pero que se refiere a direcciones físicas o direcciones MAC.

Funcinamiento:

ARP es un protocolo que se utiliza para preguntar que direcciones ip tiene una MAC. Antes de enviar una trama a una ip concreta tengo que saber que dirección MAC destino tiene.

Normalmente sabemos a que nombre o ip tenemos que mandar la petición, entonces preguntamos que MAC tiene la ip la que quiero envíar la trama. Esta pregunta llega a todos los equipos de la red mediante la MAC de difusión, el equipo correspondiente responderá así ya se puede enviar la trama a ese equipo , sabiendo la dirección ip y la dirección MAC.

Es interesante saber que la respuesta de la MAC se cachea temporalmente.

## **5. Nivel de red**

El nivel de red se encargar de que los paquetes que salen del emisor lleguen al destino final, aunque el emisor y receptor no estén "adyacentes".

Esto normalmente requiere pasar a través de nodos intermediarios denominados encaminadores (routers).

El protocolo más utilizado a nivel de red es Internet Protocol (IP). En IP los paquetes reciben el nombre de datagramas.

### **5.1. Capa IP**

Funciones principales:

* Encaminamiento de los paquetes: llegan los datos de la trama ethernet, se desencapsulán y se sube a un determinado nivel y obtiene el siguiente salto que tiene que hacer el paquete para llegar a su destino.

* Asignación de direcciones únicas a todas las máquinas de la red.

### **5.2. Enrutamiento**

En el encaminamiento estático, las entradas en la tabla de encaminamiento se definen manualmente.

Cualquier máquina IP puede ser o no un router:

* Si NO lo es, los datagramas IP que recibe que son para ella, se descartan.
* Si SÍ lo es, se tratan de enrutar, es decir, se intenta reenviar a su destino.

Cuando una máquina quiere envíar un datagrama IP a un destino, consulta su tabla de encaminamiento. En la tabla se busca si encaja la IP destino en la primera columna de alguna entrada. Si no existe ninguna entrada adecuada, el datagrama se descarta.

Orden de búsqueda en la tabla:

* Entrada con una ip igual a la ip destino.
* Entrada con una dirección de red igual a la ip destino.
* Una entrada por defecto.

### **5.3. Datagrama IPv4**

![tcp-ip](/images/proyectos/apuntes/datagrama-ipv4.png/)

Un datagrama ip tiene fundamentalmente dos partes, la cabecera y los datos que vienen del nivel superior.

Aspectos relevantes:

* Tiene mínimo 20 octetos.
* Especifíca el protocolo de lo que está encapsulado dentro.
* Contiene la ip origen y la ip destino, ambos campos ocupan 32 bits.

### **5.4. ICMP**

El protocolo ICMP permite la comunicación de errores y para que los nodos de la red hagan preguntas.

Los mensajes ICMP se transmiten encapsulados en datagramas IP.

![tcp-ip](/images/proyectos/apuntes/icmp.png/)

### **Encapsulamiento**

![tcp-ip](/images/proyectos/apuntes/encapsulamiento.png/)

## **6. Nivel de transporte**

El nivel de transporte proporciona al nivel superior una comunicación individual para cada aplicación entre los equipos origen y destino. También segmenta los datos cuando es necesario.

Ensambla los segmentos en destino para ofrecerlos al nivel superior y controla el flujo de datos.

Los principales protocolos de nivel de transporte: UDP y TCP.

### **6.1. Puertos**

Un puerto es una dirección lógica dentro de una máquina. Esta formado por 16 bits en decimal.

Los puertos por debajo de 1024 son privilegiados. También los puertos son diferentes para UDP y TCP.

### **6.2. UDP**

Las siglas significan User Datagram Protocol.

Es un protocolo simple, lo utilizan las aplicaciones con requisitos sencillos.

Es un protocolo no orientado a conexión, es decir, no hay un paso previo al envío de datos. Tampoco es fíable ni proporciona control de flujo.

#### **Cabecera UDP**

![tcp-ip](/images/proyectos/apuntes/cabecera-udp.png/)

*Un ejemplo de uso de UDP es el servicio DNS ya que este proceso petición-respuesta es esencial la velociad.*

### **6.3. TCP**

Las siglas son Transmission Control Protocol.

Es un protocolo orientado a conexión, es decir, tiene un paso previo en el que se fija la conexión entre los puertos origen y destino, a partir de ese momento se inicia la comunicación.

Este protocolo es fiable y tiene caracteristica full duplex, es decir, ambos pueden envíar y recibir a la vez. También proporciona control de flujo.

#### **Cabecera TCP**

![tcp-ip](/images/proyectos/apuntes/cabecera-tcp.png/)

* Puerto orígen y puerto destino
* Número de secuencia y número de asentimiento que garantizan la fíabilidad de la conexión.

## **7. Nivel de aplicación**

En el nivel de aplicación se encuentran las distintas aplicaciones que disponemos.

En la actualidad cada vez tenemos menos usos de aplicaciones de forma directa y accedemos a ellas a través de una plataforma web.

*Por ejemplo antiguamente para transferir ficheros se utilizaba el protocolo FTP, hoy en día para transferir ficheros se utiliza la vía web.*

Por tanto, cada vez es más habitual utilizar equipos que practicamente no tienen ningún tipo de aplicaciṕn instalada y que tienen un navegador y a través del protocolo HTTP utiliza el resto de protocolos.

Desde el punto de vista de la red terminamos construyendo un paquete para transmitir entre el cliente y el servidor.