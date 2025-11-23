# Capítulo 5.0: Recon Wi-Fi Ofensivo

### Introducción

El reconocimiento Wi-Fi ofensivo implica el uso de herramientas y técnicas especializadas para interceptar, capturar y analizar transmisiones de datos inalámbricas sin interactuar activamente con los objetivos o interactuando lo mínimo posible. Al comprender las configuraciones de la red, los dispositivos conectados, las redes ocultas y el tráfico entre dispositivos, se hace más fácil detectar debilidades de seguridad. Para este capítulo, nuestras herramientas principales serán `airodump-ng` y `wifi_db.`

## Modo Monitor y Captura de Paquetes <a href="#modo-monitor-y-captura-de-paquetes" id="modo-monitor-y-captura-de-paquetes"></a>

### Configurando el Modo Monitor

Para capturar cualquier dato útil, nuestra interfaz inalámbrica debe estar configurada en modo monitor. El modo monitor permite que la tarjeta inalámbrica capture paquetes de cualquier red, no solo de aquellas a las que estamos conectados.

Este paso lo hemos visto en el apartado anterior.

```
sudo airmon-ng start wlan0
```

### Usando airodump-ng

airodump-ng es una herramienta para escanear redes inalámbricas y capturar paquetes. En el apartado anterior, hemos visto por encima la suite aircrack-ng donde se encuentra airodump-ng, ahora vamos a ver un caso de uso de esta herramienta en la fase de reconocimiento.

* **Iniciar la Captura de Datos:**

Para iniciar una captura de paquetes amplia a través de todos los canales:

```
sudo airodump-ng wlan0mon
```

Esto proporciona datos en tiempo real sobre las redes dentro del rango, incluyendo el BSSID de cada red (dirección MAC del punto de acceso), ESSID (nombre de la red si está siendo transmitido), canal, tipo de cifrado, fuerza de la señal y cuenta de clientes.

* **Obtener Información de Todos los Canales:**

El problema de la orden anterior es que solo escanea en 2.4 GHz por defecto, por lo que es necesario indicarle que utilice todas las bandas para que escanee también en 5 GHz.

```
sudo airodump-ng --band bag -w [outputfile] --gpsd wlan0mon
```

* \--band bag: Escanear todos los canales de 2.4 GHz y 5 GHz. (Actualmente airodump-ng no dispone de un modo para escanear todas las bandas incluyendo 6 GHz, ya que es algo nuevo y con poco uso)
* -w \[outputfile]: Nombre del archivo para guardar los paquetes capturados. Es importante guardar las capturas para así poder procesar la información posteriormente y no perder información relevante. Por ese motivo salvo que el número de APs sea demasiado grande tampoco recomiendo utilizar filtros por BSSID o por ESSID. Al nombre que le pongamos se le concatena -01, aumentándose el contador cada vez que se ejecuta para no sobrescribirse.
* \--gpsd: Opcional, activa el uso de GPSD para registrar las coordenadas GPS de los puntos de acceso detectados. GPSD es un servicio que puede proporcionar datos de posición desde un receptor GPS conectado al sistema. Para que esta opción funcione correctamente, GPSD debe estar instalado y configurado en el sistema.
* **Enfocarse en un Canal Específico:**

Para enfocar la captura a un canal especifico lo podemos indicar:

```
sudo airodump-ng -c [channel] -w [outputfile] wlan0mon
```

*
  * -c \[channel]: Canal de la red objetivo.
  * -w \[outputfile]: Nombre del archivo para los paquetes capturados.

Si necesitamos enfocar el escaneo solo en un BSSID podemos utilizar el parámetro: --bssid \[BSSID]

### Interfaz de airodump-ng explicada

En la siguiente imagen se puede ver una captura de airodump-ng tomada del primer Challenge del laboratorio, escaneando en todos los canales. En esta captura se pueden distinguir dos áreas diferentes: la mitad superior, donde se encuentran los puntos de acceso (APs) detectados, y la parte inferior, donde se muestra la información de los clientes detectados durante el escaneo.

<figure><img src="../../../.gitbook/assets/image (29) (1) (1).png" alt=""><figcaption></figcaption></figure>





### Airodump-ng avanzado (ordenar, seleccionar y colorear) <a href="#airodump-ng-avanzado-ordenar-seleccionar-y-colorear" id="airodump-ng-avanzado-ordenar-seleccionar-y-colorear"></a>

En airodump-ng es posible clasificar los puntos de acceso (AP) y los clientes, seleccionar un AP específico e incluso aplicar colores a cada AP y sus clientes asociados. Esto facilita la identificación de los distintos AP, y sus clientes asociados.

El proceso en sí es bastante sencillo. El primer paso recomendado es ordenar los AP y clientes por el número de paquetes de Datos. Esto ordena todo en función de la cantidad de tráfico real que emite cada uno, de forma que los primeros AP y clientes que aparecen son los que más tráfico tienen (que suelen ser los que nos interesan). Para ello sólo tenemos que pulsar s (minúscula) hasta que el texto muestre ordenar por número de paquetes de datos (sorting by number of data packets):

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Para entrar en modo selección, basta con pulsar TAB. Una vez en modo selección, podemos navegar por los AP utilizando las teclas de flecha y ver sus clientes resaltados al mismo tiempo.

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Cuando tienes un AP seleccionado, puedes usar la tecla m para recorrer los diferentes colores soportados por airodump-ng. Esto nos permite tener capturas como las siguientes:

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## Analizando los Datos Capturados - wifi\_db

Con la opción -w en airodump-ng, los paquetes capturados se guardan en un archivo para un análisis más profundo usando herramientas como Wireshark o wifi\_db. En estas capturas se puede obtener:

* Beacon&#x73;**:** Anuncios de puntos de acceso.
* Probe&#x73;**:** Solicitudes de clientes buscando redes.
* Tramas de Autenticación/Asociació&#x6E;**:** Partes clave del proceso de conexión.

### Procesando Capturas Usando wifi\_db

[wifi\_db](https://github.com/r4ulcl/wifi_db) es una herramienta en Python3 para importar archivos generados con airodump-ng en una base de datos SQLite, ofreciendo capacidades de analítica avanzadas. Una vez en formato de base de datos, es más fácil encontrar relaciones entre AP y clientes, obtener handshakes de redes PSK, obtener identidades en redes empresariales y exponer potencialmente redes no seguras a las que se conectan dispositivos de empleados.

#### Uso

El primero paso es capturar los datos necesarios con airodump-ng guardando el output como hemos hecho en los pasos anteriores. Pero antes de eso es recomendable crear una carpeta en el home del usuario donde guardar todas las capturas.

```
mkdir ~/wifi
```

Una vez tenemos la carpeta ejecutamos el escaneo.

```
sudo airodump-ng wlan0mon -w /home/user/wifi/scan --gpsd
```

Guarda los archivos capturados en el directorio especificado (por ejemplo, \~/wifi/), luego impórtalos en wifi\_db:

```
#Acceder a la carpeta (como root)
cd /root/tools/wifi_db
# Ejecutar wifi_db
python3 wifi_db.py -d database.sqlite /home/user/wifi/
```

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* -d Nombre del archivo de la base de datos SQLite donde queremos que se guarde el resultado. Podemos ejecutar wifi\_db múltiples veces con la misma base de datos y se suma la información.
* \~/wifi/ Ruta donde se encuentran los archivos de captura de airodump-ng.

### **Ejemplo de Uso en Docker**

Alternativamente, podemos utilizar Docker para ejecutar wifi\_db sin necesidad de descargar las dependencias, aunque para esto necesitamos tener Docker instalado:

```
# Descargar la imagen de Docker
docker pull r4ulcl/wifi_db

# Especificar la carpeta local con las capturas
CAPTURESFOLDER=/home/user/wifi/
# Generar el fichero de la base de datos de salida
touch db.SQLITE

# Ejecutar el docker con los parámetros necesarios
docker run -t -v $PWD/db.SQLITE:/db.SQLITE -v $CAPTURESFOLDER:/captures/ r4ulcl/wifi_db
```

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Accede a la base de datos con un cliente como SQLite browser utilizando la interfaz gráfica:

```
sqlitebrowser database.sqlite
```

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
SQLitebrowser con wifi_db
```

Una vez abierto es necesario ir a la sección de Browse Data y seleccionar la base de datos del desplegable, a continuación, se puede ver un ejemplo de tabla ProbeClientsConnected, este ejemplo lista las solicitudes de los clientes a varias redes:

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### Uso en CLI

También es posible ver la información de la base de datos sin necesidad de una interfaz gráfica en el caso de no disponer de esta, al acceder al equipo o VM por SSH por ejemplo. A continuación, se puede ver como se abre la base de datos con sqlite3, se listan todas las tablas en la base de datos con .tables y se obtiene toda la información de la tabla de APs con el select.

```
sqlite3 db.SQLITE
.tables
SELECT * FROM APs;
```

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## Análisis de Redes Ocultas

Una red oculta en Wi-Fi es una red inalámbrica cuyo SSID no se emite públicamente, lo que significa que no aparece en la lista de redes disponibles cuando los dispositivos buscan conexiones inalámbricas. Para conectarse a una red oculta, se debe ingresar manualmente el SSID además de otros datos de autenticación necesarios.

### Detectando Redes Ocultas

Incluso si no transmiten su SSID, se pueden detectar las redes ocultas:

* **Detectar con** iwlis&#x74;**:**

```
sudo iwlist wlan0 scan
```

Busca entradas como ESSID:off/any, ESSID:"" o ESSID:"\x00\x00\x00\x00\x00" indicando redes ocultas. Esto ocurre porque en vez de mostrar el ESSID como las redes normales aparece una de esas opciones. El caso más fácil para atacar es en el que aparece un número X de \x00, ya que nos indica la longitud del ESSID oculto.

<figure><img src="../../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* **Detectar con airodump-ng**:

```
sudo airodump-ng wlan0mon
```

Las redes ocultas aparecen sin un ESSID pero muestran BSSID y posiblemente clientes asociados.

<figure><img src="../../../.gitbook/assets/image (18) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

En la imagen se observan dos tipos de redes ocultas. La primera no muestra la longitud de su ESSID, mientras que la segunda especifica que su longitud es de 9, lo que nos permite conocer esta información para posibles ataques.

### Obtención de un ESSID Oculto

#### Fuerza Bruta de SSIDs Ocultos

Una vez localizadas, realiza fuerza bruta de SSIDs ocultos usando MDK4. MDK4 es una herramienta de auditoría y pruebas de redes inalámbricas. Es una evolución de MDK3 y está diseñada para atacar redes Wi-Fi de diversas maneras, explotando vulnerabilidades y debilidades en la seguridad de estas redes. MDK4 puede realizar una amplia variedad de ataques, incluidos ataques de denegación de servicio (DoS), ataques de desautenticación, ataques de reinyección de paquetes y muchos otros.

```
mdk4 wlan0mon p -t [BSSID] -f [ESSID-DIC]
```

* p : Se utiliza para descubrir y probar SSID ocultos en una red Wi-Fi.
* -t \[BSSID]: Dirección MAC del punto de acceso objetivo.
* -f \[ESSID-DIC]: Ruta a la lista de palabras con posibles nombres de SSID. Este diccionario normalmente es personalizado basándose en otras redes públicas o con palabras claves posibles.

#### Reconexión de Cliente

Otra técnica consiste en esperar a que un cliente se reconecte o forzar la reconexión mediante un ataque de desautenticación. Este método funciona de la siguiente manera:

* Desconecta a los clientes del punto de acceso:
  * Realiza un ataque de desautenticación, que obliga a los clientes a desconectarse del AP.
* Monitorea los intentos de reconexión:
  * Durante los intentos de reconexión, los SSID ocultos pueden ser revelados en los paquetes del Handshake.
