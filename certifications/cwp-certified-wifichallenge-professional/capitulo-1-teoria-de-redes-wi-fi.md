# Capitulo 1: Teoría de redes Wi-Fi

## Diagrama básico de Wi-Fi

Una red Wi-Fi básica está dividida en 2 tipos de dispositivos. El **punto de acceso (en inglés, Access Point,**&#x41;&#x50;**)** y los **clientes o estaciones (en inglés,** client **o** STATION&#x53;**)**. En un entorno doméstico lo habitual es que haya 1 AP emitiendo 1 red y múltiples clientes conectados a él.

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1).png" alt=""><figcaption><p>Diagrama de red Wi-Fi doméstica</p></figcaption></figure>

Y en el caso de las redes corporativas, lo habitual es que haya un gran número de APs emitiendo la misma red y un gran número de clientes conectados a estos APs.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Diagrama de red Wi-Fi corporativa</p></figcaption></figure>

Este gran número de APs es invisible de cara al usuario, ya que todas emiten la misma red, por lo que los dispositivos la muestran como una única red.

## IEEE 802.11

Las características clave de los estándares Wi-Fi 802.11 incluyen:

* **Bandas de Frecuencia**: Wi-Fi opera en las bandas de frecuencia de 2.4 GHz y 5 GHz, con la banda de 6 GHz añadida en la versión 802.11ax (también conocida como Wi-Fi 6E) y utilizada también en Wi-Fi 7 o 802.11be. Cada banda tiene sus ventajas, con 2.4 GHz ofreciendo un mejor alcance, pero potencialmente más interferencia de otros dispositivos, y 5 GHz proporcionando tasas de datos más rápidas a distancias más cortas.
* **Tasas de Transferencia de Datos**: Cada versión ha aumentado las tasas potenciales de transferencia de datos. Por ejemplo, 802.11b ofrecía hasta 11 Mbps, 802.11g hasta 54 Mbps, 802.11n hasta 600 Mbps, 802.11ac (Wi-Fi 5) supera el Gbps, y 802.11ax (Wi-Fi 6) mejora aún más las velocidades y la eficiencia. Aunque la transmisión sigue siendo de medio dúplex, Wi-Fi 6 introduce la tecnología OFDMA (Acceso Múltiple por División de Frecuencia Ortogonal), que mejora la eficiencia de la red al permitir que múltiples dispositivos compartan canales de manera más efectiva
* **Seguridad**: La seguridad del Wi-Fi ha evolucionado para proteger los datos de los usuarios. Protocolos como WEP (Wired Equivalent Privacy), WPA (Wi-Fi Protected Access), WPA2 y el más reciente WPA3 ofrecen diferentes niveles de seguridad, siendo WPA3 el más seguro, proporcionando protecciones robustas contra varios tipos de ataques.
* **Acceso**: Las redes Wi-Fi pueden ser accedidas a través de puntos de acceso (AP) que emiten la señal a la que los dispositivos se conectan. Estas redes pueden configurarse en diversas topologías, desde configuraciones sencillas en el hogar con un solo AP hasta configuraciones empresariales complejas que usan múltiples AP para cubrir grandes áreas.
* **Aplicaciones**: La tecnología Wi-Fi se utiliza en una amplia gama de aplicaciones más allá del acceso a internet, incluidas la transmisión de medios, los videojuegos, los dispositivos inteligentes para el hogar e incluso en sistemas industriales, de salud y de transporte para la recopilación y comunicación de datos.

### Terminología básica

* **WLAN (Wireless Local Area Network)**: Una red que permite a los dispositivos conectar y comunicarse de forma inalámbrica dentro de un área limitada.
* **Punto de Acceso (en inglés, Access Point, AP)**: Un dispositivo que crea una red local inalámbrica (WLAN) dentro de un área específica, permitiendo que los dispositivos Wi-Fi se conecten a la red local y a internet.
* **MAC (Media Access Control)**: Un identificador único asignado a las interfaces de red para las comunicaciones en la capa de enlace de datos de un segmento de red. Una parte es fija e identifica al fabricante y el resto es aleatorio, normalmente la primera mitad indica el fabricante y el resto el dispositivo, pero no siempre es así.
* **OUI (Organizationally Unique Identifier)**: Es un código de 24 bits (3 bytes) que forma parte de la dirección MAC. Es asignado por el IEEE a un fabricante específico, permitiendo identificar a los fabricantes de dispositivos de red. La dirección MAC completa suele estar compuesta por la OUI (los primeros 24 bits) seguida de un identificador específico del dispositivo asignado por el fabricante (los últimos 24 bits).
* **BSSID (Basic Service Set Identifier)**: La dirección MAC del punto de acceso inalámbrico (AP). Cada dirección es única. (Ejemplo: F0:9F:C2:71:22:12)
* **SSID (Service Set Identifier)**: El nombre de una red Wi-Fi, permitiendo a los usuarios identificar y conectarse a la red deseada. (Ejemplo: wifi-guest)
* **ESSID (Extended Service Set Identifier)**: Versión extendida de SSID, siendo prácticamente lo mismo que el SSID, usado indistintamente para referirse al nombre de la red en entornos con múltiples puntos de acceso.
* **ESTACIÓN (en inglés, STATION)**: Un dispositivo (como un ordenador portátil, teléfono inteligente, etc.) que se conecta y comunica a través de una red inalámbrica ofrecida por un punto de acceso (AP) cercano.
* **Canal (en inglés, Channel, CH)**: Un rango específico de frecuencias dentro de la banda de frecuencia, permitiendo que múltiples redes operen simultáneamente sin interferencias.
* **Protocolo de Seguridad**: Conjunto de protocolos criptográficos y métodos de autenticación diseñados para asegurar las redes inalámbricas al garantizar tanto la privacidad como la integridad de los datos transmitidos a través de la red. (Ejemplo: WEP, WPA, WPA2, WPA3)
* **Cifrado**: El proceso de codificar mensajes o información de tal manera que solo las partes autorizadas puedan leerlo; específicamente utilizado en Wi-Fi para asegurar los datos transmitidos entre dispositivos y la red. (Ejemplos: WEP usa RC4, WPA usa TKIP, WPA2 usa TKIP y AES-CCMP y WPA3 usa GCMP)
* **Autenticación**: El proceso de verificar la identidad de un dispositivo o usuario antes de permitir el acceso a la red Wi-Fi. (Ejemplos: PSK en WPA/WPA2, EAP en WPA2 Enterprise o SAE para WPA3)

### Generaciones Wi-Fi

| Wi-Fi 1 (802.11b)  | 1999            | 2.4 GHz               | Hasta 11 Mbps   |
| ------------------ | --------------- | --------------------- | --------------- |
| Wi-Fi 2 (802.11a)  | 1999            | 5 GHz                 | Hasta 54 Mbps   |
| Wi-Fi 3 (802.11g)  | 2003            | 2.4 GHz               | Hasta 54 Mbps   |
| Wi-Fi 4 (802.11n)  | 2009            | 2.4 GHz, 5 GHz        | Hasta 600 Mbps  |
| Wi-Fi 5 (802.11ac) | 2014            | 5 GHz                 | Hasta 3.46 Gbps |
| Wi-Fi 6 (802.11ax) | 2019            | 2.4 GHz, 5 GHz        | Hasta 9.6 Gbps  |
| Wi-Fi 6E           | 2020            | 6 GHz                 | Hasta 9.6 Gbps  |
| Wi-Fi 7 (802.11be) | 2024            | 2.4 GHz, 5 GHz, 6 GHz | Hasta 46.1 Gbps |
| Wi-Fi 8 (802.11bn) | 2028 (estimado) | TBA                   | TBA             |

### Bandas de Espectro en Comunicaciones Wi-Fi

### Banda de 2.4 GHz

La banda de 2.4 GHz se usa ampliamente y es compatible con todas las generaciones de Wi-Fi. Ofrece un buen rango pero es más susceptible a interferencias de otros dispositivos, como hornos microondas y dispositivos Bluetooth.

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Banda de 5 GHz

La banda de 5 GHz, utilizada por Wi-Fi 2, Wi-Fi 4, Wi-Fi 5 y Wi-Fi 6, proporciona tasas de datos más rápidas pero a un rango más corto en comparación con 2.4 GHz. Está menos congestionada y, por lo tanto, es menos propensa a interferencias.

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### Banda de 6 GHz

Introducida con Wi-Fi 6E, la banda de 6 GHz ofrece más ancho de banda y canales, reduciendo significativamente la interferencia y mejorando las tasas de datos y la latencia. Esta banda es especialmente beneficiosa para aplicaciones de alto ancho de banda en entornos con muchos dispositivos.

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Sondas, autenticación y asociación

### Ejemplo real

Este diagrama muestra los pasos que se siguen para conectar una estación (como un portátil o smartphone) a un Punto de Acceso (AP, como un router) en una red Wi-Fi.

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Diagrama de mensajes entre cliente y AP</p></figcaption></figure>

* **Mensaje de Beacon** : El Punto de Acceso (AP) envía señales periódicas llamadas beacons para anunciar su presencia y las características de la red a todas las estaciones cercanas.
* **Mensaje de Solicitud de Probe**: Una estación envía una solicitud llamada probe request al AP, solicitando más detalles sobre la red específica a la que desea conectarse.
* **Mensaje de Respuesta de Probe**: El AP responde con un probe response, proporcionando información adicional necesaria para que la estación decida si desea conectarse a esa red.
* **Mensaje de Solicituddeautenticación**: La estación envía una solicitud de autenticación al AP, que incluye una clave cifrada con la clave pública del AP, para verificar su identidad.
* **Mensaje de Respuestadeautenticación**: El AP verifica la solicitud y responde con una autenticación cifrada, confirmando que la estación está autorizada para conectarse.
* **Mensaje de Solicitud de Asociación**: Después de autenticarse correctamente, la estación envía una solicitud para asociarse formalmente con la red.
* **Mensaje de Respuesta de Asociación**: El AP responde confirmando la asociación, lo que permite a la estación acceder a la red y utilizar sus recursos.
* **Handshake de PSK**: Si la red utiliza una clave precompartida (PSK), se realiza un proceso de verificación para asegurar que la clave es correcta y establecer una clave temporal para una conexión segura.

***

## Tipos de Redes por Cifrado

### Autenticación Abierta (OPN)

La Autenticación Abierta (OPN) se refiere a una red donde no existen mecanismos significativos de autenticación para los usuarios que intentan conectarse a ella. Esencialmente, es una red abierta, lo que significa que cualquiera puede conectarse sin necesidad de una contraseña. Este tipo de red ofrece la máxima conveniencia, a costa de la seguridad. Se encuentran principalmente en lugares públicos como cafeterías, bibliotecas o aeropuertos, donde existe la necesidad de acceso a internet de terceros de forma inmediata. Sin embargo, debido a que no hay cifrado, todos los datos transmitidos pueden ser potencialmente interceptados por otros usuarios en el mismo lugar, sin estar siquiera en la misma red. A pesar de sus riesgos, OPN sigue siendo popular por su simplicidad, facilidad de acceso y mantenimiento. En ocasiones se encuentra un portal cautivo una vez conectado a la red, bloqueando el acceso a internet tras un login o una pasarela de pago. Un **portal cautivo** es una página web a la que los usuarios de una red Wi-Fi abierta son redirigidos antes de acceder a Internet. Este portal requiere que los usuarios completen alguna acción, como autenticarse con un nombre de usuario y contraseña, aceptar términos y condiciones, pagar una tarifa, o proporcionar información personal. Su propósito es controlar y gestionar el acceso a la red, permitiendo a los operadores recopilar datos de usuarios y/o monetizar el servicio.

### Opportunistic Wireless Encryption (OWE)

El Cifrado Oportunista Inalámbrico (en inglés, Opportunistic Wireless Encryption, OWE) representa un avance significativo en el equilibrio entre conveniencia y seguridad en las redes inalámbricas abiertas. OWE proporciona comunicación cifrada en Wi-Fi sin la necesidad de una contraseña compartida. Esto significa que, a diferencia de las redes abiertas tradicionales, OWE impide que un atacante pueda escuchar los datos transmitidos entre un dispositivo y el punto de acceso Wi-Fi. Lo hace estableciendo automáticamente una conexión segura que es transparente para el usuario. OWE es particularmente importante para mejorar la seguridad en puntos de acceso Wi-Fi públicos, ofreciendo un nivel de protección sin complicar el proceso de conexión para los usuarios.

### Wired Equivalent Privacy (WEP)

La Privacidad de Comunicación Equivalente por Cable (en inglés, Wired Equivalent Privacy, WEP) fue uno de los primeros protocolos de seguridad introducidos para redes inalámbricas, diseñado para proporcionar un nivel de seguridad comparable al de una red cableada. Para esto utiliza una contraseña precompartida entre el AP y sus clientes, siendo de 10 caracteres de longitud y en hexadecimal. Sin embargo, con el tiempo, se descubrieron varias vulnerabilidades significativas en WEP, lo que hacía posible que los atacantes determinados rompieran el cifrado en cuestión de minutos. Como resultado, WEP se ha vuelto prácticamente obsoleto, reemplazado por protocolos más seguros como WPA, WPA2 y, por último, WPA3.

### Pre-Shared Key (PSK)

La Clave Pre Compartida (en inglés, Pre-Shared Key, PSK) es un modo de seguridad utilizado junto con protocolos como WPA y WPA2 (Wi-Fi Protected Access) para autenticar a los usuarios en una red inalámbrica. En PSK, se establece una clave compartida o contraseña en el punto de acceso de la red y cualquier dispositivo que desee conectarse debe proporcionar esa clave. Este método es ampliamente utilizado en redes domésticas y de pequeñas oficinas debido a su simplicidad. Aunque es significativamente más seguro que WEP, la fortaleza de una red PSK depende en gran medida de la complejidad y el secreto de la clave compartida. Si la clave es demasiado simple o se conoce por terceros no autorizados, la seguridad de la red puede verse comprometida, principalmente debido a ataques de fuerza bruta offline. En estas redes, una vez conocida la contraseña, es posible descifrar el tráfico de los usuarios, casi como en una red OPN.

### Simultaneous Authentication of Equals (SAE)

La Autenticación Simultánea de Iguales (en inglés, Simultaneous Authentication of Equals, SAE) es un protocolo de seguridad diseñado para mejorar la protección de las redes inalámbricas. Es un componente clave de WPA3, la última certificación de seguridad de la Wi-Fi Alliance. SAE mejora el método PSK al utilizar un proceso de Handshake más robusto que ofrece mejor protección contra ataques de diccionario offline, donde un atacante intenta adivinar la contraseña de la red intentando miles o millones de posibilidades. Esto hace que las redes SAE sean significativamente más seguras, especialmente en entornos donde la protección fuerte y resistente es primordial.

### WPA2/WPA3 Transitional

WPA2/WPA3 Transitional es un modo híbrido diseñado para facilitar la transición de las redes inalámbricas desde WPA2 a WPA3. Este modo permite que tanto dispositivos antiguos que solo soportan WPA2 como dispositivos modernos compatibles con WPA3 se conecten a la misma red. En las configuraciones de WPA2/WPA3 Transitional, los puntos de acceso emiten tanto el estándar WPA2 como el WPA3, proporcionando una compatibilidad amplia mientras se adopta la seguridad mejorada de WPA3.

### WPA/WPA2/WPA3-Enterprise (MGT)

WPA/WPA2/WPA3-Enterprise, a menudo referido usando el acrónimo MGT (de Management), es una versión de WPA específicamente diseñado para entornos corporativos o empresariales. A diferencia de su contraparte PSK (Clave Pre Compartida), WPA/WPA2/WPA3-Enterprise está diseñado para proporcionar un nivel de seguridad más alto mediante el uso de mecanismos de autenticación 802.1X. Este método se integra con un sistema de autenticación basado en servidor, típicamente un servidor RADIUS (Servicio de Usuario Remoto de Autorización y Autenticación), para gestionar el acceso individual de los usuarios a la red.

La ventaja principal de usar WPA/WPA2/WPA3-Enterprise reside en su capacidad para autenticar a los usuarios de manera individual, en lugar de depender de una clave compartida. Cada usuario obtiene acceso basado en credenciales únicas, como un nombre de usuario y contraseña o un certificado digital. Esta configuración mejora significativamente la seguridad al asegurar que la red permanezca protegida incluso si las credenciales de un individuo se ven comprometidas, ya que pueden ser revocadas sin afectar el acceso de otros usuarios.

Además, WPA/WPA2/WPA3-Enterprise proporciona un marco para emplear protocolos de cifrado avanzados y estrategias de gestión de claves. Soporta el uso de claves dinámicas, que se cambian regularmente y son únicas para cada sesión de usuario, reduciendo en gran medida el potencial de compromiso de claves. Adicionalmente, esta configuración permite el cifrado de tramas de gestión, que son cruciales para realizar tareas de gestión de la red, pero han sido históricamente vulnerables a ataques. Al asegurar estas tramas, WPA/WPA2/WPA3-Enterprise mejora aún más la integridad y la confidencialidad de los datos en la red.

Sin embargo, la complejidad de desplegar y gestionar una red WPA/WPA2/WPA3-Enterprise, incluyendo el requisito de un servidor de autenticación y la gestión de certificados, lo hace menos común en redes corporativas pequeñas donde se acaba utilizando WPA2-PSK debido a su simplicidad. Esta complejidad conlleva también que un gran número de redes corporativas sean mucho menos seguras que las redes PSK normales, ya que a pesar de poder ser mucho más seguras o incluso siendo imposible de explotar sin información interna, normalmente se configurara rápidamente y sin tener en cuenta la seguridad, permitiendo a los atacantes explotar fácilmente las redes MGT mal configuradas.
