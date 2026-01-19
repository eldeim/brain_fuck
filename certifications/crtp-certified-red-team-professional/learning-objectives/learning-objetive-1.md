# Learning Objetive 1

<figure><img src="../../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

Start a PowerShell session using Invisi-Shell to avoid enhanced logging. Run the below command from a command prompt on the student VM:

## Bypassing PowerShell Security

| Herramienta                 | Para qué sirve                                  | Ejemplos de comandos                                                                                                                                                                                                                                       |
| --------------------------- | ----------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Invisi-Shell**            | PowerShell stealth (AMSI + logging bypass)      | `bat C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat bat C:\AD\Tools\InviShell\RunWithPathAsAdmin.bat`                                                                                                                                                   |
| **RunWithPathAsAdmin**      | Invisi-Shell con privilegios de admin           | `bat RunWithPathAsAdmin.bat`                                                                                                                                                                                                                               |
| **RunWithRegistryNonAdmin** | Invisi-Shell sin privilegios de admin           | `bat RunWithRegistryNonAdmin.bat`                                                                                                                                                                                                                          |
| **PowerView**               | Enumeración ofensiva de Active Directory        | `powershell . C:\AD\Tools\PowerView.ps1 powershell Get-DomainUser powershell Get-DomainGroup powershell Find-InterestingDomainAcl powershell Get-DomainObjectAcl -Identity administrador -ResolveGUIDs`                                                    |
| **ADModule**                | Módulo oficial de Microsoft para administrar AD | `powershell Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll powershell Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1 powershell Get-ADUser -Filter * powershell Get-ADGroup -Filter *` |

### Using Invisi-Shell

> • With admin privileges:\
> RunWithPathAsAdmin.bat\
> • With non-admin privileges:\
> RunWithRegistryNonAdmin.bat\
> • Type exit from the new PowerShell session to complete the clean-up.

```
cd \AD\Tools
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```

```
. C:\AD\Tools\PowerView.ps1
```

Once, we do the bypass of poweshell, we proceed to enumerate all...

## All Enumerations

### Domain-Enum

```
Get-Domain

Forest                  : moneycorp.local
DomainControllers       : {dcorp-dc.dollarcorp.moneycorp.local}
Children                : {us.dollarcorp.moneycorp.local}
DomainMode              : Unknown
DomainModeLevel         : 7
Parent                  : moneycorp.local
PdcRoleOwner            : dcorp-dc.dollarcorp.moneycorp.local
RidRoleOwner            : dcorp-dc.dollarcorp.moneycorp.local
InfrastructureRoleOwner : dcorp-dc.dollarcorp.moneycorp.local
Name                    : dollarcorp.moneycorp.local
```

### Users

```
Get-DomainUser | select -ExpandProperty samaccountname

Administrator
Guest
DefaultAccount
krbtgt
ciadmin
sqladmin
srvadmin
mgmtadmin
appadmin
sql1admin
svcadmin
testda
[snip]
```

### Member Computers

Now, to enumerate member computers in the domain we can use Get-DomainComputer:

```
Get-DomainComputer | select -ExpandProperty dnshostname

dcorp-dc.dollarcorp.moneycorp.local
dcorp-adminsrv.dollarcorp.moneycorp.local
dcorp-appsrv.dollarcorp.moneycorp.local
dcorp-ci.dollarcorp.moneycorp.local
dcorp-mgmt.dollarcorp.moneycorp.local
dcorp-mssql.dollarcorp.moneycorp.local
dcorp-sql1.dollarcorp.moneycorp.local
dcorp-stdadmin.dollarcorp.moneycorp.local
dcorp-std111.dollarcorp.moneycorp.local
dcorp-std112.dollarcorp.moneycorp.local
dcorp-std113.dollarcorp.moneycorp.local
dcorp-std114.dollarcorp.moneycorp.local
dcorp-std115.dollarcorp.moneycorp.local
dcorp-std116.dollarcorp.moneycorp.local
dcorp-std117.dollarcorp.moneycorp.local
dcorp-std118.dollarcorp.moneycorp.local
dcorp-std119.dollarcorp.moneycorp.local
dcorp-std120.dollarcorp.moneycorp.local
dcorp-std121.dollarcorp.moneycorp.local
dcorp-std122.dollarcorp.moneycorp.local
dcorp-std123.dollarcorp.moneycorp.local
dcorp-std124.dollarcorp.moneycorp.local
dcorp-std125.dollarcorp.moneycorp.local
dcorp-std126.dollarcorp.moneycorp.local
dcorp-std127.dollarcorp.moneycorp.local
dcorp-std128.dollarcorp.moneycorp.local
dcorp-std129.dollarcorp.moneycorp.local
dcorp-std130.dollarcorp.moneycorp.local
```

### Domain Admins group

```
Get-DomainGroup -Identity "Domain Admins"

grouptype              : GLOBAL_SCOPE, SECURITY
admincount             : 1
iscriticalsystemobject : True
samaccounttype         : GROUP_OBJECT
samaccountname         : Domain Admins
whenchanged            : 11/14/2022 5:06:37 PM
objectsid              : S-1-5-21-719815819-3726368948-3917688648-512
name                   : Domain Admins
cn                     : Domain Admins
instancetype           : 4
usnchanged             : 40124
dscorepropagationdata  : {1/17/2026 5:28:28 PM, 1/17/2026 4:28:27 PM, 1/17/2026 3:28:27 PM, 1/17/2026 2:28:27 PM...}
objectguid             : 7d766421-bcf7-40b1-a970-17da0bedb489
description            : Designated administrators of the domain
memberof               : {CN=Denied RODC Password Replication Group,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local, CN=Administrators,CN=Builtin,DC=dollarcorp,DC=moneycorp,DC=local}
member                 : {CN=svc admin,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local, CN=Administrator,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local}
usncreated             : 12315
whencreated            : 11/12/2022 5:59:41 AM
distinguishedname      : CN=Domain Admins,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local
objectclass            : {top, group}
objectcategory         : CN=Group,CN=Schema,CN=Configuration,DC=moneycorp,DC=local
```

> * The most important:
>
> samaccountname : Domain Admins
>
> member : {CN=svc admin,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local, CN=Administrator,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local}

### Members of the Domain Admins group

```
Get-DomainGroupMember -Identity "Domain Admins"

GroupDomain             : dollarcorp.moneycorp.local
GroupName               : Domain Admins
GroupDistinguishedName  : CN=Domain Admins,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local
MemberDomain            : dollarcorp.moneycorp.local
MemberName              : svcadmin
MemberDistinguishedName : CN=svc admin,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local
MemberObjectClass       : user
MemberSID               : S-1-5-21-719815819-3726368948-3917688648-1118

GroupDomain             : dollarcorp.moneycorp.local
GroupName               : Domain Admins
GroupDistinguishedName  : CN=Domain Admins,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local
MemberDomain            : dollarcorp.moneycorp.local
MemberName              : Administrator
MemberDistinguishedName : CN=Administrator,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local
MemberObjectClass       : user
MemberSID               : S-1-5-21-719815819-3726368948-3917688648-500
```

> We can see the MemberName : svcadmin and Administrator withhis MemberSID

### &#x20;Members of the Enterprise Admins group

```
Get-DomainGroupMember -Identity "Enterprise Admins"
```

Since, this is not a root domain, the above command will return nothing. We need to query the root domain as Enterprise Admins group is present only in the root of a forest.

```
Get-DomainGroupMember -Identity "Enterprise Admins" -Domain moneycorp.local

GroupDomain             : moneycorp.local
GroupName               : Enterprise Admins
GroupDistinguishedName  : CN=Enterprise Admins,CN=Users,DC=moneycorp,DC=local
MemberDomain            : moneycorp.local
MemberName              : Administrator
MemberDistinguishedName : CN=Administrator,CN=Users,DC=moneycorp,DC=local
MemberObjectClass       : user
MemberSID               : S-1-5-21-335606122-960912869-3279953914-500
```

## Using the Active Directory module (ADModule)

### Using Invisi-Shell

> • With admin privileges:\
> RunWithPathAsAdmin.bat\
> <mark style="color:$danger;background-color:$warning;">• With non-admin privileges:</mark>\ <mark style="color:$danger;background-color:$warning;">RunWithRegistryNonAdmin.bat</mark>\
> • Type exit from the new PowerShell session to complete the clean-up.

Let's import the ADModule. <mark style="background-color:yellow;">Remember to use it from a different PowerShell session started by using Invisi-Shell.</mark> If you load PowerView and the ADModule in same PowerShell session, some functions may not work:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```

```
Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll
Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
```

### Domain Users

Enumerate all the users in the current domain using the ADModule

```
Get-ADUser -Filter *

DistinguishedName :
CN=Administrator,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local
Enabled : True
GivenName :
Name : Administrator
ObjectClass : user
ObjectGUID : d954e824-f549-47c2-9809-646c218cef36
SamAccountName : Administrator
SID : S-1-5-21-719815819-3726368948-3917688648-500
Surname :
UserPrincipalName :
DistinguishedName : CN=Guest,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local
Enabled : False
GivenName :
Name : Guest
ObjectClass : user
ObjectGUID : caa69143-af4c-4551-af91-e9edd1059080
SamAccountName : Guest
SID : S-1-5-21-719815819-3726368948-3917688648-501
[snip]
```

### List Properties

We can list specific properties. Let's list samaccountname and description for the users. Note that we are listing all the proeprties first using the -Properties parameter:

```
Get-ADUser -Filter * -Properties *| select Samaccountname,Description
```

<figure><img src="../../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

### All Computers

For the next task, list all the computers:

```
Get-ADComputer -Filter *

S C:\AD\Tools>Get-ADComputer -Filter *     
DistinguishedName : CN=DCORP-DC,OU=Domain
Controllers,DC=dollarcorp,DC=moneycorp,DC=local
DNSHostName : dcorp-dc.dollarcorp.moneycorp.local
Enabled : True
Name : DCORP-DC
ObjectClass : computer
ObjectGUID : d698b7ab-f29e-461b-9bc9-24a4a131c92d
SamAccountName : DCORP-DC$
SID : S-1-5-21-719815819-3726368948-3917688648-1000
UserPrincipalName :
DistinguishedName : CN=DCORPADMINSRV,OU=Applocked,DC=dollarcorp,DC=moneycorp,DC=local
DNSHostName : dcorp-adminsrv.dollarcorp.moneycorp.local
Enabled : True
Name : DCORP-ADMINSRV
ObjectClass : computer
ObjectGUID : 2e036483-7f45-4416-8a62-893618556370
SamAccountName : DCORP-ADMINSRV$
SID : S-1-5-21-719815819-3726368948-3917688648-1105
[snip]
```

> The most important is the Name and SID

### Enumerate Domain Administrators

```
Get-ADGroupMember -Identity 'Domain Admins'

distinguishedName :
CN=Administrator,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local
name : Administrator
objectClass : user
objectGUID : d954e824-f549-47c2-9809-646c218cef36
SamAccountName : Administrator
SID : S-1-5-21-719815819-3726368948-3917688648-500
distinguishedName : CN=svc admin,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local
name : svc admin
objectClass : user
objectGUID : 244f9c84-7e33-4ed6-aca1-3328d0802db0
SamAccountName : svcadmin
SID : S-1-5-21-719815819-3726368948-3917688648-1118
```

### Enumerate the Enterprise Administrators

```
Get-ADGroupMember -Identity 'Enterprise Admins' -Server moneycorp.local

distinguishedName : CN=Administrator,CN=Users,DC=moneycorp,DC=local
name : Administrator
objectClass : user
objectGUID : bff03156-2c42-4e55-a21c-07eb868cd5f8
SamAccountName : Administrator
SID : S-1-5-21-335606122-960912869-3279953914-500	
```

***

## BloodHound

For BloodHound, we will try with both the Legacy version and Community Edition.

&#x20;      BloodHound Legacy (To be done only after getting admin privileges)

&#x20;      BloodHound uses neo4j graph database, so that needs to be set up first.

> Note: Exit BloodHound once you have stopped using it as it uses good amount of RAM. You may also like to stop the neo4j service if you are not using BloodHound.

### BloodHound Instalation

We need to install the neo4j service. Unzip the archive C:\AD\Tools\neo4j-community-4.1.1-windows.zip

> Install and start the neo4j service as follows, into:&#x20;
>
> ```
> cd C:\AD\Tools\neo4j-community-4.4.5-windows\neo4j-community-4.4.5\bin
> ```

> For it, we need a admin user to continue with the installation

```
neo4j.bat install-service
neo4j.bat start
```

<figure><img src="../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

Once the service is started, browse to [http://localhost:7474](http://localhost:7474/)

<figure><img src="../../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

Enter the username: neo4j and password: neo4j. You need to enter a new password. Let's use BloodHound as the new password.

Now, open BloodHound from C:\AD\Tools\BloodHound-win32-x64\BloodHound-win32-x64 and provide the following details:

> bolt://localhost:7687
>
> Username: neo4j Password: BloodHound

<figure><img src="https://www.enterprisesecurity.io/JSP_JS_CSS/assets/images/labmanual/obj1(3).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="https://www.enterprisesecurity.io/JSP_JS_CSS/assets/images/labmanual/obj1(4).png" alt=""><figcaption></figcaption></figure>

### BloodHound ingestores

Run BloodHound ingestores to gather data and information about the current domain. Run the following commands to run Collector:

```
C:\AD\Tools\BloodHound-master\BloodHound-master\Collectors\SharpHound.exe --collectionmethods Group,GPOLocalGroup,Session,Trusts,ACL,Container,ObjectProps,SPNTargets --excludedcs
```

Once all the data is uploaded to BloodHound, search for shortest path to Domain Admins in dollarcorp domain. (press Ctrl to toggle labels).

### BloodHound CE of Web UI

We need to run a compatible Sharphound collector for BloodHound CE. Remember that you have Read-only access to the shared BloodHound CE UI in the lab. There is no need or way to upload the data collected to the shared instance.

As BloodHound CE consumes high amounts of RAM, in the lab, you have Read-only access to a shared BloodHound CE - [https://crtpbloodhound-altsecdashboard.msappproxy.net/](https://crtpbloodhound-altsecdashboard.msappproxy.net/)

<figure><img src="https://www.enterprisesecurity.io/JSP_JS_CSS/assets/images/labmanual/obj1(5).png" alt=""><figcaption></figcaption></figure>

Provide the following credentials to the Microsoft login page:

> Username: crtpreader@altsecdashboard.onmicrosoft.com
>
> Password: Are@d0nlyUsertO200kAtSecurityDashb0ardf0rL0g!n!A@ga!n

This would bring you to the BloodHound CE login page. Provide the same set of credentials as above to the BloodHound login page and you will be able to access the UI.

<figure><img src="https://www.enterprisesecurity.io/JSP_JS_CSS/assets/images/labmanual/obj1(6).png" alt=""><figcaption></figcaption></figure>

Always double-check the credentials in the lab portal - [https://adlab.enterprisesecurity.io](https://www.enterprisesecurity.io/)

This instance of BloodHound CE already has the database populated. Feel free to play with the data! To solve the task in the Learning Objective, proceed as follows.

> In the Web UI, click on Cypher -> Click on the Folder Icon -> Pre-Built Searches -> Active Directory -> (Scroll down) -> Shortest paths to Domain Admins

<figure><img src="https://www.enterprisesecurity.io/JSP_JS_CSS/assets/images/labmanual/obj1(7).png" alt=""><figcaption></figcaption></figure>

***

File share where studentx has Write permissions


We will use PowerHuntShares to search for file shares where studentx has Write permissions.

We will not scan the domain controller for Writable shares for a better OPSEC.

### Invisi-Shell & PowerHuntShares exec

Run the following commands from a <mark style="background-color:yellow;">PowerShell session started using</mark> [<mark style="background-color:yellow;">Invisi-Shell</mark>](learning-objetive-1.md#using-the-active-directory-module-admodule):

> After this, we need save into a file txt in C:\AD\Tools, all Domain Computer, extract its using:
>
> ```
> Get-DomainComputer | select -ExpandProperty dnshostname
> ```

<pre><code>PS C:\AD\Tools> notepad servers.txt
<strong>## Paste the servers
</strong><strong>cat C:\AD\Tools\servers.txt
</strong></code></pre>

```
Import-Module C:\AD\Tools\PowerHuntShares.psm1
Invoke-HuntSMBShares -NoPing -OutputDirectory C:\AD\Tools\ -HostList C:\AD\Tools\servers.txt
```

<figure><img src="../../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

> It generate us a .htlm in the same folder

<figure><img src="../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

> You need to copy the summary report to your host machine because the report needs interent access, which is not available on the student VM.

The Summary Report page shows, well, the summary.

<figure><img src="https://www.enterprisesecurity.io/JSP_JS_CSS/assets/images/labmanual/obj1(10).png" alt=""><figcaption></figcaption></figure>

The Critical and High findings will be for dcorp-adminsrv as studentx has admin privileges there. Another interesting observation is in the Medium findings that shows that there is a directory named 'AI' on dcorp-ci where 'BUILTIN\Users' has 'WriteData/Addfile' permissions.

<figure><img src="https://www.enterprisesecurity.io/JSP_JS_CSS/assets/images/labmanual/obj1(11).png" alt=""><figcaption></figcaption></figure>

> Go to ShareGraph -> search dcorp-ci -> Right click on dcorp-ci node -> Click expand. Tt turns out that 'Everyone' has privileges on the 'AI' folder.

<figure><img src="https://www.enterprisesecurity.io/JSP_JS_CSS/assets/images/labmanual/obj1(12).png" alt=""><figcaption></figcaption></figure>
