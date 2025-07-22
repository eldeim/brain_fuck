# Capitulo 3: Fundamentos de Linux

## SCP

SCP te permite transferir archivos de forma segura entre sistemas a través de SSH, encriptando tanto los datos como las credenciales.

### Sintaxis de SCP

```
scp [opciones] archivo_origen usuario@host_remoto:ruta_de_destino
```

#### Ejemplos

* **Copiar un archivo a un servidor remoto**

```
scp miarchivo.txt usuario@192.168.1.10:/home/usuario/
```

* **Copiar un archivo desde un servidor remoto**

```
scp usuario@192.168.1.10:/home/usuario/miarchivo.txt /ruta/local/
```

* **Copiar recursivamente un directorio**

```
scp -r /directorio/local usuario@remoto:/ruta/hacia/destino/
```

### Opciones comunes de SCP

* -r: Copia recursivamente directorios completos
* -P: Especifica un puerto personalizado
* -C: Habilita la compresión para transferencias más rápidas

***

## Túneles SSH

### Comando de Configuración

```
ssh -R [puerto_remoto]:localhost:[puerto_local] [usuario]@[servidor_remoto]
#Ejemplo
ssh -R 9090:localhost:8080 usuario@ejemplo.com
```

Este comando asigna el puerto 9090 en el servidor remoto (ejemplo.com) al puerto 8080 en tu máquina local. El servidor remoto puede acceder a tu servidor web local visitandolocalhost:9090.

Después de haber abierto el túnel dinámico, sigue los pasos a continuación para configurar Firefox para utilizar el proxy SOCKS:

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

* Haz clic en el botón de menú (tres líneas horizontales en la esquina superior derecha).
* Selecciona Settings o Preferencias
* En la parte superior buscamos proxy
* Y pulsamos en Settings

Una vez en la configuración del proxy podemos configurarlo manualmente con SOCKS v5, la IP 127.0.0.1 que es nuestro propio equipo y el puerto que hemos utilizado en el túnel SSH, en este caso 1080. Y pulsamos OK para guardar.

***

## X11 Forwarding

X11 forwarding permite ejecutar aplicaciones gráficas en un servidor remoto mientras la interfaz se muestra en la máquina local. SSH asegura el protocolo X11 encapsulándolo en un túnel cifrado.

* Instala un servidor X en tu sistema local (por ejemplo, XQuartz para macOS, Xming para Windows).
* Asegúrate de que la línea X11Forwarding esté habilitada en el archivo de configuración del servidor SSH (/etc/ssh/sshd\_config).
* Usa la opción -X al conectarte:

```
ssh -X usuario@host_remoto
```

#### Ejemplo:

Ejecuta una aplicación gráfica (como firefox) en el servidor remoto:

```
firefox
```

La ventana de la aplicación aparecerá en tu máquina local.

***

## Entendiendo el Cracking Wi-Fi: Herramientas y Técnicas

### Hashcat

Hashcat es una herramienta de recuperación de contraseñas que admite una amplia gama de algoritmos y es conocida por su velocidad y versatilidad. Utiliza principalmente la potencia de las tarjetas gráficas (GPU) para acelerar el proceso de cracking, aunque también soporta el uso del procesador (CPU).

#### Cracking MSCHAPv2

MSCHAPv2 es un protocolo de autenticación basado en contraseñas ampliamente utilizado en entornos WPA/WPA2/WPA3-Enterprise. Hashcat se puede utilizar para crackear hashes MSCHAPv2 aprovechando el modo 5500 (Microsoft Challenge Handshake Authentication Protocol v2). El proceso generalmente requiere los datos capturados de la red, donde Hashcat aplica ataques de fuerza bruta o diccionario para adivinar la contraseña.

```
hashcat -m 5500 hash.txt -a 0 -w 3 rockyou.txt
```

Este comando indica a Hashcat que crackee el hash en hash.txt utilizando la lista de palabras rockyou.txt, bajo el modo 5500 para MSCHAPv2.

```
Ejercicio: Crackea el siguiente hash:

user::::ef900eb6993bc7b399b8eaaf63de642e1e1f269f48ed3e22:f48f789635d6aaa4
```

#### Cracking WPA/WPA2-PSK con Hashcat

Hashcat también proporciona un mecanismo potente para crackear redes WPA/WPA2-PSK, utilizando el modo 22000. Este es un método eficiente para crackear Handshake WPA/WPA2 capturados. Para usar Hashcat para crackear WPA/WPA2-PSK, primero necesitas un archivo de Handshake, que se puede capturar usando herramientas como aircrack-ng o Wireshark.

```
hashcat -m 22000 hash.hccapx -a 0 -w 3 rockyou.txt
```

En este comando:

* -m 22000 especifica el modo para WPA/WPA2-PSK.
* hash.hccapx es el archivo de Handshake en formato hccapx (es posible convertir archivos .cap a .hccapx usando la herramienta cap2hccapx proporcionada por Hashcat).
* -a 0 denota el modo de ataque (directo), utilizando un archivo de diccionario.
* rockyou.txt es el archivo de diccionario utilizado para intentar el cracking.

Este enfoque depende en gran medida de la fuerza del diccionario utilizado. Para mayores tasas de éxito, es aconsejable usar diccionarios completos o utilizar ataques basados en reglas para modificar dinámicamente los diccionarios existentes durante el proceso de cracking.

Aprovechando las capacidades de aceleración de GPU de Hashcat, el cracking de Handshake WPA/WPA2 se vuelve más rápido en comparación con las herramientas basadas en CPU. Esto lo convierte en una opción preferida en escenarios donde la eficiencia de tiempo es crítica.

```
Ejercicio: Crackea el siguiente hash:

WPA*02*11ede392e66d40d84c6ca5f8982f88fa*020000000100*020000000300*77696669*cefadaf500e7f55b6ab632bea9f879bbc30d325af2c6886f6560b0380805e0bd*0103007502010a00000000000000000001bf49e050a9794dbce83a4d369b9904dbc1e30c14d2b31e8418790a99161ed4db000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001630140100000fac020100000fac040100000fac020000*00
```

### John the Ripper

John the Ripper es otra herramienta poderosa para el cracking de contraseñas, conocida por su amplia gama de tipos de hash y cifrado soportados, incluyendo MSCHAPv2.

#### Cracking MSCHAPv2

John también se puede utilizar para crackear MSCHAPv2 utilizando su capacidad para realizar ataques tanto de diccionario como de fuerza bruta. Necesitas proporcionar a John el formato correcto de datos extraídos de capturas de red u otras fuentes. John, al igual que Hashcat, tiene su propio formato, en este caso es: USER:$NETNLM$CHALLENGE$RESPONSE:

```
john --format=netntlm hash.txt
```

Este comando ejecuta John contra los hashes en hash.txt con un formato especificado adecuado para hashes MSCHAPv2.

```
Ejercicio: Crackea el siguiente hash:

user:$NETNTLM$f48f789635d6aaa4$ef900eb6993bc7b399b8eaaf63de642e1e1f269f48ed3e22
```

### aircrack-ng <a href="#aircrack-ng" id="aircrack-ng"></a>

aircrack-ng es principalmente conocido por su capacidad para crackear contraseñas Wi-Fi, especialmente aquellas aseguradas con WEP y WPA/WPA2-PSK.

#### Cracking WPA/WPA2-PSK:

La herramienta es particularmente efectiva en el manejo de redes PSK. Opera capturando paquetes de red, y una vez que se han capturado suficientes paquetes para analizar el Handshake entre clientes y puntos de acceso, aircrack-ng intenta crackear el PSK utilizando la lista de palabras proporcionada.

```
aircrack-ng -w ruta/al/diccionario.txt -e [ESSID] [capfile.cap]
```

Este comando dirige a aircrack-ng a utilizar el diccionario en /ruta/al/diccionario.txt para crackear el PSK de la red identificada por su ESSID, con los datos de captura obtenidos en \[capfile.cap]. En el caso de que no especifiquemos el ESSID nos aparecerá la siguiente imagen para seleccionar el objetivo y nos indica si tiene Handshake o PMKID.

```
aircrack-ng -w ruta/al/diccionario.txt [capfile.cap]
```

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

### Pyrit

Pyrit es una herramienta poderosa para descifrar redes WPA/WPA2-PSK, conocida por aprovechar CPUs multinúcleo y GPUs para acelerar el proceso. Es ideal para usuarios que necesitan manejar grandes listas de palabras y realizar descifrado a alta velocidad.

Características Clave

* Aceleración por GPU: Utiliza CUDA y OpenCL para aprovechar la potencia de las GPUs de NVIDIA y AMD.
* Hashes Precomputados: Permite la precomputación de PMKs (Pairwise Master Keys) para un descifrado más rápido.
* Escalabilidad: Soporta la creación de clusters para distribuir la carga entre múltiples máquinas.
* Gestión de Bases de Datos: Almacena y gestiona hashes precomputados de manera eficiente.
