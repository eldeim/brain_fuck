# 🌋 Privilege Escalation - Windows

## Enumeration

Obtener acceso a usuarios puede ser sencillo, como encontrar credenciales en archivos de texto (.txt) u hojas de cálculo (.xls) que algún usuario haya dejado sin proteger, pero no siempre será así. Dependiendo de la situación, podríamos necesitar abusar de algunas de las siguientes debilidades:

* Configuraciones erróneas en servicios de Windows o tareas programadas
* Privilegios excesivos asignados a nuestra cuenta
* Software vulnerable
* Falta de parches de seguridad de Windows

Hay distintos tipos de usuarios en un sistema Windows

### Windows Users

1º **Administrators** : Son los usuarios con más privilegios. Pueden modificar cualquier parámetro de configuración del sistema y acceder a cualquier fichero del mismo.

2º **Standard Users** : Estos usuarios pueden acceder al ordenador pero sólo realizar tareas limitadas. Normalmente estos usuarios no pueden realizar cambios permanentes o esenciales en el sistema y están limitados a sus archivos.

Cualquier usuario con privilegios de adminstrador formará parte del grupo **Administrators.** Por otro lado, los usuarios estándar forman parte del grupo **Users**

### Otros Comandos

`dir C:\ /s /p /a:-d | findstr flag.txt`&#x20;

or&#x20;

`where /r C:\ flag.txt`

***

`whoami /groups`

> Lista todos los grupos a los que pertenece el usuario actual y los permisos asignados a cada grupo

`whoami /priv`

> Muestra los privilegios que tiene el usuario actual, como el acceso a tomar posesión de archivos, reiniciar el sistema, etc

## Automated Enumeration Tools

WinPEAS: [https://github.com/itm4n/PrivescCheck](https://github.com/itm4n/PrivescCheck)

`C:> winpeas.exe > outputfile.txt`



