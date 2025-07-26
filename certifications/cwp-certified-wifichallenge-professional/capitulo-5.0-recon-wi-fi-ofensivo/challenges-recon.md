# Challenges RECON

En este capítulo vamos a realizar los siguientes retos:

* Challenge 1 - ¿Cuál es el canal que está utilizando actualmente el punto de acceso (AP) wifi-global?
* Challenge 2 - ¿Cuál es la MAC del cliente de la red wifi-IT?
* Challenge 3 - ¿Cuál es el Probe de 78:C1:A7:BF:72:46 que sigue el formato de las otras redes del alcance (wifi-)?
* Challenge 4 - ¿Cuál es el ESSID del AP oculto (mac F0:9F:C2:6A:88:26)?

***

## Challenge 1

* ¿Cuál es el canal que está utilizando actualmente el punto de acceso (AP) wifi-global?

<figure><img src="../../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>



## Challenge 2&#x20;

* ¿Cuál es la MAC del cliente de la red wifi-IT?

Primero veo que canal tiene; 11

<figure><img src="../../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

Luego le lanzo un escaneo solo a ese canal para filtrar mejor

```
airodump-ng wlan0 --manufacturer --band bag -c11
```

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>



## Challenge 3&#x20;

* ¿Cuál es el Probe de 78:C1:A7:BF:72:46 que sigue el formato de las otras redes del alcance (wifi-)?

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>



## Challenge 4&#x20;

* ¿Cuál es el ESSID del AP oculto (mac F0:9F:C2:6A:88:26)?

Primero vemos el BSSID de esa MAC, descubriendo el canal que usa

<figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

Ahora tras saber que esta en el canal  cono la BSSID F0:9F:C2:6A:88:26, configuramos el iwconfig y lanzamos el ataque de fuerza bruta -->

```
## Conguracion del canal 11
iwconfig wlan0 channel 11
## Ataque de fuerza bruta
mdk4 wlan0 p -t F0:9F:C2:6A:88:26 -f /root/rockyou-top100000.txt 
```

<figure><img src="../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

Con esto vemos que hace un ataque de fuerza bruta en los nombres y prueba todos los del rocku. Suponiendo que es un wifi, filtro dentro del rocku por algo que empiece por "wifi" y repito el ataque. Descubriendo asi la el SSID de la wifi -->

```
cat /root/rockyou-top100000.txt | awk '{print "wifi-" $1}' > /root/wifi-rockyou.txt
```

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>
