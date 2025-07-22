# Creación de Redes Wi-Fi (Access Points)

## Red OPN

```
interface=wlan0
driver=nl80211
hw_mode=g
channel=6
ssid=$ESSID
wpa=0
# Filtro de Mac
#macaddr_acl=1
#accept_mac_file=/root/open/acceptMac.txt
ap_isolate=1
```

* **interface**: Especifica la interfaz inalámbrica que se va a utilizar.
* **driver**: Indica el controlador que se usa para gestionar la interfaz inalámbrica.
* **hw\_mode**: Define el modo de hardware (por ejemplo, ‘g’ para 2.4 GHz).
* **channel**: El canal en el que operará la red inalámbrica (ejemplo: 6).
* **ssid**: El nombre de la red inalámbrica.
* **wpa**: Establece el protocolo de seguridad (0 para sin cifrado, abierto).
* **macaddr\_acl**: Especifica el control de acceso por dirección MAC (opcional y comentado).
* **accept\_mac\_file**: Archivo con la lista de direcciones MAC permitidas (opcional y comentado).
* **ap\_isolate**: Aisla a los clientes conectados al punto de acceso, evitando que se comuniquen entre sí.

## Red OWE

```
interface=wlan0
driver=nl80211
ssid=MyOWENetwork
hw_mode=g
channel=6
ieee80211w=1
wpa=2
wpa_key_mgmt=OWE
rsn_pairwise=CCMP
```

* **ieee80211w**: Habilita la gestión protegida de frames (1 habilitado).
* **wpa**: Establece el protocolo de seguridad (2 para WPA2).
* **wpa\_key\_mgmt**: Método de gestión de claves WPA (OWE para Opportunistic Wireless Encryption).
* **rsn\_pairwise**: Cifrado usado para la red (CCMP).

## Red WEP

```
interface=wlan0
driver=nl80211
#ignore_broadcast_ssid=2
hw_mode=g
channel=1
ssid=wifi-old
auth_algs=1
wep_default_key=0
wep_key0=1122334455
```

* **ignore\_broadcast\_ssid**: Controla si el SSID se transmite en las tramas de beacon (opcional y comentado).
* **auth\_algs**: Algoritmo de autenticación (1 para sistema abierto).
* **wep\_default\_key**: Clave WEP predeterminada.
* **wep\_key0**: Valor de la clave WEP (primer índice, normalmente 0).

## Red PSK

```
interface=wlan0
driver=nl80211
hw_mode=g
channel=6
ssid=wifi-mobile
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP CCMP
wpa_passphrase=passwordPSK
```

* **wpa\_key\_mgmt**: Método de gestión de claves WPA (WPA-PSK para Pre-Shared Key).
* **wpa\_pairwise**: Indica los métodos de cifrado (TKIP y CCMP).
* **wpa\_passphrase**: Frase de contraseña precompartida.

## Red SAE

```
interface=wlan0
ctrl_interface=/var/run/hostapd
ssid=wifi-management
hw_mode=g
channel=11
wmm_enabled=1
wpa=2
wpa_passphrase=passwordSAE
wpa_key_mgmt=SAE
ieee80211w=2
```

* **ctrl\_interface**: Ruta del directorio de control.
* **wmm\_enabled**: Habilita Wi-Fi Multimedia (WMM).
* **wpa\_key\_mgmt**: Método de gestión de claves WPA (SAE para Simultaneous Authentication of Equals).
* **ieee80211w**: Habilita la gestión protegida de frames (2 para obligatorio).

## Red MGT

En las redes MGT no solo es necesario el fichero de configuración de hostapd, también hace falta un certificado TLS y un fichero con las credenciales de los usuarios o una conexión a un servidor Radius (todo esto lo veremos más en profundidad durante el Capítulo 5.7)

```
interface=wlan0
ssid=wifi-corp
channel=44
hw_mode=a
country_code=ES
ieee8021x=1
ieee80211d=1
wpa=2
wpa_key_mgmt=WPA-EAP
wpa_pairwise=CCMP
rsn_pairwise=CCMP
eap_server=1
eapol_key_index_workaround=0
own_ip_addr=127.0.0.1
eap_user_file=/root/mgt/hostapd_wpe.eap_user
ca_cert=/root/mgt/certs/ca.crt
server_cert=/root/mgt/certs/server.crt
private_key=/root/mgt/certs/server.key
private_key_passwd=whatever
```

* **country\_code**: Código del país para establecer las regulaciones.
* **ieee8021x**: Habilita IEEE 802.1X para autenticación de red.
* **ieee80211d**: Habilita IEEE 802.11d (anuncio de regulaciones locales).
* **wpa\_key\_mgmt**: Método de gestión de claves WPA (WPA-EAP para EAP based authentication).
* **eap\_server**: Habilita el servidor EAP interno.
* **eapol\_key\_index\_workaround**: Parámetro relacionado con la gestión de claves EAPOL.
* **own\_ip\_addr**: Dirección IP del punto de acceso.
* **eap\_user\_file**: Archivo de configuración de usuarios EAP.
* **ca\_cert**: Ruta al certificado de la autoridad de certificación.
* **server\_cert**: Ruta al certificado del servidor.
* **private\_key**: Ruta a la clave privada del servidor.
* **private\_key\_passwd**: Contraseña de la clave privada.

Una vez creado el fichero de hostapd es necesario crear un archivo de configuración eap\_user para hostapd. Este fichero está especificado en la ruta /root/mgt/hostapd\_wpe.eap\_user dentro del fichero hostapd.

> Nota: En un caso más avanzado y realista se utilizaría un servidor Radius para la autenticación

```
# Phase 1 users
"example user"	TLS
"DOMAIN\user"	MSCHAPV2	"password"
# Phase 2 (tunnelled within EAP-PEAP or EAP-TTLS) users
"DOMAIN\t-mschapv2"	MSCHAPV2	      "password"	[2]
"t-gtc"		     GTC	            "password"	[2]
"user"		     MD5,GTC,MSCHAPV2	 "password"	[2]
"test user"	     MSCHAPV2	      hash:000102030405060708090a0b0c0d0e0f	[2]
```

Explicación del fichero:

* **Usuarios de Fase 1:**
  * example user usa EAP-TLS, que se basa en certificados en lugar de una contraseña.
* DOMAIN\user usa EAP-MSCHAPv2 con la contraseña "password."
* **Usuarios de Fase 2 (túnel dentro de EAP-PEAP o EAP-TTLS):**
  * DOMAIN\t-mschapv2 usa EAP-MSCHAPv2 con la contraseña "password," específicamente para la Fase 2.
  * t-gtc usa EAP-GTC con la contraseña "password" en la Fase 2.
  * user puede autenticarse usando EAP-MD5, EAP-GTC o EAP-MSCHAPv2 con la contraseña "password" en la Fase 2.
  * test user usa EAP-MSCHAPv2 con una contraseña encriptada en la Fase 2.

Las entradas de la Fase 1 manejan la autenticación inicial, mientras que las entradas de la Fase 2 se utilizan dentro de un túnel encriptado para la autenticación segura. Por ejemplo, DOMAIN\t-mschapv2 se utilizaría dentro de un túnel EAP-PEAP o EAP-TTLS (Fase 2), mientras que DOMAIN\user se utilizaría sin un túnel cifrado (Fase 1).

Podemos utilizar openssl para generar los archivos de configuración necesarios para TLS. Para automatizar este proceso, se puede emplear o un script en bash como el siguiente para generar todo.

```
#!/bin/bash

# Variables
ESSID="YourSSID"
CA_DIR=~/hostapd_certs
OPENSSL_CNF=$CA_DIR/openssl.cnf
HOSTAPD_CONF=/etc/hostapd/hostapd.conf

# Crear directorio para certificados
mkdir -p $CA_DIR
cd $CA_DIR

# Crear archivo de configuración de OpenSSL
cat > $OPENSSL_CNF <<EOL
[ ca ]
default_ca = CA_default

[ CA_default ]
dir = $CA_DIR
database = \$dir/index.txt
new_certs_dir = \$dir/newcerts
certificate = \$dir/cacert.pem
serial = \$dir/serial
private_key = \$dir/private/cakey.pem
default_days = 365
default_md = sha256
preserve = no
policy = policy_anything

[ policy_anything ]
countryName = optional
stateOrProvinceName = optional
localityName = optional
organizationName = optional
organizationalUnitName = optional
commonName = supplied
emailAddress = optional

[ req ]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = req_distinguished_name

[ req_distinguished_name ]
C = US
ST = California
L = San Francisco
O = MyCompany
OU = MyDepartment
CN = MyAP

[ v3_ca ]
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer
basicConstraints = CA:true

[ v3_req ]
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
EOL

# Crear directorios y archivos necesarios
mkdir -p private newcerts
chmod 700 private
touch index.txt
echo 1000 > serial

# Generar certificado de la CA
openssl req -new -x509 -extensions v3_ca -keyout private/cakey.pem -out cacert.pem -config $OPENSSL_CNF -days 3650

# Generar clave y certificado del servidor
openssl genrsa -out server.key 2048
openssl req -new -key server.key -out server.csr -config $OPENSSL_CNF
openssl ca -batch -keyfile private/cakey.pem -cert cacert.pem -in server.csr -out server.pem -config $OPENSSL_CNF -extensions v3_req

# Opcional: Generar clave y certificado del cliente
openssl genrsa -out client.key 2048
openssl req -new -key client.key -out client.csr -config $OPENSSL_CNF
openssl ca -batch -keyfile private/cakey.pem -cert cacert.pem -in client.csr -out client.pem -config $OPENSSL_CNF -extensions v3_req

# Configurar hostapd
sudo bash -c "cat > $HOSTAPD_CONF <<EOL
interface=wlan0
driver=nl80211
ssid=$ESSID
hw_mode=g
channel=6
wpa=2
wpa_key_mgmt=WPA-EAP
rsn_pairwise=CCMP

ieee8021x=1
eapol_version=2
eap_server=0
ca_cert=$CA_DIR/cacert.pem
server_cert=$CA_DIR/server.pem
private_key=$CA_DIR/server.key
EOL"

# Reiniciar hostapd
sudo systemctl restart hostapd

echo "Certificados generados y hostapd configurado correctamente."
```
