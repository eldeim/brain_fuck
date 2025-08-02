# Capitulo 4: Redes Wi-Fi en Linux

### Información de las Interfaces

Para obtener información sobre las interfaces Wi-Fi en un sistema Linux, es posible usar el comando iwconfig o ip. Sin embargo, el comando iw proporciona información más detallada y es el reemplazo moderno de iwconfig.

Para cambiar el modo de forma de nombrar las interfaces de red en Linux, puedes modificar la configuración del sistema. Si deseas utilizar el esquema clásico (wlan0, eth0, etc.), se puede deshabilitar el esquema moderno de nombres predecibles. Para hacking de redes Wi-Fi se recomienda utilizar siempre el formato clásico, ya que hay herramientas como wash que dan problemas con el formato nuevo.

Esto se puede arreglar de 2 formas diferentes, una temporal y otra permanente. La temporal sería únicamente modificar el nombre de la interfaz de la siguiente forma.

```
ip link set wlx00c0ca9208dc down && ip link set wlx00c0ca9208dc name wlan0 && ip link set wlan0 up
```

Aunque con este método tras reiniciar el nombre volvería al original.

Para dejar la modificación permanente se logra agregando un parámetro al arranque del Kernel. Edita el archivo de configuración del cargador de arranque (como GRUB) y añadirnet.ifnames=0 y biosdevname=0 a la línea de comandos del Kernel. Por ejemplo, en sistemas basados en GRUB, es posible conseguir esto editando el fichero /etc/default/grub y modificando la línea GRUB\_CMDLINE\_LINUX añadiendo estos parámetros:

```
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
```

Después, se actualiza la configuración de GRUB ejecutando sudo update-grub y se reinicia el sistema. Al reiniciar, las interfaces de red deberían volver a usar los nombres clásicos.

* **Uso del comando** iw
* iw dev: Lista todas las interfaces inalámbricas en el sistema.
* iw dev wlan0 info: Proporciona información detallada sobre una interfaz específica, en este caso wlan0.
* **Uso del comando**ip
* ip link show: Muestra todas las interfaces de red, incluidas las Wi-Fi.

***

## Gestión de Interfaces Wi-Fi

### Activar o Desactivar una Interfaz

* Para desactivar una interfaz Wi-Fi: sudo ip link set dev wlan0 down.
* Para activar una interfaz Wi-Fi: sudo ip link set dev wlan0 up.

Reemplaza wlan0 con el nombre de la interfaz Wi-Fi específica.

### Escaneo de Redes Wi-Fi

El escaneo de redes Wi-Fi se puede hacer fácilmente mediante el comando iw.

* Asegurarse de que la interfaz Wi-Fi está activa: sudo ip link set dev wlan0 up.
* Escanear las redes Wi-Fi: sudo iw dev wlan0 scan.

Este comando listará todas las redes Wi-Fi disponibles con información detallada como SSID, fuerza de señal, tipo de cifrado, etc.

Para un enfoque más sencillo, se puede usar la herramienta nmcli que controla el NetworkManager:

* Listar las redes Wi-Fi disponibles: nmcli dev wifi list.
* Conectarse a una red Wi-Fi: nmcli dev wifi connect SSID\_NAME password YOUR\_PASSWORD.

Reemplaza SSID\_NAME con el nombre de la red y YOUR\_PASSWORD con la contraseña.

Aquí hay comandos adicionales de ejemplo para facilitar diferentes operaciones:

* Muestra las configuraciones y el estado del dispositivo inalámbrico: iw dev o ip link show wlan0.
* Activa una interfaz Wi-Fi: ip link set wlan0 up.
* Verifica el estado del enlace: iw wlan0 link.
* Inicia un escaneo directo de puntos de acceso: iw wlan0 scan.
* Para una salida más limpia que sólo muestre SSIDs: iw wlan0 scan | grep "SSID:".
* Para ver resultados detallados del escaneo que incluyen BSSID, SSID, y protocolos de seguridad: iw dev wlan0 scan | grep "^BSS\\|SSID|WSP\\|Authentication\\|WPA".

### Configurar manualmente la interfaz en un canal específico

Esta operación se realiza mediante el comando iwconfig. En este caso, vamos a configurar la interfaz wlan0mon para que opere en el canal 11. A continuación, se detalla el comando:

```
iwconfig wlan0mon channel 11
```

* wlan0mon: Interfaz a la que se quiere cambiar el canal, puede estar en modo monitor o no.
* 11 Canal en el que se quiere configurar la interfaz.

### Modo Monitor

El modo monitor permite a una interfaz Wi-Fi capturar todo el tráfico captado por la red inalámbrica, independientemente de si está destinado a dicha interfaz o no. Este modo es esencial para realizar análisis detallados de redes y pruebas de penetración, ya que permite observar todo el tráfico que se produce en un canal inalámbrico específico.

#### Modo Manual

**1.Desactivar Network Manager**: Para evitar que el gestor de redes interfiera con la configuración de la interfaz Wi-Fi al habilitar el modo monitor, primero se debe detener el servicio de Network Manager. Esto se puede hacer con el siguiente comando:

```
sudo systemctl stop NetworkManager
```

En algunas distribuciones de Linux este comando puede cambar:

* **Wifislax**

```
service restart networkmanager
```

* **Suse, Fedora, Gentoo y CentOS**

```
service NetworkManager restart
```

* **Pentoo**

```
rc-service NetworkManager restart
```

**2.Habilitar el Modo Monitor**: Antes de habilitar el modo monitor, es necesario identificar el nombre de la interfaz inalámbrica. Esto se puede lograr usando comandos como iwconfig y ip. Una vez identificado, la secuencia de comandos para habilitar el modo monitor sería:

```
sudo ip link set wlan0 down # Desactiva la interfaz
sudo iwconfig wlan0 mode monitor # Cambia el modo de la interfaz a monitor
sudo ip link set wlan0 up # Vuelve a activar la interfaz
```

En este ejemplo, wlan0 es el nombre de la interfaz. Este nombre puede variar según el sistema.

**3.Verificar Configuración**: Para confirmar que la interfaz está en modo monitor, se utiliza el comando iwconfig. La salida del comando debe mostrar Mode:Monitor:

```
iwconfig
```

Para desactivar el modo monitor y volver al modo Managed hacemos el mismo paso modificando el comando de iwconfig

```
sudo ip link set wlan0 down # Desactiva la interfaz
sudo iwconfig wlan0 mode managed # Cambia el modo de la interfaz a monitor
sudo ip link set wlan0 up # Vuelve a activar la interfaz
```

### Monitoreo de Wi-Fi con tcpdump

tcpdump es una herramienta de línea de comandos poderosa para capturar y analizar paquetes TCP/IP en una red. Para usar tcpdump de manera efectiva en análisis de Wi-Fi, es crucial que la interfaz esté en modo monitor. Esto permite capturar toda la información transmitida en el aire, sin importar el destino del tráfico.

Ejemplo de comando para capturar paquetes:

```
sudo tcpdump -i wlan0mon -w /ruta/al/archivo.cap
```

En este ejemplo, wlan0mon es la interfaz en modo monitor y /ruta/al/archivo.cap es donde se guardarán los paquetes capturados. Este archivo puede ser analizado posteriormente para obtener información como la intensidad de la señal y los datos transmitidos.

### Cambiar la Dirección MAC

Modificar la dirección MAC de una interfaz puede ser útil para proporcionar anonimato o para evadir filtros de seguridad basados en MAC.

**1.Desactivar la Interfaz**:

```
sudo ifconfig wlan0 down
```

**2. Cambiar la Dirección MAC**: Usando ip, macchanger o ifconfig, se puede cambiar la dirección MAC. Algunos ejemplos son:

```
ip link set wlan0 down
ip link set dev wlan0 address 00:11:22:33:44:55
ip link set wlan0 up
```

Una MAC aleatoria con macchanger:

```
sudo macchanger -r wlan0 # Cambia la MAC a una aleatoria
```

Una especifica con:

```
sudo macchanger -m 00:11:22:33:44:55 wlan0
```

o

```
sudo ifconfig wlan0 hw ether 00:11:22:33:44:55 # Establece una MAC específica
```

Nota: ifconfig está absoleto y se debería actualizar por otras herramientas como ip

**3. Reactivar la Interfaz**:

```
sudo ifconfig wlan0 up
```

### Detener los Servicios de Gestión de Red

La manera más fácil de detener los Servicios de Gestión de Red para que no interfieran con la interfaz mientras está en uso es detenerlo directamente.

#### Detener NetworkManager

En la mayoría de los sistemas Linux modernos, NetworkManager es el servicio predeterminado. Desactívalo utilizando los comandos apropiados para tu sistema de inicio:

**Sistemas basados en Systemd (Ubuntu, Fedora, Debian, etc.)**

```
sudo systemctl stop NetworkManager
```

**Sistemas SysVinit (CentOS 6, versiones antiguas de Debian/Ubuntu)**

```
sudo service NetworkManager stop
```

**Sistemas OpenRC (Gentoo, Alpine Linux)**

```
sudo rc-service NetworkManager stop
```

**Ejemplo:**

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### Detener el Servicio de Red

Algunos sistemas, especialmente los más antiguos, utilizan el servicio de red en lugar de NetworkManager.

**Systemd:**

```
sudo systemctl stop networking
```

**SysVinit:**

```
sudo service networking stop
```

#### Detener Wicd (si está instalado)

Wicd es otra herramienta de gestión de red que se utiliza ocasionalmente en distribuciones ligeras.

**Systemd:**

```
sudo systemctl stop wicd
```

**SysVinit:**

```
sudo service wicd stop
```

### Excluir Temporalmente una Interfaz de la Gestión

Independientemente del servicio que gestione la red, es posible excluir temporalmente interfaces específicas para evitar perder acceso a internet mientras utilizamos otra interfaz.

#### Excluir Interfaces en NetworkManager

* **Usa**nmcli **para dejar de gestionar la interfaz:**

```
sudo nmcli dev set  managed no
```

**Ejemplo:**

```
sudo nmcli dev set wlan0 managed no
```

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* **Para reactivar la gestión:**

```
sudo nmcli dev set  managed yes
```

#### Excluir Interfaces en el Servicio de Red

Modifica /etc/network/interfaces para excluir temporalmente la interfaz:

* **Abre el archivo de configuración:**

```
sudo nano /etc/network/interfaces
```

* **Comenta el bloque de la interfaz relevante añadiendo # al inicio de cada línea:**

```
#auto wlan0
#iface wlan0 inet dhcp
```

* **Reinicia el servicio para aplicar los cambios:**

```
sudo systemctl restart networking
```

#### Excluir Interfaces en Wicd

Wicd permite excluir interfaces a través de su archivo de configuración:

* **Edita /etc/wicd/manager-settings.conf:**

```
sudo nano /etc/wicd/manager-settings.conf
```

* **Añade la interfaz a la lista de exclusión:**

```
ignore_interfaces = wlan0
```

* **Reinicia Wicd:**

```
sudo systemctl restart wicd
```

#### Verificación de la Exclusión

Para asegurarte de que la interfaz está excluida o no gestionada:

* **Verifica el estado de la interfaz con** nmcl&#x69;**:**

```
nmcli dev status
```

Las interfaces excluidas deberían aparecer como "no gestionadas".

* **Usa el comando**ip **para confirmar que no se ha asignado una IP por el gestor:**

```
ip addr show 
```

* **Revisa los registros del gestor de red en uso:**
* **Para** NetworkManage&#x72;**:**

```
sudo journalctl -u NetworkManager
```

* **Para el servicio de red:**

```
sudo journalctl -u networking
```

* **Usa** dmesg **para revisar los registros a nivel de**kernel **de la interfaz:**

```
dmesg | grep 
```

***

## Conexión a Redes Wi-Fi

Para la gestión de redes Wi-Fi en Linux, dos herramientas populares son nmcli y wpa\_supplicant. nmcli es un cliente de línea de comandos para NetworkManager y es adecuado para tareas rápidas. Por otro lado, wpa\_supplicant proporciona un mayor control, especialmente útil para configuraciones complejas. Se van a ver ejemplos de uso de estos comandos en los retos del laboratorio WiFiChallenge Lab.

> Nota: No es necesario saberlo de memoria, es principalmente para poder copiar y pegar cuando llegue el momento.

> Nota: NOTA2: Para ver ejemplos más complejos y un poco toda la información definida se puede ver el fichero de ejemplo oficial: [https://w1.fi/cgit/hostap/plain/wpa\_supplicant/wpa\_supplicant.conf](https://w1.fi/cgit/hostap/plain/wpa_supplicant/wpa_supplicant.conf)

### Redes Abiertas (OPN)

A continuación, vamos a ver como conectarse a redes abiertas desde la terminal.

#### Usando nmcli

```
nmcli device wifi connect SSID_NAME
```

* Reemplaza SSID\_NAME con el nombre de la red.

#### Usando wpa\_supplicant

Crear un archivo de configuración llamado wifi-opn.conf:

```
network={
  ssid="SSID_NAME"
  key_mgmt=NONE
  scan_ssid=1
}
```

Explicación:

* ssid: Nombre de la red inalámbrica a la que deseas conectarte.
* key\_mgmt: Tipo de gestión de claves, en este caso NONE indica que no hay cifrado.
* scan\_ssid: Para encontrar redes con SSID oculto, scan\_ssid=1 puede usarse en el bloque de red con nl80211. Esto fuerza al sistema a buscar y conectarse a redes que no son visibles en el escaneo normal de SSID.

Conectar usando wpa\_supplicant:

```
sudo wpa_supplicant -i wlan0 -c wifi-opn.conf
```

* Reemplaza wlan0 con el nombre de la interfaz de red a utilizar.

### Redes OWE (cifrado Oportunista)

#### Usando wpa\_supplicant

Crear un archivo de configuración llamado wifi-owe.conf:

```
network={
  ssid="SSID_NAME"
  key_mgmt=OWE
}
```

Explicación:

* ssid: Nombre de la red a la que deseas conectarte.
* key\_mgmt: Gestión de claves, OWE para indicar cifrado Oportunista.

Conectar usando wpa\_supplicant:

<pre><code><strong>sudo wpa_supplicant -i wlan0 -c wifi-owe.conf
</strong></code></pre>

### Redes WEP (Privacidad Equivalente por Cable)

#### Usando nmcli

Para conectar a una red WEP:

```
nmcli device wifi connect SSID_NAME password PASSWORD
```

Reemplaza PASSWORD con la clave WEP.

#### Usando wpa\_supplicant

Crear un archivo de configuración llamado wifi-wep.conf:

```
network={
  ssid="SSID_NAME"
  key_mgmt=NONE
  wep_key0=PASSWORD
  wep_tx_keyidx=0
}
```

Explicación:

* wep\_key0: Clave WEP para la red. Siempre que la contraseña sea en hexadecimal, se debe poner sin comillas y sin dos puntos :, Por ejemplo, una contraseña 00:11:22:33:44 se pondría wep\_key0=0011223344
* wep\_tx\_keyidx: Índice de la clave WEP (normalmente 0).

Conectar usando wpa\_supplicant:

```
sudo wpa_supplicant -i wlan0 -c wifi-wep.conf
```

### Redes WPA/WPA2-PSK (Clave Precompartida)

#### Usando nmcli

Para conectar:

```
nmcli device wifi connect SSID_NAME password PASSWORD
```

#### Usando wpa\_supplicant

Crear un archivo de configuración llamado wifi-psk.conf:

```
network={
  ssid="SSID_NAME"
  psk="PASSWORD"
}
```

Explicación:

* psk: Clave pre compartida para WPA/WPA2.

Conectar usando wpa\_supplicant:

```
sudo wpa_supplicant -i wlan0 -c wifi-psk.conf
```

### Redes WPA3-SAE

#### Usando nmcli

Para conectar:

```
nmcli device wifi connect SSID_NAME password PASSWORD
```

#### Usando wpa\_supplicant

Crear un archivo de configuración llamado wifi-sae.conf:

```
network={
  ssid="SSID_NAME"
  psk="PASSWORD"
  key_mgmt=SAE
}
```

Explicación:

* key\_mgmt: Gestión de claves, SAE para WPA3.

Conectar usando wpa\_supplicant:

```
sudo wpa_supplicant -i wlan0 -c wifi-sae.conf
```

### Redes WPA/WPA2/WPA3-Enterprise (MGT)

Para WPA/WPA2/WPA3-Enterprise, se necesita una configuración específica debido a los diversos métodos de autenticación.

#### Usando nmcli

Para conectar:

```
nmcli device wifi connect SSID_NAME password PASSWORD identity USERNAME eap PEAP phase2-auth MSCHAPV2
```

Reemplaza USERNAME y PASSWORD con las credenciales necesarias.

#### Usando wpa\_supplicant

Crear un archivo de configuración llamado wifi-mgt.conf:

```
network={
  ssid="SSID_NAME"
  key_mgmt=WPA-EAP
  eap=PEAP
  anonymous_identity="anonymous"
  identity="USERNAME"
  password="PASSWORD"
  phase2="auth=MSCHAPV2"
}
```

Explicación:

* key\_mgmt: Gestión de claves WPA-EAP para Enterprise.
* eap: Métodos de autenticación EAP.
* anonymous\_identity: Identidad anónima.
* identity: Nombre de usuario para la autenticación.
* password: Contraseña para la autenticación.
* phase2: Tipo de autenticación de segunda fase, en este caso MS-CHAP v2.

Conectar usando wpa\_supplicant:

```
sudo wpa_supplicant -i wlan0 -c wifi-mgt.conf
```

#### Notas

* Reemplazar wlan0 con la interfaz necesaria en cada caso.
* wpa\_supplicant requiere privilegios de root para la mayoría de las operaciones.
* Para wpa\_supplicant, usar -B lo ejecuta en segundo plano. Omite -B para operación en primer plano y salida más detallada.
* Asegúrate de que tu wpa\_supplicant y NetworkManager estén actualizados para soportar protocolos más nuevos como WPA3-SAE y OWE.
* Si el ESSID está oculto, debes añadir hidden yes en el comando nmcli.
* En wpa\_supplicant se puede ejecutar en segundo plano con -B. En este modo se pueden ver los logs con sudo journalctl -u wpa\_supplicant o con sudo grep wpa\_supplicant /var/log/syslog

### Gestionando Redes Inalámbricas con WPA GUI en Ubuntu

WPA GUI ofrece una interfaz gráfica fácil de usar para gestionar conexiones Wi-Fi en Ubuntu, eliminando la necesidad de comandos en la terminal.

#### Instalación de WPA GUI

* **Instalar WPA GUI**

```
sudo apt install wpa-gui
```

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* **Iniciar WPA GUI**
* **Comando:** wpa\_gui o buscar "WPA GUI" en el menú de aplicaciones.

#### Descripción General de la Interfaz de WPA GUI

* **Estado Actual**
  * **Descripción:** Muestra el SSID conectado, la intensidad de la señal y la dirección IP.
* **Escanear**
  * **Descripción:** Lista las redes disponibles con la intensidad de la señal y los tipos de cifrado.
* **Configuración de Red**
  * **Descripción:** Añadir, editar o eliminar perfiles de red.
* **Mensajes**
  * **Descripción:** Muestra los registros de wpa\_supplicant para resolución de problemas.

#### Conectarse a una Red Inalámbrica

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* **Escanear Redes**
  * Navega a la pestaña **Escanear** y haz clic en **Escanear**. Selecciona la red deseada.
* **Configurar la Red**
  * Haz doble clic en una red o ve a **Configuración de Red**, haz clic en **Añadir**, ingresa el SSID, elige la seguridad (por ejemplo, WPA2) e introduce la contraseña. Guarda la configuración.
* **Conectar**
  * Regresa a **Estado Actual** y haz clic en **Conectar** para unirte a la red.
* **Obtener una IP**
  * Una vez conectado, ejecuta dhclient como de costumbre.
