# Capítulo 7: Experiencia en el mundo real

## Desautenticación en el mundo real

En auditorias de seguridad Wi-Fi, es común usar comandos como aireplay-ng -0 0 -a \[BSSID] para desautenticar a todos los dispositivos conectados a un punto de acceso, o aireplay-ng -0 0 -a \[ BSSID ] -c \[MAC] para desconectar solo a un cliente específico. Sin embargo, MDK4 ofrece una alternativa más potente y flexible. A continuación, se explica cómo hacer estos ataques con MDK4:

**Para desautenticar a todos los clientes de un AP**:

```
mdk4 wlan0 d -B [BSSID]
```

**Para desautenticar a un solo cliente específico**:

```
mdk4 wlan0 d -B [BSSID] -S [MAC]
```

Donde:

* \[BSSID] es la dirección MAC del punto de acceso objetivo.
* \[MAC] es la dirección MAC del cliente específico a desautenticar.

**Consideraciones**:

* **Probar en una Red Propia**: Es recomendable probar en una red controlada, ya que la efectividad del ataque de desautenticación puede variar.
* **Compatibilidad con WPA3 y PMF**: Las redes WPA3 con 802.11w (Protected Management Frames) impedirán este tipo de ataque. Algunas redes WPA/WPA2 también pueden tener esta protección en entornos empresariales.
* **Limitaciones de Chipset**: La eficacia del ataque puede depender del tipo de chipset Wi-Fi y de si soporta funciones como VIF (Virtual Interface Functionality).

***

## Wardriving

### Introducción al Wardriving

El término wardriving proviene de la combinación de war dialing (una técnica antigua de hacking) y conducción. Se refiere a la práctica de buscar redes inalámbricas Wi-Fi mientras se está en un vehículo en movimiento, utilizando un ordenador portátil o un smartphone para detectar estas redes. Esta actividad implica mapear la ubicación de las redes Wi-Fi, analizar sus protocolos de seguridad y, a menudo, documentar esta información. Si bien el wardriving puede realizarse con fines maliciosos, como hackear redes no seguras, también se utiliza por razones legítimas como el mapeo del rendimiento de la red, la investigación y la auditoría de seguridad.

### Cómo Funciona el Wardriving

* **Equipamiento y Software**: Las herramientas básicas para el wardriving incluyen un vehículo, un ordenador portátil o dispositivo móvil con capacidad Wi-Fi, una antena de alta ganancia para aumentar el rango de detección de señal Wi-Fi y software especializado que pueda registrar señales Wi-Fi. Herramientas de software populares incluyen NetStumbler para Windows, Kismet para Linux, y WiGLE para Android e iOS.
* **Detección y Registro**: A medida que el wardriver se desplaza, su equipo escanea continuamente en busca de señales Wi-Fi disponibles. Cuando se detecta una red, el software registra varios datos como el Identificador del Conjunto de Servicios (SSID), el tipo de cifrado (WEP, WPA, etc.), la intensidad de la señal y las coordenadas GPS si están disponibles.
* **Análisis y Mapeo**: Los datos recopilados pueden analizarse con diversos propósitos. Un uso común es mapear la distribución de redes Wi-Fi en un área, lo que puede mostrar la densidad de redes, los protocolos de seguridad predominantes y redes potencialmente vulnerables. Herramientas como WiGLE permiten a los usuarios contribuir con sus hallazgos a un mapa global de redes Wi-Fi.

### Wigle

WiGLE es una plataforma que permite a los usuarios compartir y visualizar redes Wi-Fi detectadas durante actividades de wardriving. Esta plataforma ofrece una base de datos global de ubicaciones y detalles de redes Wi-Fi, permitiendo el análisis de tendencia y la identificación de puntos débiles.

#### Registro en WiGLE

* Crear una cuenta en WiGLE.net
* Descargar las aplicaciones disponibles para Android e iOS

#### Uso de la Aplicación WiGLE

* Instalar y configurar la aplicación WiGLE en un dispositivo móvil
* Realizar escaneos mientras se conduce o se camina
* Subir los datos recopilados a la base de datos de WiGLE

#### Análisis en WiGLE

* Usar la plataforma para analizar densidades de red y tipos de cifrado utilizados
* Identificar redes no seguras o mal configuradas
* Contribuir al mapeo global de redes Wi-Fi para investigación
