# 🌇 CRTP-Certified Red Team Professional

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://www.alteredsecurity.com/adlab" %}

***

## 23 Learning Objectives, 59 Tasks, >120 Hours of Torture

**I. Active Directory Enumeration**

* Use scripts, built-in tools and Active Directory module to enumerate the target domain.
* Understand and practice how useful information like users, groups, group memberships, computers, user properties etc. from the domain controller is available to even a normal user.
* Understand and enumerate intra-forest and inter-forest trusts. Practice how to extract information from the trusts.
* Enumerate Group policies.
* Enumerate ACLs and learn to find out interesting rights on ACLs in the target domain to carry out attacks.
* Learn to use BloodHound and understand its applications in a red team operation.

**II. Offensive PowerShell Tradecraft**

* Learn how PowerShell tools can still be used for enumeration.
* Learn to modify existing tools to bypass Windows Defender.
* Bypass PowerShell security controls and enhanced logging like System Wide Transcription, Anti Malware Scan Interface (AMSI), Script Blok Logging and Constrained Language Mode (CLM)

**III. Offensive .NET Tradecraft**

* Learn how to modify and use .NET tools to bypass Windows Defender and Microsoft Defender for Endpoint (MDE).
* Learn to use .NET Loaders that can run assemblies in-memory.

**IV. Local Privilege Escalation**

* Learn and practice different local privilege escalation techniques on a Windows machine.
* Hunt for local admin privileges on machines in the target domain using multiple methods.
* Abuse enterprise applications to execute complex attack paths that involve bypassing antivirus and pivoting to different machines.

**V. Domain Privilege Escalation**

* Learn to find credentials and sessions of high privileges domain accounts like Domain Administrators, extracting their credentials and then using credential replay attacks to escalate privileges, all of this with just using built-in protocols for pivoting.
* Learn to extract credentials from a restricted environment where application whitelisting is enforced. Abuse derivative local admin privileges and pivot to other machines to escalate privileges to domain level.
* Understand the classic Kerberoast and its variants to escalate privileges.
* Enumerate the domain for objects with unconstrained delegation and abuse it to escalate privileges.
* Find domain objects with constrained delegation enabled. Understand and execute the attacks against such objects to escalate privileges to a single service on a machine and to the domain administrator using alternate tickets.
* Learn how to abuse privileges of Protected Groups to escalate privileges

**VI. Domain Persistence and Dominance**

* Abuse Kerberos functionality to persist with DA privileges. Forge tickets to execute attacks like Golden ticket, Silver ticket and Diamond ticket to persist.
* Subvert the authentication on the domain level with Skeleton key and custom SSP.
* Abuse the DC safe mode Administrator for persistence.
* Abuse the protection mechanism like AdminSDHolder for persistence.
* Abuse minimal rights required for attacks like DCSync by modifying ACLs of domain objects.
* Learn to modify the host security descriptors of the domain controller to persist and execute commands without needing DA privileges.

**VII. Cross Trust Attacks**

* Learn to elevate privileges from Domain Admin of a child domain to Enterprise Admin on the forest root by abusing Trust keys and krbtgt account.
* Execute intra-forest trust attacks to access resources across forest.
* Abuse SQL Server database links to achieve code execution across forest by just using the databases.

**VIII. Abusing AD CS**&#x20;

* Learn about Active Directory Certificate Services and execute some of the most popular attacks.
* Execute attacks across Domain trusts to escalate privileges to Enterprise Admins.

**IX. Defenses and bypass – MDE EDR**

* Learn about Microsoft’s EDR – Microsoft Defender for Endpoint.
* Understand the telemetry and components used by MDE for detection.
* Execute an entire chain of attacks across forest trust without triggering any alert by MDE.
* Use Security 365 dashboard to verify MDE bypass.

**X. Defenses and bypass – MDI**

* Learn about Microsoft Identity Protection (MDI).
* Understand how MDI relies on anomaly to spot an attack.
* Bypass various MDI detections throughout the course.

**XI. Defenses and bypass – Architecture and Work Culture Changes**

* Learn briefly about architecture and work culture changes required in an organization to avoid the discussed attacks. We discuss Temporal group membership, ACL Auditing, LAPS, SID Filtering, Selective Authentication, credential guard, device guard, Protected Users Group, PAW, Tiered Administration and ESAE or Red Forest

**XII. Defenses – Monitoring**

* Learn about useful events logged when the discussed attacks are executed.

**XIII. Defenses and Bypass – Deception**

* Understand how Deception can be effective deployed as a defense mechanism in AD.
* Deploy decoy user objects, which have interesting properties set, which have ACL rights over other users and have high privilege access in the domain along with available protections.
* Deploy computer objects and Group objects to deceive an adversary.
* Learn how adversaries can identify decoy objects and how defenders can avoid the detection.

***

### Tools

{% embed url="https://monikaalteredsecurity-my.sharepoint.com/personal/coursevideosone_monikaalteredsecurity_onmicrosoft_com/_layouts/15/onedrive.aspx?id=%2Fpersonal%2Fcoursevideosone%5Fmonikaalteredsecurity%5Fonmicrosoft%5Fcom%2FDocuments%2FAccess%2DLab%2DMaterial%2FCRTP%2FTools%2DBootcamp%2FTools%2Ezip&parent=%2Fpersonal%2Fcoursevideosone%5Fmonikaalteredsecurity%5Fonmicrosoft%5Fcom%2FDocuments%2FAccess%2DLab%2DMaterial%2FCRTP%2FTools%2DBootcamp&ga=1" %}

> ```
> CRTPRocks!
>
> Use 7Zip to Extract Tools.zip
> ```
