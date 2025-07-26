# Capítulo 5.3: Ataques Wi-Fi - WEP (Wired Equivalent Privacy)

Este documento aborda el protocolo WEP (Wired Equivalent Privacy), diseñado para proporcionar seguridad en redes LAN inalámbricas, pero que resultó ser vulnerable. Se explican los conceptos clave de WEP, como el Vector de Inicialización (IV), el cifrado RC4 y el Valor de Comprobación de Integridad (ICV). Además, se detallan los pasos para capturar y descifrar datos con herramientas como airodump-ng y aircrack-ng, y se describen métodos para acelerar la recopilación de paquetes IV, como la autenticación falsa, los ataques de repetición de solicitudes ARP y los ataques de desautenticación.

***

## **Ataques WEP**

**WEP, Privacidad Equivalente por Cable (en inglés,**&#x57;ired Equivalent Privac&#x79;**)**, es un protocolo de cifrado muy débil, ya que fue el primer intento por cifrar las comunicaciones Wi-Fi en 1999. Fue diseñado para proporcionar a una red LAN inalámbrica un nivel de seguridad y privacidad comparable al de una LAN cableada. Fue introducido como parte del estándar IEEE 802.11, pero se identificaron numerosas vulnerabilidades en WEP, lo que llevó a su reemplazo por protocolos más seguros como WPA en 2003 y posteriormente WPA2 en 2004.

Aquí están los conceptos teóricos clave involucrados en WEP:

* **Vector de Inicialización (**&#x49;&#x56;**):** Un valor aleatorio utilizado junto con una clave secreta para cifrar paquetes. El IV de WEP tiene solo 24 bits de longitud, lo que significa que se reutiliza frecuentemente en redes ocupadas.
* **Cifrado de Flujo RC4:** WEP utiliza el cifrado de flujo RC4 para el cifrado. Las fallas de implementación de RC4 dentro de WEP facilitan la predicción del flujo de clave, comprometiendo la seguridad.
* **Valor de Comprobación de Integridad (ICV):** WEP emplea un ICV para verificar la integridad de los paquetes utilizando CRC-32, un Checksum no criptográfico que es vulnerable a ciertos tipos de ataques.
* **Autenticación de Clave Compartida (SKA)**: WEP utiliza un mecanismo de autenticación denominado Shared Key Authentication (SKA), en el que los dispositivos deben compartir la misma clave estática para autenticarse. Sin embargo, esta autenticación es insegura, ya que expone partes del proceso de cifrado, permitiendo que un atacante deduzca la clave.

### Captura y Descifrado

Una vez que haya reunido suficiente información, se pueden realizar ataques con los siguientes pasos:

* **Identificar Tráfico de Red:** Usando herramientas como airodump-ng, comience a capturar paquetes en la red objetivo para recopilar IVs.
* **Analizar Datos Capturados:** Use aircrack-ng o herramientas similares para analizar los datos capturados e intentar descifrar la clave WEP.

Cuantos más paquetes de datos capture, mayores serán las posibilidades de encontrar IVs que se repitan, lo que facilita el descifrado.

### Aceleración de recopilación de paquetes IV

#### Autenticación Falsa

Para acelerar la recopilación de IV, los ataques de autenticación falsa (fake authentication attack) son efectivos:

* **Enviar solicitudes de autenticación falsa** utilizando herramientas como aireplay-ng. Esto engaña al AP haciéndole creer que su dispositivo es un cliente auténtico.

```
aireplay-ng -1 0 -a -h  <INTERFAZ>
```

#### Ataque de Repetición de Solicitudes ARP (arpreplay)

Otra forma de acelerar la generación de IV es a través de ataques de repetición de solicitudes ARP

* **Capturar paquetes ARP válidos**, luego reinyéctelos repetidamente en la red.
* **Monitorizar la tasa de generación de IV**, que debería aumentar debido al tráfico inducido.

```
aireplay-ng -3 -b -h  <INTERFAZ>
```

#### Ataque de Desautenticación

Los ataques de desautenticación fuerzan la desconexión temporal de los clientes legítimos de la red, lo que les obliga a reconectarse y autenticarse nuevamente. Esto genera nuevo tráfico, incluyendo solicitudes ARP.

```
aireplay-ng -0 1 -a -c  <INTERFAZ>
```
