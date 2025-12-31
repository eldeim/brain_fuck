# 🏔️ Privilege Escalation - Linux

## Enumeration

`lsb_release`

> Devuelve información sobre la distribución instalada

`hostname`

> Devuelve el hostname de la maquina. (e.g. Ubuntu-3487340239)

`uname -a`

> Devuelve información adicional sobre el kernel usado en el sistema

`which /algun_ejecutable-base64`

> Se utiliza para localizar la ruta completa de un ejecutable en el sistema $PATH

`/proc/version`

> Contiene información sobre la versión del kernel y datos adicionales como si un compilador (por ejemplo, GCC) está instalado

`/etc/issue`

> Este archivo suele contener información sobre el sistema operativo, pero puede personalizarse o modificarse fácilmente.

`env`

> Muestra variables de entorno (a veces los users guardan contraseñas aqui)

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

`sudo -l`

> Listar todos los comandos que tu usuario puede ejecutar usando `sudo`

`id`

> Muestra una visión general de los privilegios del usuario y su grupo![](<../../.gitbook/assets/Pasted image 20250120135343 (1).png>)

`/etc/passwd`

> Puedes leer el archivo `etc/passwd` para identificar los usuario que hay en el sistema

<figure><img src="../../.gitbook/assets/Pasted image 20250120135456.png" alt=""><figcaption></figcaption></figure>

### Netstat Command

El comando `netstat` puede utilizarse con varias opciones diferentes para recopilar información sobre las conexiones existentes: `netstat -a` : Muestra todos los puertos de escucha y las conexiones establecidas. `netstat -at or netstat-au` : Se utilizarse para listar protocolos TCP o UDP `netstat -l` : Lista los puertos en modo «escucha». Se puede utilizar con la opción «-t» para listar sólo los puertos que están a la escucha utilizando el protocolo TCP

<figure><img src="../../.gitbook/assets/Pasted image 20250120140100.png" alt=""><figcaption></figcaption></figure>

`netstat -s` Lista estadísticas de uso de red por protocolo. Tambien se puede utilizar con las opciones -t o -u para limitar la salida a un protocolo específico![](<../../.gitbook/assets/Pasted image 20250120140242.png>)

`netstat -tp` : Lista las conexiones con el nombre del servicio y la información PID. También se puede utilizar con la opción `-l` para listar los puertos de escucha&#x20;

<figure><img src="../../.gitbook/assets/Pasted image 20250120140504.png" alt=""><figcaption></figcaption></figure>

&#x20;`netstat -i` : Muestra las estadísticas de la interfaz

<figure><img src="../../.gitbook/assets/Pasted image 20250120140659 (1).png" alt=""><figcaption></figcaption></figure>

### Find Command

`find` files:

`find . -name flag1.txt` : Encuentra el archivo llamado «flag1.txt» en el directorio actual&#x20;

`find /home -name flag1.txt` : Encuentra el archivo llamado «flag1.txt» en el directorio /home&#x20;

`find / -type d -name config` : Busca el directorio config en «/».&#x20;

`find / -type f -perm 0777` : Busca los archivos con permisos 777 (archivos legibles, escribibles y ejecutables por todos los usuarios)&#x20;

`find / -perm a=x` : Busca ficheros ejecutables&#x20;

`find /home -user frank` : Busca todos los archivos del usuario «frank» en «/home».&#x20;

`find / -mtime 10` : Busca los archivos modificados en los últimos 10 días&#x20;

`find / -atime 10` : Busca los archivos a los que se ha accedido en los últimos 10 días&#x20;

`find / -cmin -60` : Busca los archivos modificados en la última hora (60 minutos)&#x20;

`find / -amin -60` : Busca los archivos a los que se ha accedido en la última hora (60 minutos)&#x20;

`find / -size 50M` : Buscar ficheros con un tamaño de 50 MB

### Ps Command

La salida del `ps` (Estado del Proceso) mostrará lo siguiente;

PID: El ID del proceso (único para el proceso) TTY: Tipo de terminal utilizado por el usuario Tiempo: Cantidad de tiempo de CPU utilizado por el proceso (NO es el tiempo que el proceso ha estado ejecutándose) CMD: El comando o ejecutable que se está ejecutando (NO mostrará ningún parámetro de la línea de comandos)

`ps` tiene alguna opciones utiles:&#x20;

`ps aux` : Muestra todos los procesos de todos los usuarios&#x20;

`ps -A` : Devuelve todos los procesos en ejecución, pero muestra menos información&#x20;

`ps axjf` : Devuelve en forma de tree, los procesos

<figure><img src="../../.gitbook/assets/Pasted image 20250120134255.png" alt=""><figcaption></figcaption></figure>

### Automated Enumeration Tools

* <mark style="color:green;">LinPeas</mark> : https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS

