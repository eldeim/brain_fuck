# 21 - FTP

> El protocolo FTP (File Transfer Protocol) para la comunicación de control. En una conexión FTP, hay dos canales de comunicación: el canal de control y el canal de datos

### Connection

```bash
 ftp IP_TARGET
```

### Brute Force

```
hydra -l eldeim -P passwords.txt ftp://127.0.0.1 -t 10
```
