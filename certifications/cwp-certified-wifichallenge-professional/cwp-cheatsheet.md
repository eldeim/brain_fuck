# 👑 CWP Cheatsheet

## Redes Ocultas Scan

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

1. Escanear redes ocultas para encontrar su nombre. Esto lo hacemos con `mk4` -->

```
## Conguracion del canal 11
iwconfig wlan0 channel 11
## Ataque de fuerza bruta
mdk4 wlan0 p -t F0:9F:C2:6A:88:26 -f /root/rockyou-top100000.txt
```

> Suponiendo que es un wifi, filtro dentro del rocku por algo que empiece por "wifi" y repito el ataque. Descubriendo asi la el SSID de la wifi -->

```
cat /root/rockyou-top100000.txt | awk '{print "wifi-" $1}' > /root/wifi-rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.0-recon-wi-fi-ofensivo/challenges-recon" %}

***

## Redes OPN

1. Es posible que la red OPN (de wifi abierta) tenga el nombre oculto, asi que tras encontrar el nombre, debemos conectarnos
2. Probar contraseñas por defecto en el router/puerta de enlace
3. Evadir portal cautivo cambiando nuestra MAC por la de otro cliente con macchanger (ejemplo en Capitulo 5.1, reto 6)

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

```
## Apagamos la interfaz wlan2 (interfaz en modo normal que no utiliza modo monitor)
ip link set wlan2 down
## Cambiamos la direccion MAC por uno de los clientes
macchanger -m 80:18:44:BF:72:47 wlan2
## Levantamos la interfaz
ip link set wlan2 up
## Paramos NetworkManager por posibles errores
systemctl stop network-manager.service
## Comprobamos MACs
macchanger wlan2
```

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

4. Interceptar trafico en escucha con `airodump-ng` y capturar peticiones como; credenciales de acceso al router

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.1-ataques-wi-fi-opn-redes-publicas-abiertas/challenges-opn" %}

***

## Redes WEP

1. (Modo facil) - Utilizamos la herramienta automatizada `Besside-ng`, especificando el BSSID del AP para que haga el ataque de fuerza bruta al "PIN". Por ultimo nos conectamos

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.3-ataques-wi-fi-wep-wired-equivalent-privacy/challenges-wep#besside-ng" %}

***

## Redes PSK - WPA/WPA2

1. Escanear trafico de ese AP/BSSID para captura el handshake y crackear la contraseña
   1. Si es necesario, hacer ataques de des autentificación
2. Si tenemos la contraseña de la wifi y el trafico capturado/handshake, podemos desencriptar el trafico con `airdecap-ng` para ver posible credenciales enviadas y/o cookies de sesion (Capitulo 5.4, challenge 9)
3. Una vez dentro de la red, podemos comprobar si existe aislamiento entre clientes, lanzado un `arp-scan` y `nmap` para ver sus puertos y tambien si tienen webs corriendo
4. Comprobar usuarios conectados a APs/BSSIDs imbisibles, pero con probes. Si existen, realizar un ataque `NO AP` con `hostapd-mana` para obtener su hash y crakear la password

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

{% embed url="https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.4-ataques-wi-fi-psk-pre-shared-key/challenges-psk#challenge-12" %}

***

## Redes SAE - WPA3
