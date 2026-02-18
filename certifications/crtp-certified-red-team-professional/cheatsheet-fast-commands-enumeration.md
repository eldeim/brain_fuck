# 🏁 Cheatsheet - Fast Commands (ENUMERATION)

## Invisible Shells + Addons

<table><thead><tr><th width="134">Herramienta</th><th width="159">Para qué sirve</th><th width="410">Ejemplos de comandos</th></tr></thead><tbody><tr><td><strong>Invisi-Shell</strong></td><td>PowerShell stealth (AMSI + logging bypass)</td><td><p><code>C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat</code> </p><blockquote><p>or</p></blockquote><p><code>C:\AD\Tools\InviShell\RunWithPathAsAdmin.bat</code></p></td></tr><tr><td><strong>PowerView</strong></td><td>Enumeración ofensiva de Active Directory</td><td><p><code>. C:\AD\Tools\PowerView.ps1</code> </p><blockquote><p>Example Commands:</p><p><code>powershell Get-DomainUser powershell Get-DomainGroup</code> </p><p><code>powershell Find-InterestingDomainAcl</code> </p><p><code>powershell Get-DomainObjectAcl -Identity administrador -ResolveGUIDs</code></p></blockquote></td></tr><tr><td><strong>ADModule</strong></td><td>Módulo oficial de Microsoft para administrar AD</td><td><p><code>Import-Module C:\AD\Tools\ADModulemaster\Microsoft.ActiveDirectory.Management.dll</code> </p><blockquote><p>and</p></blockquote><p><code>Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1</code> </p><blockquote><p>Example Commands:</p><p><code>powershell Get-ADUser -Filter * powershell Get-ADGroup -Filter *</code></p></blockquote></td></tr></tbody></table>

### Using Invisi-Shell

> • With admin privileges:\
> `RunWithPathAsAdmin.bat`\
> • With non-admin privileges:\
> `RunWithRegistryNonAdmin.bat`\
> •  Type exit from the new PowerShell session to complete the clean-up.

```
cd \AD\Tools
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```

```
. C:\AD\Tools\PowerView.ps1
```

***

## All Enumeration - PowerView

* Domain Info
* Users
  * Member Computers
* Computers
* Domain Administrators
  * Members of the Domain Admins group
* Enterprise Administrators
  * Members of the Enterprise Administrators

{% embed url="https://eldeim.gitbook.io/brain_fuck/checklists/certifications/crtp-certified-red-team-professional/learning-objectives/learning-objetive-1#all-enumerations" %}

<table><thead><tr><th width="228">Commands</th><th width="136">Function</th><th>Example</th></tr></thead><tbody><tr><td><code>Get-Domain</code></td><td>General Info by Domain</td><td><pre><code><strong>Forest : moneycorp.local
</strong>DomainControllers : {dcorp-dc.dollarcorp.moneycorp.local}
Children : {us.dollarcorp.moneycorp.local}
</code></pre></td></tr><tr><td><code>Get-DomainUser | select -ExpandProperty samaccountname</code></td><td>List All Current Domain Users</td><td><pre><code><strong>Administrator
</strong><strong>Guest
</strong>[snip]
</code></pre></td></tr><tr><td><code>Get-DomainComputer | select -ExpandProperty dnshostname</code></td><td>List Names of Machines: DCs, Servers, PCs, WorkStations, etc...</td><td><pre><code>dcorp-dc.dollarcorp.moneycorp.local
dcorp-adminsrv.dollarcorp.moneycorp.local
dcorp-appsrv.dollarcorp.moneycorp.local
dcorp-ci.dollarcorp.moneycorp.local
dcorp-mgmt.dollarcorp.moneycorp.local
dcorp-mssql.dollarcorp.moneycorp.local
dcorp-sql1.dollarcorp.moneycorp.local
dcorp-stdadmin.dollarcorp.moneycorp.local
dcorp-std111.dollarcorp.moneycorp.local
[snip]
</code></pre></td></tr><tr><td><code>Get-DomainGroup -Identity "Domain Admins"</code></td><td>List Domain Admins: members, SID, descriptions, DN, etc..</td><td><pre><code>samaccountname : Domain Admins
member : {CN=svc admin,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local, CN=Administrator,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local}
</code></pre></td></tr><tr><td><p><code>Get-DomainGroupMember</code> </p><p><code>-Identity "Domain Admins"</code></p></td><td>List Doamin Admins into the forest</td><td><pre><code>MemberName : Administrator
MemberName : svcadmin
</code></pre></td></tr><tr><td><code>Get-DomainGroupMember</code></td><td>List members of the Enterprise Administrators</td><td><pre><code>GroupDomain : moneycorp.local
GroupName : Enterprise Admin
MemberName : Administrator
</code></pre></td></tr></tbody></table>

***

## All Enumeration - ADModule

* Domain Users
  * List Properties
* All Computers
* Enumerate Domain Administrators
* Enumerate the Enterprise Administrators

{% embed url="https://eldeim.gitbook.io/brain_fuck/checklists/certifications/crtp-certified-red-team-professional/learning-objectives/learning-objetive-1#using-the-active-directory-module-admodule" %}

<table><thead><tr><th width="228">Commands</th><th width="136">Function</th><th>Example</th></tr></thead><tbody><tr><td><code>Get-ADUser -Filter *</code></td><td>List all users in the current domain</td><td><pre><code>SamAccountName : Administrator
SID : S-1-5-21-719815819-3726368948-3917688648-500
SamAccountName : Guest
SID : S-1-5-21-719815819-3726368948-3917688648-501
</code></pre></td></tr><tr><td><code>Get-ADUser -Filter * -Properties *| select Samaccountname,Description</code></td><td>List specific properties: Samaccountname,Description</td><td><pre><code>Samaccountname Description
-------------- -----------
Administrator  Built-in account for administering the computer/domain
Guest          Built-in account for guest access to the computer/domain
krbtgt         Key Distribution Center Service Account
[snip]
</code></pre></td></tr><tr><td><code>Get-ADComputer -Filter *</code></td><td>List All Computers</td><td><pre><code>DistinguishedName : CN=DCORP-ADMINSRV,OU=Applocked,DC=dollarcorp,DC=moneycorp,DC=local
DNSHostName       : dcorp-adminsrv.dollarcorp.moneycorp.local
Enabled           : True
<a data-footnote-ref href="#user-content-fn-1">Name              : DCORP-ADMINSRV</a>
ObjectClass       : computer
ObjectGUID        : 2e036483-7f45-4416-8a62-893618556370
SamAccountName    : DCORP-ADMINSRV$
SID               : S-1-5-21-719815819-3726368948-3917688648-1105
[snip]
</code></pre></td></tr><tr><td><p><code>Get-ADGroupMember</code> </p><p><code>-Identity 'Domain Admins'</code></p></td><td>List Domain Admins: members, SID, descriptions, DN, etc..</td><td><pre><code>samaccountname : Domain Admins
member : {CN=svc admin,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local, CN=Administrator,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local}
</code></pre></td></tr><tr><td><code>Get-ADGroupMember -Identity 'Enterprise Admins' -Server moneycorp.local</code></td><td>List Doamin Admins into the forest</td><td><pre><code>distinguishedName : CN=Administrator,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local
name              : Administrator
objectClass       : user
objectGUID        : d954e824-f549-47c2-9809-646c218cef36
SamAccountName    : Administrator
SID               : S-1-5-21-719815819-3726368948-3917688648-500

distinguishedName : CN=svc admin,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local
name              : svc admin
objectClass       : user
objectGUID        : 244f9c84-7e33-4ed6-aca1-3328d0802db0
SamAccountName    : svcadmin
SID               : S-1-5-21-719815819-3726368948-3917688648-1118
</code></pre></td></tr></tbody></table>

***

## Bloodhound

{% embed url="https://eldeim.gitbook.io/brain_fuck/checklists/certifications/crtp-certified-red-team-professional/learning-objectives/learning-objetive-1#bloodhound" %}

We need to install the neo4j service. Unzip the archive C:\AD\Tools\neo4j-community-4.1.1-windows.zip

### Instalation

> Install and start the neo4j service as follows, into:
>
> ```
> cd C:\AD\Tools\neo4j-community-4.4.5-windows\neo4j-community-4.4.5\bin
> ```

> For it, we need a admin user to continue with the installation

```
neo4j.bat install-service
neo4j.bat start
```

Once the service is started, browse to [http://localhost:7474](http://localhost:7474/)

Enter the username: neo4j and password: neo4j. You need to enter a new password. Let's use BloodHound as the new password.

Now, open BloodHound from C:\AD\Tools\BloodHound-win32-x64\BloodHound-win32-x64 and provide the following details:

> bolt://localhost:7687
>
> Username: neo4j Password: BloodHound

### Ingestores

Run BloodHound ingestores to gather data and information about the current domain. Run the following commands to run Collector:

```
C:\AD\Tools\BloodHound-master\BloodHound-master\Collectors\SharpHound.exe --collectionmethods Group,GPOLocalGroup,Session,Trusts,ACL,Container,ObjectProps,SPNTargets --excludedcs
```

```
C:\AD\Tools\>C:\AD\Tools\BloodHound-master\BloodHound-master\Collectors\SharpHound.exe --collectionmethods Group,GPOLocalGroup,Session,Trusts,ACL,Container,ObjectProps,SPNTargets --excludedcs
[+] Successfully unhooked ETW!
[+++] NTDLL.DLL IS UNHOOKED!
[+++] KERNEL32.DLL IS UNHOOKED!
[+++] KERNELBASE.DLL IS UNHOOKED!
[+++] ADVAPI32.DLL IS UNHOOKED!
[+] URL/PATH : C:\AD\Tools\BloodHound-master\BloodHound-master\Collectors\SharpHound.exe Arguments : --collectionmethods Group,GPOLocalGroup,Session,Trusts,ACL,Container,ObjectProps,SPNTargets ?excludedcs
[snip]
2024-12-19T02:51:45.7390124-08:00|INFORMATION|SharpHound Enumeration Completed at 2:51 AM on 12/19/2024! Happy Graphing!
```

Once all the data is uploaded to BloodHound, search for shortest path to Domain Admins in dollarcorp domain. (press Ctrl to toggle labels).

***

## PowerHuntShares

Run the following commands from a PowerShell session started using [Invisi-Shell](https://eldeim.gitbook.io/brain_fuck/checklists/certifications/crtp-certified-red-team-professional/learning-objectives/learning-objetive-1#using-the-active-directory-module-admodule):

> After this, we need save into a file txt in C:\AD\Tools, all Domain Computer, extract its using:
>
> ```
> Get-DomainComputer | select -ExpandProperty dnshostname
> ```

```
PS C:\AD\Tools> notepad servers.txt
## Paste the servers
cat C:\AD\Tools\servers.txt
```

```
Import-Module C:\AD\Tools\PowerHuntShares.psm1
Invoke-HuntSMBShares -NoPing -OutputDirectory C:\AD\Tools\ -HostList C:\AD\Tools\servers.txt
```

> It generate us a .htlm in the same folder
>
> You need to copy the summary report to your host machine because the report needs interent access, which is not available on the student VM.

Go to ShareGraph -> search dcorp-ci -> Right click on dcorp-ci node -> Click expand. Tt turns out that 'Everyone' has privileges on the 'AI' folder.

***

## ACLs Enumeration - Domain Admins Group

{% embed url="https://eldeim.gitbook.io/brain_fuck/checklists/certifications/crtp-certified-red-team-professional/learning-objectives/learning-objetive-2#enumerate-acls-for-the-domain-admins-group" %}

> Remember to continúe using the PowerShell session started using Invisi-Shell

<table><thead><tr><th width="228">Commands</th><th width="136">Function</th><th>Example</th></tr></thead><tbody><tr><td><code>Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs</code></td><td>List all users in the current domain</td><td><pre><code>AceQualifier           : AccessAllowed
ObjectDN               : CN=Domain Admins,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local
ActiveDirectoryRights  : ReadProperty
ObjectAceType          : User-Account-Restrictions
ObjectSID              : S-1-5-21-719815819-3726368948-3917688648-512
InheritanceFlags       : None
</code></pre></td></tr></tbody></table>

### Excesive Permissions - On us account

Finally, to check for modify rights/permissions for the studentx, we can use Find-InterestingDomainACL from PowerView:

```
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "student113"}
```

### Member of the RDPUsers group&#xD;

> Note that the output in your lab for the below command will be different and will depend on your lab instance:

<table><thead><tr><th width="228">Commands</th><th width="136">Function</th><th>Example</th></tr></thead><tbody><tr><td><code>Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}</code></td><td>List all users in the current domain</td><td><pre><code>ObjectDN                : CN=ControlxUser,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local
AceQualifier            : AccessAllowed
ActiveDirectoryRights   : GenericAll
ObjectAceType           : None
AceFlags                : None
AceType                 : AccessAllowed
InheritanceFlags        : None
SecurityIdentifier      : S-1-5-21-719815819-3726368948-3917688648-1123
IdentityReferenceName   : RDPUsers
IdentityReferenceDomain : dollarcorp.moneycorp.local
IdentityReferenceDN     : CN=RDP Users,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local
IdentityReferenceClass  : group
[snip]
</code></pre></td></tr></tbody></table>

### Analyze the permissions - BloodHound UI

> Note that it is easier to analyze ACLs using BloodHound as it shows interesting ACLs for the user and the groups it is a member of. Let's look at the 'Outbound Object Control' for the studentx in the BloodHound CE UI:

<figure><img src="https://eldeim.gitbook.io/brain_fuck/~gitbook/image?url=https%3A%2F%2F3697469405-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSrcwXlKGkwhzbrKa8vLU%252Fuploads%252FYEpsbPeJVuxnn9TDIbzx%252Fimage.png%3Falt%3Dmedia%26token%3Dec0dd414-580e-42dd-b12b-02caedb13579&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=d69bf187&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Multiple permissions stand out in the above diagram. Due to the membership of the RDPUsers group, the studentx user has following interesting permissions

* Full Control/Generic All over supportx and controlx users.
* Enrollment permissions on multiple certificate templates.
* Full Control/Generic All on the Applocked Group Policy.

<figure><img src="https://eldeim.gitbook.io/brain_fuck/~gitbook/image?url=https%3A%2F%2F3697469405-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSrcwXlKGkwhzbrKa8vLU%252Fuploads%252FXmy9FfZOxfrmYm3Q1IaT%252Fimage.png%3Falt%3Dmedia%26token%3Dca1979bc-79a5-4f91-a6c5-d26bd1e6e457&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=d5d3c98d&#x26;sv=2" alt=""><figcaption></figcaption></figure>

***

## Enumerate OUs

<table><thead><tr><th width="228">Commands</th><th width="136">Function</th><th>Example</th></tr></thead><tbody><tr><td><code>Get-DomainOU</code></td><td>Enumerate folders and his complete info of AD</td><td><pre><code>description            : Default container for domain controllers
systemflags            : -1946157056
iscriticalsystemobject : True
gplink                 : [LDAP://CN={6AC1786C-016F-11D2-945F-00C04fB984F9},CN=Policies,CN=System,DC=dollarcorp,DC=moneycorp,DC=local;0]
whenchanged            : 11/12/2022 5:59:00 AM
objectclass            : {top, organizationalUnit}
showinadvancedviewonly : False
usnchanged             : 7921
dscorepropagationdata  : {11/15/2022 3:49:24 AM, 11/12/2022 5:59:41 AM, 1/1/1601 12:04:16 AM}
name                   : Domain Controllers
distinguishedname      : OU=Domain Controllers,DC=dollarcorp,DC=moneycorp,DC=local
ou                     : Domain Controllers
</code></pre></td></tr><tr><td><p><code>Get-DomainOU | select</code> </p><p><code>-ExpandProperty name</code></p></td><td>Enumerate only the names of the folder of AD</td><td><pre><code><strong>Domain Controllers
</strong>StudentMachines
Applocked
Servers
DevOps
</code></pre></td></tr><tr><td><code>(Get-DomainOU -Identity DevOps).distinguishedname | %{Get-DomainComputer -SearchBase $_} | select name</code></td><td>List all the computers in the DevOps OU</td><td><pre><code>name
----
DCORP-CI
[snip]
</code></pre></td></tr></tbody></table>

***

## Enumerate GPOs

<table><thead><tr><th width="228">Commands</th><th width="136">Function</th><th>Example</th></tr></thead><tbody><tr><td><code>Get-DomainGPO</code></td><td>List info GPOs of AD</td><td><pre><code>flags                    : 0
displayname              : DevOps Policy
gpcmachineextensionnames : [{35378EAC-683F-11D2-A89A-00C04FBBCFA2}{D02B1F72-3407-48AE-BA88-E8213C6761F1}][{827D319E-6EAC-11D2-A4EA-00C04F79F83A}{803E14A0-B4FB-11D0-A0D0-00A0C90F574B}]
whenchanged              : 12/19/2024 12:00:15 PM
versionnumber            : 3
name                     : {0BF8D01C-1F62-4BDC-958C-57140B67D147}
cn                       : {0BF8D01C-1F62-4BDC-958C-57140B67D147}
usnchanged               : 314489
dscorepropagationdata    : {12/18/2024 7:31:56 AM, 1/1/1601 12:00:00 AM}
objectguid               : fc0df125-5e26-4794-93c7-e60c6eecb75f
gpcfilesyspath           : \\dollarcorp.moneycorp.local\SysVol\dollarcorp.moneycorp.local\Policies\{0BF8D01C-1F62-4BDC-958C-57140B67D147}
distinguishedname        : CN={0BF8D01C-1F62-4BDC-958C-57140B67D147},CN=Policies,CN=System,DC=dollarcorp,DC=moneycorp,DC=local
whencreated              : 12/18/2024 7:31:22 AM
showinadvancedviewonly   : True
usncreated               : 293100
gpcfunctionalityversion  : 2
instancetype             : 4
objectclass              : {top, container, groupPolicyContainer}
objectcategory           : CN=Group-Policy-Container,CN=Schema,CN=Configuration,DC=moneycorp,DC=local
[snip]
</code></pre></td></tr><tr><td><p><code>(Get-DomainOU</code> </p><p><code>-Identity DevOps).gplink</code></p></td><td>Get GUID of specific police</td><td><pre><code>[LDAP://cn={0BF8D01C-1F62-4BDC-958C-57140B67D147},cn=policies,cn=system,DC=dollarcorp,DC=moneycorp,DC=local;0]
</code></pre></td></tr><tr><td><p><code>Get-DomainGPO</code> </p><p><code>-Identity '{GUID}'</code></p></td><td>List all datails of specific GPO</td><td><pre><code>flags                    : 0
displayname              : DevOps Policy
gpcmachineextensionnames : [{35378EAC-683F-11D2-A89A-00C04FBBCFA2}{D02B1F72-3407-48AE-BA88-E8213C6761F1}][{827D319E-6EAC-11D2-A4EA-00C04F79F83A}{803E14A0-B4FB-11D0-A0D0-00A0C90F574B}]
whenchanged              : 12/19/2024 12:00:15 PM
versionnumber            : 3
name                     : {0BF8D01C-1F62-4BDC-958C-57140B67D147}
cn                       : {0BF8D01C-1F62-4BDC-958C-57140B67D147}
usnchanged               : 314489
dscorepropagationdata    : {12/18/2024 7:31:56 AM, 1/1/1601 12:00:00 AM}
objectguid               : fc0df125-5e26-4794-93c7-e60c6eecb75f
gpcfilesyspath           : \\dollarcorp.moneycorp.local\SysVol\dollarcorp.moneycorp.local\Policies\{0BF8D01C-1F62-4BDC-958C-57140B67D147}
distinguishedname        : CN={0BF8D01C-1F62-4BDC-958C-57140B67D147},CN=Policies,CN=System,DC=dollarcorp,DC=moneycorp,DC=local
whencreated              : 12/18/2024 7:31:22 AM
showinadvancedviewonly   : True
usncreated               : 293100
gpcfunctionalityversion  : 2
instancetype             : 4
objectclass              : {top, container, groupPolicyContainer}
objectcategory           : CN=Group-Policy-Container,CN=Schema,CN=Configuration,DC=moneycorp,DC=local
</code></pre></td></tr></tbody></table>

***

### Enumerate ACLs of GPOs

To enumerate the ACLs for the Applocked and DevOps GPO, let's use the BloodHound CE UI.

Search for Applocker in the UI -> Click on the node -> Click on Inboud Object Control

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

It turns out that the RDPUsers group has GenericAll over the policy.

Similary, search for DevOps and look at its 'Inbound Object Control':

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

A user named 'devopsadmin' has 'WriteDACL' on DevOps Policy.

***

## Enumerate all domains in the current Forest

> Note: Remenber use a silent powershell

```
Get-ForestDomain -Verbose
```

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

***

### Enumerate all Trust of "dollarcorp" Domain

> Note: Remenber use a silent powershell

```
Get-DomainTrust
```

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### List external trusts & Extact Infromation

{% embed url="https://eldeim.gitbook.io/brain_fuck/checklists/certifications/crtp-certified-red-team-professional/learning-objectives/learning-objetive-4#list-only-the-external-trusts-in-the-moneycorp.local-forest" %}

<table><thead><tr><th width="274">Commands</th><th width="136">Function</th><th>Example</th></tr></thead><tbody><tr><td><p><code>Get-ForestDomain</code> </p><p><code>| %{Get-DomainTrust</code> </p><p><code>-Domain $_</code><em><code>.Name} | ?{$_.</code></em><code>TrustAttributes -eq</code> </p><p><code>"FILTER_SIDS"}</code></p></td><td>List only the external trusts in the "moneycorp.local" forest</td><td><pre><code><strong>SourceName      : dollarcorp.moneycorp.local
</strong>TargetName      : eurocorp.local
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FILTER_SIDS
TrustDirection  : Bidirectional
WhenCreated     : 11/12/2022 8:15:23 AM
WhenChanged     : 1/25/2026 4:16:41 AM
</code></pre></td></tr><tr><td><code>Get-DomainTrust | ?{$_.TrustAttributes -eq "FILTER_SIDS"}</code></td><td>Enumerate external trusts of the "dollarcorp" domain</td><td><pre><code>SourceName      : dollarcorp.moneycorp.local
TargetName      : eurocorp.local
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FILTER_SIDS
TrustDirection  : Bidirectional
WhenCreated     : 11/12/2022 8:15:23 AM
WhenChanged     : 1/25/2026 4:16:41 AM
</code></pre><blockquote><p>Since the above is a Bi-Directional trust, we can extract information from the eurocorp.local forest.</p><blockquote><p>We either need bi-directional trust or one-way trust from eurocorp.local to dollarcorp to be able to use the below command</p></blockquote></blockquote></td></tr><tr><td>Get-ForestDomain -Forest eurocorp.local | %{Get-DomainTrust -Domain $_.Name}</td><td>Extract information from the eurocorp.local forest</td><td><pre><code><strong>SourceName      : dollarcorp.moneycorp.local
</strong><strong>TargetName      : eurocorp.local
</strong><strong>TrustType       : WINDOWS_ACTIVE_DIRECTORY
</strong><strong>TrustAttributes : FILTER_SIDS
</strong><strong>TrustDirection  : Bidirectional
</strong><strong>WhenCreated     : 11/12/2022 8:15:23 AM
</strong><strong>WhenChanged     : 1/25/2026 4:16:41 AM
</strong></code></pre><blockquote><p>Notice the error above. It occurred because PowerView attempted to list trusts even for eu.eurocorp.local. Because external trust is non-transitive it was not possible!</p></blockquote></td></tr></tbody></table>

***

### Using AD Module in a PowerShell - Invisi-Shell <a href="#a-d-module-in-a-powershell-invisi-shell" id="a-d-module-in-a-powershell-invisi-shell"></a>

> Import the AD Module in a PowerShell session started using Invisi-Shell:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll
Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
```

<table><thead><tr><th width="274">Commands</th><th width="136">Function</th><th>Example</th></tr></thead><tbody><tr><td><code>(Get-ADForest).Domains</code></td><td>Enumerate all the domains</td><td><pre><code>dollarcorp.moneycorp.local
moneycorp.local
us.dollarcorp.moneycorp.local
</code></pre></td></tr><tr><td><code>Get-ADTrust -Filter *</code></td><td>Enumerate all the Trusts in the current domain</td><td><pre><code>Direction : BiDirectional
DisallowTransivity : False
DistinguishedName :
CN=moneycorp.local,CN=System,DC=dollarcorp,DC=moneycorp,DC=local
ForestTransitive : False
IntraForest : True
IsTreeParent : False
IsTreeRoot : False
Name : moneycorp.local
ObjectClass : trustedDomain
ObjectGUID : 01c3b68d-520b-44d8-8e7f-4c10927c2b98
SelectiveAuthentication : False
SIDFilteringForestAware : False
SIDFilteringQuarantined : False
Source : DC=dollarcorp,DC=moneycorp,DC=local
Target : moneycorp.local
TGTDelegation : False
TrustAttributes : 32
TrustedPolicy :
TrustingPolicy :
TrustType : Uplevel
UplevelOnly : False
UsesAESKeys : False
UsesRC4Encryption : False
[snip]
</code></pre></td></tr><tr><td><p><code>Get-ADForest | %{Get-ADTrust</code></p><p> <code>-Filter *}</code></p></td><td>Enumerate all the trusts in the moneycorp.local forest</td><td><pre><code>Direction : BiDirectional
DisallowTransivity : False
DistinguishedName :
CN=moneycorp.local,CN=System,DC=dollarcorp,DC=moneycorp,DC=local
ForestTransitive : False
IntraForest : True
IsTreeParent : False
IsTreeRoot : False
Name : moneycorp.local
ObjectClass : trustedDomain
ObjectGUID : 01c3b68d-520b-44d8-8e7f-4c10927c2b98
SelectiveAuthentication : False
SIDFilteringForestAware : False
SIDFilteringQuarantined : False
Source : DC=dollarcorp,DC=moneycorp,DC=local
Target : moneycorp.local
TGTDelegation : False
TrustAttributes : 32
TrustedPolicy :
TrustingPolicy :
TrustType : Uplevel
UplevelOnly : False
UsesAESKeys : False
UsesRC4Encryption : False
[snip]
</code></pre></td></tr><tr><td><code>(Get-ADForest).Domains | %{Get-ADTrust -Filter '(intraForest -ne $True) -and (ForestTransitive -ne $True)' -Server $_}</code></td><td>Enumerate external trusts in moneycorp.local domain</td><td><pre><code>Direction : BiDirectional
DisallowTransivity : False
DistinguishedName :
CN=eurocorp.local,CN=System,DC=dollarcorp,DC=moneycorp,DC=local
ForestTransitive : False
IntraForest : False
IsTreeParent : False
IsTreeRoot : False
Name : eurocorp.local
ObjectClass : trustedDomain
ObjectGUID : d4d64a77-63be-4d77-93c2-6524e73d306d
SelectiveAuthentication : False
SIDFilteringForestAware : False
SIDFilteringQuarantined : True
Source : DC=dollarcorp,DC=moneycorp,DC=local
Target : eurocorp.local
TGTDelegation : False
TrustAttributes : 4
TrustedPolicy :
TrustingPolicy :
TrustType : Uplevel
UplevelOnly : False
UsesAESKeys : False
UsesRC4Encryption : False	
</code></pre></td></tr></tbody></table>

[^1]: 
