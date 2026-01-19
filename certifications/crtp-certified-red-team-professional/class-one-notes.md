# 📄 Class One - Notes

<figure><img src="../../.gitbook/assets/image (2).png" alt="" width="375"><figcaption></figcaption></figure>

## Active Directory look like

<figure><img src="../../.gitbook/assets/image (3).png" alt="" width="477"><figcaption></figcaption></figure>

### Structure

<figure><img src="../../.gitbook/assets/image (4).png" alt="" width="563"><figcaption></figcaption></figure>

### PowerShell Scripts and Modules

<figure><img src="../../.gitbook/assets/image (5).png" alt="" width="563"><figcaption></figcaption></figure>

### PowerShell Script Excecution

<figure><img src="../../.gitbook/assets/image (6).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (7).png" alt="" width="563"><figcaption></figcaption></figure>

### Bypassing PowerShell Security

<figure><img src="../../.gitbook/assets/image (8).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (10).png" alt="" width="563"><figcaption></figcaption></figure>

Offensive .NET - Tradecraft - AV bypass – Source\
Code Obfuscation
----------------

<figure><img src="../../.gitbook/assets/image (12).png" alt="" width="563"><figcaption></figcaption></figure>

### Offensive .NET - Tradecraft - Payload Delivery&#xD;

<figure><img src="../../.gitbook/assets/image (13).png" alt="" width="563"><figcaption></figcaption></figure>

## Attack  Methodology

<figure><img src="../../.gitbook/assets/image (11).png" alt="" width="563"><figcaption></figcaption></figure>

***

## Domain Enumeration

{% file src="../../.gitbook/assets/Attacking_and_Defending_ActiveDirectory - SlideNotes.pdf" %}

### Import-Module Microsoft.ActiveDirector

```
Import-Module Microsoft.ActiveDirectory.Management.dll
Import-Module ActiveDirectory.psd1
. C:\AD\Tools\PowerView.ps1
```

<figure><img src="../../.gitbook/assets/image (14).png" alt="" width="488"><figcaption></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image (15).png" alt="" width="476"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (16).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (17).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (18).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (19).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (20).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (21).png" alt="" width="563"><figcaption></figcaption></figure>

### Find Shares

<figure><img src="../../.gitbook/assets/image (23).png" alt="" width="400"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (24).png" alt="" width="563"><figcaption></figcaption></figure>

### BloodHound

<figure><img src="../../.gitbook/assets/image (25).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (26).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (27).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (28).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (29).png" alt="" width="563"><figcaption></figcaption></figure>

***

## Domain Enumeration - ACLs

<figure><img src="../../.gitbook/assets/image (36).png" alt="" width="509"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (37).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (38).png" alt="" width="563"><figcaption></figcaption></figure>

***

## Domain Enumeration - Group Policy

<figure><img src="../../.gitbook/assets/image (528).png" alt=""><figcaption></figcaption></figure>

