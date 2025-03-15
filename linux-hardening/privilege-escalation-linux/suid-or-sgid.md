# SUID or SGID

**SUID**: es un bit que se asigna a los ficheros. Permite que cualquier usuario que ejecute el programa lo haga con los mismos permisos que el creador de dicho fichero

**SGID**: igual que SUID pero se ejecuta con los permisos del grupo al que pertenece el creador. Podemos hacer una búsqueda de estos ficheros

Usando `find` de la siguiente manera:&#x20;

`find / -type f -perm -4000 2>/dev/null` # Muestra los ficheros con permisos SUID&#x20;

`find / -perm -4000 -user root 2>/dev/null` # Para users especificos

`find / -type f -perm -2000` # Muestra los ficheros con permisos SGID&#x20;

`find / -type -f -user root -perm -u=s 2>/dev/null` # Otra manera de mostrar los ficheros con permisos SUID&#x20;

`find / -perm +6000 2>/dev/null` # muestra los ficheros con bits SUID y SGID activos
