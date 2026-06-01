# Learning Objetive 4

<figure><img src="../../../../.gitbook/assets/image (139).png" alt=""><figcaption></figcaption></figure>

* Trust Direction for the trust between dollarcorp.moneycorp.local and eurocorp.local

## Enumerate all domains in the current forest

> Note: Remenber use a silent powershell

```
PS C:\AD\Tools> Get-ForestDomain -Verbose

VERBOSE: [Get-DomainUser] filter string: (&(samAccountType=805306368)(|(samAccountName=krbtgt))
VERBOSE: [Get-DomainSearcher] search base: LDAP://DCORP-DC.DOLLARCORP.MONEYCORP.LOCAL/DC=moneycorp,DC=local
VERBOSE: [Invoke-LDAPQuery] filter string: (&(samAccountType=805306368)(|(samAccountName=krbtgt)))


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

Forest                  : moneycorp.local
DomainControllers       : {mcorp-dc.moneycorp.local}
Children                : {dollarcorp.moneycorp.local}
DomainMode              : Unknown
DomainModeLevel         : 7
Parent                  :
PdcRoleOwner            : mcorp-dc.moneycorp.local
RidRoleOwner            : mcorp-dc.moneycorp.local
InfrastructureRoleOwner : mcorp-dc.moneycorp.local
Name                    : moneycorp.local

Forest                  : moneycorp.local
DomainControllers       : {us-dc.us.dollarcorp.moneycorp.local}
Children                : {}
DomainMode              : Unknown
DomainModeLevel         : 7
Parent                  : dollarcorp.moneycorp.local
PdcRoleOwner            : us-dc.us.dollarcorp.moneycorp.local
RidRoleOwner            : us-dc.us.dollarcorp.moneycorp.local
InfrastructureRoleOwner : us-dc.us.dollarcorp.moneycorp.local
Name                    : us.dollarcorp.moneycorp.local
```

## Enumerate all trust of "dollarcorp" domain

```
PS C:\AD\Tools> Get-DomainTrust

SourceName      : dollarcorp.moneycorp.local
TargetName      : moneycorp.local
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST
TrustDirection  : Bidirectional
WhenCreated     : 11/12/2022 5:59:01 AM
WhenChanged     : 1/25/2026 4:05:24 AM

SourceName      : dollarcorp.moneycorp.local
TargetName      : us.dollarcorp.moneycorp.local
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST
TrustDirection  : Bidirectional
WhenCreated     : 11/12/2022 6:22:51 AM
WhenChanged     : 1/25/2026 4:16:41 AM

SourceName      : dollarcorp.moneycorp.local
TargetName      : eurocorp.local
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FILTER_SIDS
TrustDirection  : Bidirectional
WhenCreated     : 11/12/2022 8:15:23 AM
WhenChanged     : 1/25/2026 4:16:41 AM
```

### List only the external trusts in the "moneycorp.local" forest

```
Get-ForestDomain | %{Get-DomainTrust -Domain $_.Name} | ?{$_.TrustAttributes -eq "FILTER_SIDS"}


SourceName      : dollarcorp.moneycorp.local
TargetName      : eurocorp.local
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FILTER_SIDS
TrustDirection  : Bidirectional
WhenCreated     : 11/12/2022 8:15:23 AM
WhenChanged     : 1/25/2026 4:16:41 AM
```

> **External Trust** con filtrado de SID habilitado

### Enumerate external trusts of the "dollarcorp" domain

```
Get-DomainTrust | ?{$_.TrustAttributes -eq "FILTER_SIDS"}

SourceName      : dollarcorp.moneycorp.local
TargetName      : eurocorp.local
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FILTER_SIDS
TrustDirection  : Bidirectional
WhenCreated     : 11/12/2022 8:15:23 AM
WhenChanged     : 1/25/2026 4:16:41 AM
```

Since the above is a Bi-Directional trust, we can extract information from the eurocorp.local forest.

> We either need bi-directional trust or one-way trust from eurocorp.local to dollarcorp to be able to use the below command

Let's go for the last task and enumerate trusts for eurocorp.local forest:

### Extract information from the eurocorp.local forest

```
S C:\AD\Tools> Get-ForestDomain -Forest eurocorp.local | %{Get-DomainTrust -Domain $_.Name}

SourceName      : eurocorp.local
TargetName      : eu.eurocorp.local
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST
TrustDirection  : Bidirectional
WhenCreated     : 11/12/2022 5:49:08 AM
WhenChanged     : 3/3/2023 10:15:16 AM

SourceName      : eurocorp.local
TargetName      : dollarcorp.moneycorp.local
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FILTER_SIDS
TrustDirection  : Bidirectional
WhenCreated     : 11/12/2022 8:15:23 AM
WhenChanged     : 2/24/2023 9:10:52 AM

Exception calling "FindAll" with "0" argument(s): "A referral was returned from the server.
```

> Notice the error above. It occurred because PowerView attempted to list trusts even for eu.eurocorp.local. Because external trust is non-transitive it was not possible!

***

## Using Active Directory module

### AD Module in a PowerShell - Invisi-Shell

> Import the AD Module in a PowerShell session started using Invisi-Shell:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll
Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
```

### Enumerate all the domains

```
(Get-ADForest).Domains

dollarcorp.moneycorp.local
moneycorp.local
us.dollarcorp.moneycorp.local
```

### Enumerate all the Trusts in the current domain

```
Get-ADTrust -Filter *

Direction : BiDirectional
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
```

### Enumerate all the trusts in the moneycorp.local forest

```
Get-ADForest | %{Get-ADTrust -Filter *} 

Direction : BiDirectional
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
```

### Enumerate external trusts in moneycorp.local domain

```
(Get-ADForest).Domains | %{Get-ADTrust -Filter '(intraForest -ne $True) -and (ForestTransitive -ne $True)' -Server $_} 

Direction : BiDirectional
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
```
