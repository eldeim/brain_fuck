# Challenges OPN

* Challenge 5 - ¿Cuál es la FLAG en el router del AP oculto tras unas credenciales por defecto?
* Challenge 6 - ¿Cuál es la FLAG en el router AP de la red wifi-guest?

***

## Challenge 5

* ¿Cuál es la FLAG en el router del AP oculto tras unas credenciales por defecto?

Al saber el SSID de la red gracias al ejercicio anterior "wifi-free", al ser una ser OPN nos podemos conectar directamente creando/generando el fichero de `wpa_supplicant` -->

<pre class="language-bash"><code class="lang-bash"><strong>## Creamos el fichero
</strong><strong>nano wifi-free.conf
</strong><strong>## Configuramos el archivo
</strong><strong>network={
</strong><strong>        ssid="wifi-free"
</strong><strong>        key_mgmt=NONE
</strong><strong>        scan_ssid=1
</strong><strong>}
</strong></code></pre>

> `ssid=` Nombre de la wifi
>
> `key_mgmt=NONE` Tipo de red a la que nos conectamos, en esta caso NONE ya que es una red OPN
>
> `scan_ssid=1` Al ser un red oculta, es importante ponerlo en 1 para que "wpa\_supplicant" mande probes obligatoriamente

Ahora nos conectamos a la red con `wpa_supplicant`, desde la interfaz y el fichero de cofiguracion -->

```
wpa_supplicant -i wlan2 -c wifi-free.conf
```

<figure><img src="../../../.gitbook/assets/image (19) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora, tras tener conexión hacemos una consulta DHCP con `dhclient` para saber nuestra ip y la ip del router -->

```
sudo dhclient -v wlan2
```

<figure><img src="../../../.gitbook/assets/image (20) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora podemos irnos al navegardor, pegar la ip de la puerta de enlace "router" y probamos contraseñas por defecto -->

<figure><img src="../../../.gitbook/assets/image (21) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

`admin:admin`

<figure><img src="../../../.gitbook/assets/image (22) (1) (1) (1) (1) (1).png" alt="" width="486"><figcaption></figcaption></figure>



## Challenge 6

* ¿Cuál es la FLAG en el router AP de la red wifi-guest?

Tras analizar el trafico con `airodump`, veo la wifi-guest -->

<figure><img src="../../../.gitbook/assets/image (23) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora configuramos el arhcivo para conectarnos de la misma mandera que antes -->

```bash
## Creamos el fichero
nano wifi-guest.conf
## Configuramos el archivo
network={
        ssid="wifi-guest"
        key_mgmt=NONE
}
```

Ahora utilizamos `wpa_supplicant` para conectarnos con una interfaz normal en modo manage

```
wpa_supplicant -i wlan2 -c wifi-guest.conf
```

<figure><img src="../../../.gitbook/assets/image (24) (1) (1).png" alt=""><figcaption></figcaption></figure>

Y en una nueva terminal pedimos ip con dhclient -->

```
sudo dhclient -v wlan2
```

<figure><img src="../../../.gitbook/assets/image (25) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora nos vamos al panel web de esta puerta de enlace "192.168.10.1" y vemos que es un portal cautivo

<figure><img src="../../../.gitbook/assets/image (26) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (27) (1) (1).png" alt=""><figcaption></figcaption></figure>

Para acceder a esta red necesitamos pasar este portal cautivo de alguna forma, asi que debemos escanear el canal correcto para analizar el trafico y las personas conectadas -->

<figure><img src="../../../.gitbook/assets/image (28) (1) (1).png" alt=""><figcaption></figcaption></figure>

Vemos que la "wifi-guest" se encuentra en el canal 6, asi que hacemos un escaneo solo sobre ese canal para guardar todo el trafico -->

```
airodump-ng wlan0 -w /home/user/wifi/OPN/wifi-guest --manufacturer --band bag -c6
```

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### MACChanger

Ahora podemos cambiarnos la dirección mac de nuestra tarjeta de red utilizada, a una direcion MAC de un usario previamente registrado dentro de ese portal causitvo y que sabemos que tiene acceso a la red por la cantidad de data que se envia -->

```bash
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

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>MAC cambiada</p></figcaption></figure>

Ahora tras configurar esto, volvemos a conectarnos con `wpa_supplicant` -->

<pre><code><strong>## Nos conectamos a la interfaz
</strong><strong>wpa_supplicant -i wlan2 -c wifi-guest.conf
</strong>## Pedimos IP al DHCP
sudo dhclient -v wlan2
</code></pre>

Y volvemos a la pagina web en la `192.168.10.1` -->

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="563"><figcaption><p>Pedimos IP al DHCP</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora podemos ver este panel web diferente tras tener acceso a la red. Al ser una wifi abierta podemos filtrar el trafico previemanete capturado con airodump-ng y ver data, viendo asi credenciales -->

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

`free2:5LqwwccmTg6C39y`

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
