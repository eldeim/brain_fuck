# Learning Objetive 2

<figure><img src="../../../../.gitbook/assets/image (1401).png" alt=""><figcaption></figcaption></figure>

***

## Enumerate ACLs for the Domain Admins Group

> Remember to conitnue using the PowerShell session started using Invisi-Shell
>
> ```
> cd \AD\Tools
> C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
> . C:\AD\Tools\PowerView.ps1
> ```

```
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs -Verbose

VERBOSE: [Get-DomainSearcher] search base: LDAP://DCORP-DC.DOLLARCORP.MONEYCORP.LOCAL/DC=DOLLARCORP,DC=MONEYCORP,DC=LOCAL
VERBOSE: [Get-DomainUser] filter string: (&(samAccountType=805306368)(|(samAccountName=krbtgt))
VERBOSE: [Get-DomainSearcher] search base: LDAP://DCORP-DC.DOLLARCORP.MONEYCORP.LOCAL/DC=moneycorp,DC=local
[snip]

AceQualifier           : AccessAllowed
ObjectDN               : CN=Domain Admins,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local
ActiveDirectoryRights  : ReadProperty
ObjectAceType          : User-Account-Restrictions
ObjectSID              : S-1-5-21-719815819-3726368948-3917688648-512
InheritanceFlags       : None
BinaryLength           : 60
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent, InheritedObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-32-554
AccessMask             : 16
AuditFlags             : None
IsInherited            : False
AceFlags               : None
InheritedObjectAceType : inetOrgPerson
OpaqueLength           : 0

[snip]
```

## Excesive Permissions on us account

Finally, <mark style="background-color:yellow;">to check for modify rights/permissions for the studentx</mark>, we can use Find-InterestingDomainACL from PowerView:

```
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "student113"}
```

> Nothing interesting!

## Member of the RDPUsers group

<mark style="background-color:yellow;">Since studentx is a member of the RDPUsers group</mark>, let us check permissions for it too.

> Note that the output in your lab for the below command will be different and will depend on your lab instance:

```
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}

ObjectDN                : CN=ControlxUser,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local
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
```

## Analyze the permissions for studentx using BloodHound UI

> Note that it is easier to analyze ACLs using BloodHound as it shows interesting ACLs for the user and the groups it is a member of. Let's look at the 'Outbound Object Control' for the studentx in the BloodHound CE UI:

<figure><img src="../../../../.gitbook/assets/image (1396).png" alt=""><figcaption></figcaption></figure>

Multiple permissions stand out in the above diagram. Due to the membership of the RDPUsers group, the studentx user has following interesting permissions

* Full Control/Generic All over supportx and controlx users.
* Enrollment permissions on multiple certificate templates.
* Full Control/Generic All on the Applocked Group Policy.

<figure><img src="../../../../.gitbook/assets/image (1397).png" alt=""><figcaption></figcaption></figure>
