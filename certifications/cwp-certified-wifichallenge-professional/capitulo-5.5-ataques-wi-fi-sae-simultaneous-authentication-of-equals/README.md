# Capítulo 5.5: Ataques Wi-Fi - SAE (Simultaneous Authentication of Equals)

La Autenticación Simultánea de Iguales (SAE) en inglés, Simultaneous Authentication of Equals, es un protocolo de autenticación y de intercambio de claves basado en contraseñas, utilizado en WPA3 (Wi-Fi Protected Access 3). Mejora la seguridad respecto a los protocolos anteriores al proporcionar resistencia a los ataques de fuerza bruta offline. SAE aprovecha la criptografía de curva elíptica para establecer una conexión segura incluso si la contraseña es débil.

## Ataques a redes WPA2/WPA3 transitional o Mixed Mode

Las redes Wi-Fi que operan bajo el modo WPA2/WPA3 Transitional presentan una serie de vulnerabilidades aprovechadas por atacantes malintencionados. En este modo, se permite la conexión tanto de dispositivos que soportan WPA3 como aquellos que solo son compatibles con WPA2, introduciendo una superficie de ataque más amplia. En estas redes suele haber clientes que utilizan WPA2, por lo que a pesar de que la mayoría de los clientes estén configurados para usar WPA3-SAE, es posible obtener un Handshake de WPA2 y realizar diferentes ataques como si fuese una red WPA2. Es importante tener en cuenta que si un cliente estaba configurado con WPA2 y posteriormente se cambió la red a WPA2/WPA3 Transitional seguirá estando conectado con WPA2, ya que el dispositivo detecta la red WPA3 como diferente, ya que cambia el método de autenticación.



### Detección de soporte AP de Transitional Mode

Para comprobar el soporte AP de PSK o SAE, se siguen estos pasos:

1.  **Captura de tráfico inalámbrico**

    Se ejecuta airodump-ng y se guarda la captura.

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

2. **Inspección del RSN IE en Wireshark**

* Se abre el archivo .cap en Wireshark.
* Se localiza un Beacon Frame y, dentro de él, la sección: IEEE 802.11 Wireless Management ▶ Tagged parameters ▶ RSN IE ▶ Auth Key Management (AKM) list.
* Se comprueba si aparece solo SAE, solo PSK o ambos (Transitional).

<figure><img src="../../../.gitbook/assets/image (14) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**Comprobación de MFP**

* En el mismo beacon, se localiza el campo RSN Capabilities.
* Se revisan dos indicadores binarios:
  * MFPC: **0** = no soportado; **1** = soportado.
  * MFPR: **0** = no obligatorio; **1** = obligatorio.

<figure><img src="../../../.gitbook/assets/image (15) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Por defecto, todas las redes SAE requieren MFP.

### Detección de uso de MFP por cliente

Para entender si los ataques de desautenticación serán efectivos o bloqueados, es preciso comprobar la configuración de MFP del cliente.

* Si el cliente no utiliza MFP, se podrán enviar tramas de desautenticación.
* Si MFP está habilitado, se espera a que el cliente se reconecte.

El MFP del cliente solo debe comprobarse cuando la bandera de soportado (capable) del AP esté en 1. Si es obligatorio, siempre estará habilitado; si no es capaz, siempre estará deshabilitado.

El estado de protección de tramas se verifica así:

1.  **Filtrado de Association Requests de un cliente específico**Copiar

    ```
    wlan.fc.type_subtype == 0x0000
    ```
2. **Localización de RSN Capabilities**
   * Se comprueba el bit de MFP Required en RSN Capabilities:
     * **0** = no obligatorio.
     * **1** = obligatorio.
   * Se comprueba el bit de MFP Capable en RSN Capabilities:
     * **0** = no soportado.
     * **1** = soportado.

<figure><img src="../../../.gitbook/assets/image (16) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (17) (1).png" alt=""><figcaption></figcaption></figure>

### Detección de tipo de conexión de clientes en redes de transición (PSK ↔ SAE)

Para verificar si el cliente emplea PSK o SAE en la red de transición, se realizan estos pasos:

1.  **Búsqueda de Association Responses en Wireshark**Copiar

    ```
    wlan.fc.type_subtype == 0x0001
    ```
2. **Localización del RSN IE (Element ID 0x30)**
3. **Omisión de las secciones Version, Group-Cipher y Pairwise-Cipher**
4.  **Lectura de las AKM Suites**

    El contador de 2 octetos AKM-count indica cuántas suites siguen; cada suite es un selector de 4 octetos que se puede ver en la sección de Association ID o en hexadecimal abajo:

    * **PSK:** (AID = `0x01C0`): `00-0F-AC-02`

<figure><img src="../../../.gitbook/assets/image (18) (1).png" alt=""><figcaption></figcaption></figure>

* **SAE:** (AID = `0x02C0`): `00-0F-AC-08`

<figure><img src="../../../.gitbook/assets/image (19) (1).png" alt=""><figcaption></figcaption></figure>

## Ataques de Degradación a WPA2

Este tipo de ataque consiste en engañar al dispositivo para que se conecte utilizando el protocolo WPA2 en lugar de WPA3, siendo vulnerable a ataques de WPA2. A continuación, se puede ver el proceso para realizar el downgrade.

**1.Capturar tráfico de red:** Utiliza una herramienta de captura de paquetes como airodump-ng para monitorear las conexiones en el área.

```
sudo airodump-ng wlan0
```

**2.Verificar si el AP y el cliente utilizan MFP**: Podemos utilizar wifi\_db o Wireshark para verificar si el cliente y la red soportan y están utilizando MFP, ya que si estuviese habilitado no sería posible realizar un ataque de desautenticación

<figure><img src="../../../.gitbook/assets/image (20) (1).png" alt=""><figcaption></figcaption></figure>

Toda la información sobre MFP está en el capitulo 1 802.11w (MFP)

**3.Ataque de desautenticación de la conexión WPA3:** Usamos aireplay-ng para enviar desautenticaciones y forzar la desconexión de las estaciones de la red WPA3.

```
sudo aireplay-ng --deauth 10 -a <BSSID> -c <CLIENT> wlan0
```

**4.Configurar un punto de acceso falso con WPA2:** Configura un punto de acceso malicioso que sólo soporte WPA2 utilizando hostapd con el mismo nombre y una contraseña al azar para obtener el Handshake.

Edita un archivo de configuración hostapd.conf:

```
interface=wlan0
driver=nl80211
ssid=FakeAP
channel=6
hw_mode=g
ieee80211n=1
ieee80211ac=1
wmm_enabled=1
# WPA2 settings
wpa=2
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
auth_algs=1
wpa_passphrase=maliciouspassword
```

Y levanta el punto de acceso:

```
sudo hostapd hostapd.conf
```

**5.Captura la negociación WPA2:** Utiliza airodump-ng nuevamente para capturar la negociación de la clave WPA2.

```
sudo airodump-ng -c 6 --bssid <FakeAP_MAC> -w capture wlan0
```

También se podría hacer como un ataque de NoAP de PSK con hostapd-mana.

## Ataques de Fuerza Bruta Online

Aunque SAE está diseñado para ser resistente a ataques de fuerza bruta gracias a la utilización de Dragonfly handshake, los atacantes pueden intentar ataques de fuerza bruta online. Necesitan realizar múltiples intentos contra la red, pero esto puede ser detectado y mitigado si se cuenta con una buena política de contraseñas, principalmente evitando contraseñas predecibles o inseguras.

### **Ejemplo de Ataque de Fuerza Bruta Online:**

Realiza intentos de fuerza bruta utilizando wacker:

```
./wacker.py --wordlist $DICCIONARIO --ssid $ESSID --bssid $BSSID --interface $INTERFACE --freq $FRECUENCIA_CANAL
```

* \--wordlist $DICTIONARY: Especifica la ruta del archivo de diccionario que se usará para los ataques de fuerza bruta.
* \--ssid $ESSID: Define el SSID de la red objetivo.
* \--bssid $BSSID: Establece el BSSID de la red objetivo.
* \--interface $INTERFACE: Indica la interfaz de red que se usará.
* \--freq $CHANNEL\_FREQUENCY: Especifica la frecuencia del canal de la red objetivo.

La frecuencia la podemos obtener de la tabla de canales de, por ejemplo en la siguiente imagen podemos ver las frecuencias de los canales de 2.4 GHz.

<figure><img src="../../../.gitbook/assets/image (21) (1).png" alt=""><figcaption></figcaption></figure>

También se puede obtener desde la terminal con iwlist:

```
sudo iwlist wlan0 frequency | grep 'Channel 12 :'
```

Este ataque también se puede realizar con airgeddon de forma automática utilizando el plugin wpa3\_online\_attack precompilados para cualquier arquitectura, más info .

### Ataques de Evil Twin

Si se consigue la clave de la red es posible crear una red indistinguible de la original con el mismo ESSID y la misma contraseña y esperar a que los clientes se conecten para realizar ataques en local o para realizar MiTM y ataques de envenenamiento. El proceso del ataque sería el mismo que en PSK pero con la configuración de hostapd de SAE.

### Ataques de Evil Twin OPN

Al igual que en redes WPA2 un atacante podría realizar ataques con un AP con el mismo ESSID y sin contraseña forzando un MiTM o un portal cautivo para robar información. El proceso del ataque sería el mismo que en PSK.
