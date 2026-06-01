# Class One - Notes

<figure><img src="../../../../.gitbook/assets/image (177).png" alt="" width="375"><figcaption></figcaption></figure>

## Active Directory look like

<figure><img src="../../../../.gitbook/assets/image (178).png" alt="" width="477"><figcaption></figcaption></figure>

### Structure

<figure><img src="../../../../.gitbook/assets/image (179).png" alt="" width="563"><figcaption></figcaption></figure>

### PowerShell Scripts and Modules

<figure><img src="../../../../.gitbook/assets/image (180).png" alt="" width="563"><figcaption></figcaption></figure>

### PowerShell Script Excecution

<figure><img src="../../../../.gitbook/assets/image (181).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (182).png" alt="" width="563"><figcaption></figcaption></figure>

### Bypassing PowerShell Security

<figure><img src="../../../../.gitbook/assets/image (183).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (184).png" alt="" width="563"><figcaption></figcaption></figure>

Offensive .NET - Tradecraft - AV bypass – Source\
Code Obfuscation
----------------

<figure><img src="../../../../.gitbook/assets/image (186).png" alt="" width="563"><figcaption></figcaption></figure>

### Offensive .NET - Tradecraft - Payload Delivery

<figure><img src="../../../../.gitbook/assets/image (187).png" alt="" width="563"><figcaption></figcaption></figure>

## Attack Methodology

<figure><img src="../../../../.gitbook/assets/image (185).png" alt="" width="563"><figcaption></figcaption></figure>

***

## Domain Enumeration

{% file src="../../../../.gitbook/assets/Attacking_and_Defending_ActiveDirectory - SlideNotes.pdf" %}

### Import-Module Microsoft.ActiveDirector

```
Import-Module Microsoft.ActiveDirectory.Management.dll
Import-Module ActiveDirectory.psd1
. C:\AD\Tools\PowerView.ps1
```

<figure><img src="../../../../.gitbook/assets/image (188).png" alt="" width="488"><figcaption></figcaption></figure>

```
PS C:\Users\student113> Get-Domain

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

```
PS C:\Users\student113> Get-Domain -Domain moneycorp.local

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

```

<figure><img src="../../../../.gitbook/assets/image (189).png" alt="" width="476"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (190).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (191).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (192).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (193).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (194).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (195).png" alt="" width="563"><figcaption></figcaption></figure>

### Find Shares

<figure><img src="../../../../.gitbook/assets/image (196).png" alt="" width="400"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (197).png" alt="" width="563"><figcaption></figcaption></figure>

### BloodHound

<figure><img src="../../../../.gitbook/assets/image (198).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (199).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (200).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (201).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (202).png" alt="" width="563"><figcaption></figcaption></figure>

***

## Domain Enumeration - ACLs

<figure><img src="../../../../.gitbook/assets/image (208).png" alt="" width="509"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (209).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (210).png" alt="" width="563"><figcaption></figcaption></figure>

***

## Domain Enumeration - Group Policy (GPO & OU)

<figure><img src="../../../../.gitbook/assets/image (1400).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (128).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (129).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (130).png" alt="" width="563"><figcaption></figcaption></figure>

***

## Domain Trust Enumeration

<figure><img src="../../../../.gitbook/assets/image (173).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (174).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (175).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (171).png" alt="" width="563"><figcaption></figcaption></figure>

### Forest Enumeration

<figure><img src="../../../../.gitbook/assets/image (172).png" alt=""><figcaption></figcaption></figure>

***

## Domain Enumeration - User Hunting

<figure><img src="../../../../.gitbook/assets/image (132).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (133).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (134).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (135).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (136).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (137).png" alt=""><figcaption></figcaption></figure>
