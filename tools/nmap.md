# Nmap

### Escaneo de Puertos Clásico

<pre class="language-bash"><code class="lang-bash"><strong>sudo nmap -p- --open -sS --min-rate 5000 -v -n -Pn -oG allPorts IP_TARGET
</strong></code></pre>

> `-sS` Especifica que se utilice un escaneo de tipo SYN (half-open scan)
>
> > SYN > (RST (closed) | SYN/ACK > RST) -- Se cierra la conexión forzadamente con un RST (generalmente se hace con un ACK y se guarda en un .log)
>
> `--min-rate`  (+ num.)  Se utiliza para establecer la velocidad mínima de envío de paquetes durante el escaneo
>
> `-oG` Guarda en formato grepeable, el escaneo de todos los puertos

### Escaneo de Servicios y Versiones

```bash
nmap -p22,25,80,139,445 -sCV -oN targeted 192.168.18.213
```

> `-sCV`  Scripts de detección por defecto de Nmap y detecta versiones de los servicios
>
> `-oN` Guarda en formato normal, el escaneo de versiones de todos los puertos

### Escaneo de Hosts

```
nmap -sP 20.20.20.0/24
```

> `-sP` Ping Scan (ahora `-sn` en versiones recientes de Nmap)

### Escaneos con Proxychains

```
proxychains nmap --top-ports 1000 -sT --open -T5 -v -n -Pn 10.10.10.128 2>&1 | grep -vE "timeout|OK"
```

> `-sT`  Es útil cuando **no se tiene acceso a privilegios de root** o cuando se necesita un escaneo básico sin configuraciones avanzadas como el **escaneo SYN**



