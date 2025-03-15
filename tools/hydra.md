# Hydra

> Es una herramienta de hacking que se utiliza para llevar a cabo ataques de fuerza bruta en diversos sistemas
>
> Es remendable usar la flag `-u`  si el diccionario de usuarios es pequeño.

### Brute Force

#### RDP

```bash
hydra -V -f -L users.txt -P passwords.txt rdp ://<IP> -t 20
```

#### SMB

```bash
hydra -L users -P passwd smb://IP_TARGET -t 10
```

#### SSH

```bash
hydra -L users.txt -P passwd.txt ssh://IP_TARGET -t 15 -u
```

#### FTP

```bash
hydra -l eldeim -P passwords.txt ftp://127.0.0.1 -t 10
```
