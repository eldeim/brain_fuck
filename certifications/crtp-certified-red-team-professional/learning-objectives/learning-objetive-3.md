# Learning Objetive 3

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## Invisible Shell

```
cd \AD\Tools
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```

```
. C:\AD\Tools\PowerView.ps1
```

## Enumerate OUs

<pre><code>Get-DomainOU

description            : Default container for domain controllers
systemflags            : -1946157056
iscriticalsystemobject : True
gplink                 : [LDAP://CN={6AC1786C-016F-11D2-945F-00C04fB984F9},CN=Policies,CN=System,DC=dollarcorp,DC=moneycorp,DC=local;0]
whenchanged            : 11/12/2022 5:59:00 AM
objectclass            : {top, organizationalUnit}
showinadvancedviewonly : False
usnchanged             : 7921
dscorepropagationdata  : {11/15/2022 3:49:24 AM, 11/12/2022 5:59:41 AM, 1/1/1601 12:04:16 AM}
<a data-footnote-ref href="#user-content-fn-1">name                   : Domain Controllers</a>
distinguishedname      : OU=Domain Controllers,DC=dollarcorp,DC=moneycorp,DC=local
ou                     : Domain Controllers
</code></pre>

### Names of the OUs

```
Get-DomainOU | select -ExpandProperty name

Domain Controllers
StudentMachines
Applocked
Servers
DevOps
```

### List all the computers in the DevOps OU

```
(Get-DomainOU -Identity DevOps).distinguishedname | %{Get-DomainComputer -SearchBase $_} | select name

name
----
DCORP-CI
[snip]
```

***

## Enumerate GPOs

<pre><code>Get-DomainGPO

flags                    : 0
systemflags              : -1946157056
<a data-footnote-ref href="#user-content-fn-1">displayname              : Default Domain Policy</a>

[snip]

flags                    : 0
<a data-footnote-ref href="#user-content-fn-1">displayname              : DevOps Policy</a>
gpcmachineextensionnames : [{35378EAC-683F-11D2-A89A-00C04FBBCFA2}{D02B1F72-3407-48AE-BA88-E8213C6761F1}][{827D319E-6EAC-11D2-A4EA-00C04F79F83A}{803E14A0-B4FB-11D0-A0D0-00A0C90F574B}]
whenchanged              : 12/19/2024 12:00:15 PM
versionnumber            : 3
<a data-footnote-ref href="#user-content-fn-1">name                     : {0BF8D01C-1F62-4BDC-958C-57140B67D147}</a>
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
</code></pre>

### GPO applied on the DevOps OU

We need the name of the policy from the gplink attribute from the OU

<pre><code>(Get-DomainOU -Identity DevOps).gplink

[LDAP://cn={<a data-footnote-ref href="#user-content-fn-1">0BF8D01C-1F62-4BDC-958C-57140B67D147</a>},cn=policies,cn=system,DC=dollarcorp,DC=moneycorp,DC=local;0]
</code></pre>

Now, copy the highlighted string from above (no square brackets, no semicolon and nothing after semicolon) and use the it below:

```
Get-DomainGPO -Identity '{0BF8D01C-1F62-4BDC-958C-57140B67D147}'
```

<pre><code>PS C:\AD\Tools>Get-DomainGPO -Identity '{0BF8D01C-1F62-4BDC-958C-57140B67D147}'


flags                    : 0
<a data-footnote-ref href="#user-content-fn-1">displayname              : DevOps Policy</a>
gpcmachineextensionnames : [{35378EAC-683F-11D2-A89A-00C04FBBCFA2}{D02B1F72-3407-48AE-BA88-E8213C6761F1}][{827D319E-6EAC-11D2-A4EA-00C04F79F83A}{803E14A0-B4FB-11D0-A0D0-00A0C90F574B}]
whenchanged              : 12/19/2024 12:00:15 PM
versionnumber            : 3
<a data-footnote-ref href="#user-content-fn-1">name                     : {0BF8D01C-1F62-4BDC-958C-57140B67D147}</a>
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
</code></pre>

<mark style="background-color:yellow;">It is possible to hack both the commands together in a single command</mark> (profiting from the static length for GUIDs)

```
Get-DomainGPO -Identity (Get-DomainOU -Identity DevOps).gplink.substring(11,(Get-DomainOU -Identity DevOps).gplink.length-72)

flags                    : 0
displayname              : DevOps Policy
gpcmachineextensionnames : [{35378EAC-683F-11D2-A89A-00C04FBBCFA2}{D02B1F72-3407-48AE-BA88-E8213C6761F1}][{827D319E-6EAC-11D2-A4EA-00C04F79F83A}{803E14A0-B4FB-11D0-A0D0-00A0C90F574B}]
[snip]
```

***

## Enumerate ACLs

To enumerate the ACLs for the Applocked and DevOps GPO, let's use the BloodHound CE UI.

Search for Applocker in the UI -> Click on the node -> Click on Inboud Object Control

<figure><img src="https://www.enterprisesecurity.io/JSP_JS_CSS/assets/images/labmanual/obj3(1).png" alt=""><figcaption></figcaption></figure>

It turns out that the RDPUsers group has GenericAll over the policy.

Similary, search for DevOps and look at its 'Inbound Object Control':

<figure><img src="https://www.enterprisesecurity.io/JSP_JS_CSS/assets/images/labmanual/obj3(2).png" alt=""><figcaption></figcaption></figure>

A user named 'devopsadmin' has 'WriteDACL' on DevOps Policy.

***

* Display name of the GPO applied on StudentMachines OU

```
Get-DomainGPO -Identity (Get-DomainOU -Identity StudentMachines).gplink.substring(11,(Get-DomainOU -Identity DevOps).gplink.length-72)

flags                    : 0
displayname              : Students
gpcmachineextensionnames : [{35378EAC-683F-11D2-A89A-00C04FBBCFA2}{D02B1F72-3407-48AE-BA88-E8213C6761F1}][{827D319E-6EAC-11D2-A4EA-00C04F79F83A}{803E14A0-B4FB-11D0-A0D0-00A0C90F574B}]
whenchanged              : 7/30/2024 1:30:35 PM
versionnumber            : 9
name                     : {7478F170-6A0C-490C-B355-9E4618BC785D}
cn                       : {7478F170-6A0C-490C-B355-9E4618BC785D}
usnchanged               : 247100
dscorepropagationdata    : {12/5/2024 12:47:28 PM, 1/1/1601 12:00:01 AM}
objectguid               : 0076f619-ffef-4488-bfdb-1fc028c5cb14
gpcfilesyspath           : \\dollarcorp.moneycorp.local\SysVol\dollarcorp.moneycorp.local\Policies\{7478F170-6A0C-490C-B355-9E4618BC785D}
distinguishedname        : CN={7478F170-6A0C-490C-B355-9E4618BC785D},CN=Policies,CN=System,DC=dollarcorp,DC=moneycorp,DC=local
whencreated              : 11/15/2022 5:46:19 AM
showinadvancedviewonly   : True
usncreated               : 45927
gpcfunctionalityversion  : 2
instancetype             : 4
objectclass              : {top, container, groupPolicyContainer}
objectcategory           : CN=Group-Policy-Container,CN=Schema,CN=Configuration,DC=moneycorp,DC=local
```

[^1]: 
