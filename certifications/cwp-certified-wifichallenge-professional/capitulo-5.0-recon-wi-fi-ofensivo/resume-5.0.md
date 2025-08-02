# Resume - 5.0

## Interfaz modo monitor

```bash
## Listamos todas la interfaces de red
sudo su && iwconfig
# or
sudo su && airmon-ng

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

## Detener modo monitor {wlan0}
ip link set wlan0 down && iwconfig wlan0 mode managed

```

### Airodump-ng

```
airodump-ng wlan0 -w /home/user/wifi/scan --manufacturer --band bag
```

> `-w` guardar ruta del escano
>
> `--manufacturer` para optener informacion de los fabricantes
>
> `--band bag` scan 2.4g y 5g

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> `BSSID` MAC del punto de acceso
>
> `PWR` potencia (cuanto mas cerca de 0 mas cerca esta el punto de acceso)
>
> `CH` canal que utiliza el AP
>
> `ENC CIPHER` tipo de red cifrado
>
> `AUTH` tipo de autentificación

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> Con \<TAB> podemos seleccionar las APs y con \<TAB> + m, podemos marcar colores

Podemos para la captura con `Q` y veremos en el directorio ejecutado varios archivos. El mas importante el el `.cap` ya que contiene toda la información en bruto, podemos abrirlo con `wireshark`

<figure><img src="../../../.gitbook/assets/image (12) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
wireshark scan-01.cap
```

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Con esto podemos ver trafico de redes wifi abiertas en el que el que la comunicación no esta cifrada

### Airdoump-ng Chanel

Tras hacer un escaneo de todos los canales, podemos hacer otro escaneo pero solo de un canal en especifico -->

```
airodump-ng wlan0 -w /home/user/wifi/scanCH6 --manufacturer --band bag -c6
```

> `-c6` Seleccionamos el canal, en este caso el 6

<figure><img src="../../../.gitbook/assets/image (14) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Con esto, nos generan nuevos archivos dentro de la ruta y podemos verlos todos generando una base de datos con `Wifi_db`

### Wifi\_db

```
python3 /root/tools/wifi_db/wifi_db.py -d Wifi1test.db /home/user/wifi/
```

> `-d` Escribimos el nombre de la base de datos a generar
>
> `/` Seleccionamos la ruta donde se encutrar los ficheros

<figure><img src="../../../.gitbook/assets/image (15) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora podemos abrirlo con `sqlitebrowser Wifi1test.db`

<figure><img src="../../../.gitbook/assets/image (16) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> Con esto podemos ver toda la informacion ordenada incluyendo los handshake capturados
