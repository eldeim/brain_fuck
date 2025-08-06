# Extracción de Credenciales Wi-Fi

### 1. Windows

1. Abre el Símbolo del sistema como Administrador.
2. Lista las redes Wi-Fi guardadas:

```
netsh wlan show profiles
```

3. Muestra las credenciales de una red específica:

```
netsh wlan show profile "NOMBRE_SSID" key=clear
```

4. Ubica tu contraseña en el campo Contenido de la clave.

### 2. Linux

1. Abre una terminal.
2. Navega al directorio de NetworkManager :

```
cd /etc/NetworkManager/system-connections/
```

3. Lee los archivos de credenciales Wi-Fi (requiere permisos de root):

```
sudo cat NOMBRE_SSID.nmconnection
```

4. Encuentra tu contraseña después de psk= o password=.

> Asegúrate de tener acceso root para ejecutar comandos en Linux.

### 3. macOS

1. Abre la Terminal.
2. Usa el comando security para listar las credenciales:

```
security find-generic-password -ga "NOMBRE_SSID"
```

3. Introduce la contraseña de tu Mac si se te solicita.
4. Tu contraseña Wi-Fi aparecerá entre comillas junto a password:.

### 4. Android

Esta guía es para Android 10 o superior.

1. Ve a Ajustes > Red e Internet > Wi-Fi.
2. Selecciona la red a la que estás conectado y haz clic en Compartir.
3. Autentícate con huella digital o contraseña.
4. La contraseña aparece debajo del código QR generado o dentro del propio QR. Puedes escanearlo con otro teléfono.

> En dispositivos con root, puedes encontrar las contraseñas en `/data/misc/wifi/wpa_supplicant.conf`.

### 5. iOS

Esta guía es para iOS 16 o superior.

1. Ve a Ajustes > Wi-Fi.
2. Toca el icono ⓘ junto a tu red Wi-Fi .
3. Toca Contraseña y autentícate con Face ID, Touch ID o código de acceso.
4. La contraseña se mostrará claramente.

> Para versiones anteriores de iOS, utiliza la sincronización con el Llavero de Mac para acceder a las contraseñas.
