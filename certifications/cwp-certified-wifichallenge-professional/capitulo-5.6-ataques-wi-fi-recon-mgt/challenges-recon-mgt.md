# Challenges RECON MGT

En este capítulo vamos a realizar los siguientes retos:

* Challenge 15 - ¿Cuál es el dominio de los usuarios de la red wifi-regional?
* Challenge 16 - ¿Cuál es la dirección de correo electrónico del certificado del servidor?
* #### Challenge 17 - ¿Cuál es el método EAP soportado por el AP wifi-global?

***

## Challenge 15

* ¿Cuál es el dominio de los usuarios de la red wifi-regional?

Primero como siempre, escaneamos todos los canales con `airodump-ng` y encontramos el "wifi-regional", asi que ahora escaneamos solo ese canal y guardamos su info -->

```
airodump-ng wlan0 --band bag -c 44 --bssid F0:9F:C2:71:22:16 -w /home/user/wifi/MGT/regi
```

<figure><img src="../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

Tras escanear un rato, obtenemos un handshake del usuario conectado, <mark style="background-color:yellow;">este handshake no nos sirve para ningún ataque (por el tipo de red MGT que es)</mark>, pero nos indica que hemos visto a un cliente autentificarse contra el AP,  que es donde se manda la info sin cifrar.

### Wireshark

Ahora podemos abrir la captura con `wireshark` y filtramos por `eap` -->

```
wireshark regi-01.cap
```

<figure><img src="../../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

> Buscamos en la cabecera `Info` un `Response, Identity` para que aparezcan todos juntos

<figure><img src="../../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

### Wifi\_db

```
python3 /root/tools/wifi_db/wifi_db.py -d wifiMGT /home/user/wifi/MGT/
```

Abirmos la bbdd con `sqlitebrowser`

```
sqlitebrowser wifiMGT
```

<figure><img src="../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

> Filtramos por `IdentifyAP`

<figure><img src="../../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

### Tshark

Esta información tambien la podemos sacar con `tshark` directamente filtrando por paquetes seap&#x20;

```
tshark -r /home/user/wifi/MGT/regi-01.cap -Y "eap.identity" -T fields -e eap.identity
```

<figure><img src="../../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

***

## Challenge 16

* ¿Cuál es la dirección de correo electrónico del certificado del servidor?

En las redes MGT utilizan un certificado TLS para enviar las credenciales cifradas, asi que podemos utilizar la captura de la "wifi-regional" anterior y usar `pcapFilter` -->

```
bash /root/tools/pcapFilter.sh -f /home/user/wifi/MGT/regi-01.cap -C | more
```

> `-C` : para obtener la información de los certificados
>
> `| more` : para mostrar por paginas

<figure><img src="../../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

***

## Challenge 17

* ¿Cuál es el método EAP soportado por el AP wifi-global?

Para ver que método EAP soporta AP, necesitamos su `SSID` y un `nombre de usuario valido` (no vale anonymous)

Escanemos la red en busca de "wifi-global" y guardamos su info -->

```
airodump-ng wlan0 --band bag --bssid F0:9F:C2:71:22:17 -c 44 -w /home/user/wifi/MGT/wifi-global
```

<figure><img src="../../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

Ahora montamos la bbdd con `wifi_bd`

```
python3 /root/tools/wifi_db/wifi_db.py -d wifiGLOBAL /home/user/wifi/MGT/wifi-global/
```

<figure><img src="../../../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

Abrimos con `sqlitebrowser` -->

<figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

Encontramos a un `GLOBAL\GlobalAdmin` que podemos utilizar

Ahora utilizamos `EAP_buster` para hacer las pruebas, que automatiza la conexión con cada método par ver cual sorporta el AP

Antes <mark style="background-color:yellow;">es recomendable parar todos los procesos que pueden dar problemas</mark> usando `check kill`

```
airmon-ng check kill
```

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

Y ahora ejecutamos la herramienta, especificando el nombre de la red a la que queremos conectarnos, el nombre del usuario y interfaz a utilizar, esperamos....

```
bash /root/tools/EAP_buster/EAP_buster.sh wifi-global 'GLOBAL\\GlobalAdmin' wlan2
```

