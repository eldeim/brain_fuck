# PATH Hijacking

El PATH en Linux es una variable de entorno que indica al sistema operativo dónde buscar ejecutables.

Una búsqueda sencilla de carpetas con permisos de escritura puede realizarse utilizando el comando `find / -writable 2>/dev/null`

Con `strings /archivo` podemos ver las líneas legibles de un archivo/binario

Luego, podemos modificar el PATH asi : `export PATH=/tmp:$PATH`

<figure><img src="../../.gitbook/assets/Pasted image 20250121143309.png" alt=""><figcaption></figcaption></figure>
