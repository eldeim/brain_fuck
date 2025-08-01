# Listado de herramientas Hacking Wi‑Fi

## Selección

* [aircrack‑ng](https://www.aircrack-ng.org/): Aircrack‑ng es un conjunto completo de herramientas para evaluar la seguridad de redes Wi‑Fi. Desde poner en modo monitor y monitorizar pasivamente redes Wi‑Fi hasta ataques a dichas redes.
* [hostapd‑wpe](https://github.com/aircrack-ng/aircrack-ng/tree/master/patches/wpe/hostapd-wpe): Hostapd‑wpe es una versión modificada del software hostapd para realizar ataques EAP.
* [eaphammer](https://github.com/r4ulcl/eaphammer.git): Eaphammer es un conjunto de herramientas para realizar ataques de gemelo malvado dirigidos a redes WPA2‑Enterprise.
* [wifi\_db](https://github.com/r4ulcl/wifi_db): wifi\_db es una herramienta que transforma capturas de airodump‑ng en una base de datos estructurada, facilitando el análisis de datos de redes inalámbricas.
* [pcapFilter.sh](https://gist.githubusercontent.com/r4ulcl/f3470f097d1cd21dbc5a238883e79fb2/raw/a22ac3095e197dc97745d36ece49bb455fc6d1ae/pcapFilter.sh): PcapFilter.sh es un script para filtrar y procesar archivos cap y pcap.
* [airgeddon](https://github.com/v1s1t0r1sh3r3/airgeddon.git): Airgeddon es un script multiuso en bash para sistemas Linux que audita redes inalámbricas.
* [hostapd‑mana](https://github.com/sensepost/hostapd-mana): Hostapd‑mana es un software para ataques de punto de acceso falso que puede capturar credenciales.
* [eapeak](https://github.com/securestate/eapeak): Eapeak analiza la seguridad de las comunicaciones EAP sobre redes inalámbricas.
* [EAP\_buster](https://github.com/blackarrowsec/EAP_buster): EAP\_buster es una herramienta diseñada para verificar los mecanismos de autenticación EAP soportados.
* [reaver](https://github.com/t6x/reaver-wps-fork-t6x): Reaver es una herramienta para realizar ataques de fuerza bruta contra WPS para recuperar contraseñas Wi‑Fi.
* [wpa\_sycophant](https://github.com/sensepost/wpa_sycophant): wpa\_sycophant es una herramienta diseñada para retransmitir (relay) mensajes EAP en una red, lo que permite acceder a redes MGT sin crackear la contraseña.
* [berate\_ap](https://github.com/sensepost/berate_ap): Berate\_ap es una herramienta para configurar puntos de acceso falso y manipular el tráfico inalámbrico.
* [air‑hammer](https://github.com/Wh1t3Rh1n0/air-hammer): Air‑hammer es una herramienta de automatización para llevar a cabo ataques de autenticación forzada y ataques EAP online directamente contra el AP.
* [hcxtools](https://github.com/ZerBea/hcxtools.git): Hcxtools es un conjunto de herramientas para capturar y convertir paquetes de tráfico WLAN.
* [wifiphisher](https://github.com/wifiphisher/wifiphisher.git): Wifiphisher es una herramienta de seguridad que monta ataques de phishing automatizados y personalizados para las víctimas en clientes Wi‑Fi.

### Otras opciones

* [wifite2](https://github.com/derv82/wifite2.git): Wifite2 automatiza la auditoría de redes inalámbricas atacando eficientemente redes encriptadas con WEP, WPA y WPS usando varias herramientas.
* [fluxion](https://www.github.com/FluxionNetwork/fluxion.git): Fluxion realiza análisis Wi‑Fi con ataques de gemelo malvado y phishing para evaluar vulnerabilidades de la red.
* [kismet](https://www.kismetwireless.net/): Kismet es un detector de redes y sniffer de paquetes que identifica el tráfico de la red y redes inalámbricas ocultas.
* [create\_ap](https://github.com/oblique/create_ap): create\_ap permite la creación de puntos de acceso virtuales en Linux para redes inalámbricas rápidas y temporales.
* [wifipumpkin3](https://github.com/P0cL4bs/wifipumpkin3.git): Wifipumpkin3 es un marco flexible para pruebas de seguridad Wi‑Fi, soportando ataques de punto de acceso falso.
* [wacker](https://github.com/blunderbuss-wctf/wacker): Wacker es una herramienta para romper contraseñas de redes WPA/WPA2/WPA3, utilizando métodos de diccionario o fuerza bruta online directamente contra el AP.
* [mdk4](https://github.com/aircrack-ng/mdk4): mdk4 es la versión mejorada de mdk3, con más modos de ataque y optimizaciones para redes Wi‑Fi.
* [PixieWPS](https://github.com/wiire-a/pixiewps): PixieWPS explota la vulnerabilidad Pixie Dust en implementaciones de WPS.
* [bully](https://github.com/aanarchyy/bully): bully automatiza ataques de fuerza bruta contra WPS.
* [fern-wifi-cracker](https://github.com/savio-code/fern-wifi-cracker): Fern-Wifi-Cracker ofrece una interfaz gráfica para automatizar ataques a redes Wi‑Fi.
* [UnicastDeauth](https://github.com/mamatb/UnicastDeauth): UnicastDeauth permite enviar paquetes de desautenticación unicast dirigidos a estaciones específicas en redes Wi‑Fi.
* [EAPgrade](https://github.com/mamatb/EAPgrade): EAPgrade evalúa la seguridad de métodos de autenticación EAP en redes inalámbricas, proporcionando un informe de fortalezas y debilidades.

Esta sección se irá ampliando según vaya utilizando nuevas herramientas.

***

## Aircrack-ng

### Introducción

aircrack-ng es una colección de herramientas para la auditoría de seguridad de redes inalámbricas 802.11. Es conocido por su capacidad para romper contraseñas WEP y WPA/WPA2-PSK, entre otras funcionalidades.

### Instalación

Para instalar aircrack-ng en sistemas basados en Ubuntu/Debian:

```
sudo apt-get update
sudo apt-get install aircrack-ng
```

### Uso ultra básico

**1.Activar Modo Monitor**:

```
sudo airmon-ng start wlan0
```

**2.Capturar Paquetes con** airodump-ng:

```
sudo airodump-ng wlan0mon
```

Aquí, wlan0mon es el nombre de la interfaz en modo monitor.

**3.Romper Contraseña con** aircrack-ng: Una vez capturados suficientes paquetes, se puede intentar romper la contraseña utilizando una lista de palabras:

```
sudo aircrack-ng -w path/to/wordlist.txt -b xx:xx:xx:xx:xx:xx /path/to/capture.cap
```

En este comando, path/to/wordlist.txt es la ruta a la lista de contraseñas, xx:xx:xx:xx:xx:xx es el BSSID del objetivo, y /path/to/capture.cap es el archivo de captura.

### Modo monitor

**Iniciar el Modo Monitor usando** airmon-ng: El modo monitor permite que la tarjeta de red inalámbrica capture todos los paquetes que pasan cerca, sin necesidad de estar asociada a una red específica. Es esencial para realizar auditorías de seguridad y análisis de tráfico.

**Iniciar el modo monitor:**

```
sudo airmon-ng check kill
```

Este comando finaliza procesos que pueden interferir con airmon-ng, como el NetworkManager y el wpa\_supplicant.

```
sudo airmon-ng start wlan0
```

Aquí, wlan0 es el nombre de la interfaz inalámbrica que deseas poner en modo monitor. airmon-ng cambiará el nombre de la interfaz y agregará mon al final, por ejemplo, wlan0mon.

* **Volver al modo gestionado (managed):** Si deseas regresar al modo gestionado (el modo estándar de operación de las redes inalámbricas), puedes usar el siguiente comando:

```
sudo airmon-ng stop wlan0mon
```

* **Explicación:** Este comando detendrá el modo monitor de la interfaz wlan0mon y la devolverá a su estado original (wlan0).

### Inyección de Paquetes

La inyección de paquetes en una red es una técnica utilizada para probar la respuesta de la red o evaluar su seguridad haciendo ataques de desautenticación principalmente. Es fundamental asegurarse de que el dispositivo soporte esta función.

**1.Probar Soporte para Inyección**:

```
sudo aireplay-ng --test wlan0
```

Si el dispositivo soporta la inyección de paquetes, se mostrará un mensaje de confirmación.

**2.Realizar Inyección de Paquetes**: Con aireplay-ng, se pueden llevar a cabo diversas técnicas de inyección. Un ejemplo simple es la inyección de solicitudes ARP:

```
sudo aireplay-ng -3 -b [target BSSID] -h [your MAC] wlan0
```

Aquí, \[target BSSID] es el BSSID de la red objetivo, \[your MAC] es la dirección MAC de la interfaz y -3 el modo de inyección de ARP.

***

## Wireshark

### Instalación

Wireshark puede instalarse fácilmente desde los repositorios oficiales de la mayoría de las distribuciones de Linux. Aquí se muestra cómo instalarlo en sistemas basados en Ubuntu/Debian:

```
sudo apt-get update
sudo apt-get install wireshark
```

Durante la instalación, puede ser necesario configurar permisos para que usuarios no root puedan capturar paquetes. Esto se hace respondiendo según las preferencias de seguridad del usuario.

### Ejecución de Wireshark

Para iniciar Wireshark, se puede ejecutar el comando wireshark desde el terminal o buscarlo en el menú de aplicaciones del entorno de escritorio. Para ejecutarlo sin privilegios de root, puede ser necesario agregar el usuario al grupo wireshark:

```
sudo usermod -a -G wireshark $USER
```

Después, es necesario cerrar y abrir sesión para que los cambios tengan efecto.

### Uso Básico

<figure><img src="../../../.gitbook/assets/image (19) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Abrir Wireshark.
* Seleccionar la interfaz de red para capturar paquetes.
* Hacer clic en el ícono de la aleta de tiburón para iniciar la captura.
* Utilizar la barra de filtros para enfocarse en los paquetes de interés, como http para tráfico HTTP.
* Para detener la captura, pulsar el ícono de cuadrado rojo.
