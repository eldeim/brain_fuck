# 📄 Inform

## Writing a Good Report

he essential elements of a good bug report are (the element order can vary):

<table><thead><tr><th width="251"></th><th></th></tr></thead><tbody><tr><td><code>Vulnerability Title</code></td><td>Including vulnerability type, affected domain/parameter/endpoint, impact etc.</td></tr><tr><td><code>CWE &#x26; CVSS score</code></td><td>For communicating the characteristics and severity of the vulnerability.</td></tr><tr><td><code>Vulnerability Description</code></td><td>Better understanding of the vulnerability cause.</td></tr><tr><td><code>Proof of Concept (POC)</code></td><td>Steps to reproduce exploiting the identified vulnerability clearly and concisely.</td></tr><tr><td><code>Impact</code></td><td>Elaborate more on what an attacker can achieve by fully exploiting the vulnerability. Business impact and maximum damage should be included in the impact statement.</td></tr><tr><td><code>Remediation</code></td><td>Optional in bug bounty programs, but good to have.</td></tr></tbody></table>

Readable and well-formatted bug reports can drastically minimize both vulnerability reproduction time and time to triage.

### Why CWE & CVSS?

MITRE describes [Common Weaknesses Enumeration (CWE)](https://cwe.mitre.org/) as a community-developed list of software and hardware weakness types. It serves as a common language, a measuring stick for security tools, and as a baseline for weakness identification, mitigation, and prevention efforts. In the case of a vulnerability chain, choose a CWE related to the initial vulnerability.

When it comes to communicating the severity of an identified vulnerability, then [Common Vulnerability Scoring System (CVSS)](https://www.first.org/cvss/) should be used, as it is a published standard used by organizations worldwide.

## Using CVSS Calculator

Let us now see how we can use the [CVSS v3.1 Calculator](https://www.first.org/cvss/calculator/3.1) identify the severity of an identified vulnerability.

We will focus on the `Base Score` area only.

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

### Attack Vector

Shows how the vulnerability can be exploited.

Network (N): Attackers can only exploit this vulnerability through the network layer (remotely exploitable).

Adjacent (A): Attackers can exploit this vulnerability only if they reside in the same physical or logical network (secure VPN included).

Local (L): Attackers can exploit this vulnerability only by accessing the target system locally (e.g., keyboard, terminal, etc.) or remotely (e.g., SSH) or through user interaction.

Physical (P): Attackers can exploit this vulnerability through physical interaction/manipulation.

### Attack Complexity

Depicts the conditions beyond the attackers' control and must be present to exploit the vulnerability successfully.

Low (L): No special preparations should take place to exploit the vulnerability successfully. The attackers can exploit the vulnerability repeatedly without any issue.

High (H): Special preparations and information gathering should take place to exploit the vulnerability successfully.

### Privileges Required

Show the level of privileges the attacker must have to exploit the vulnerability successfully.

None (N): No special access related to settings or files is required to exploit the vulnerability successfully. The vulnerability can be exploited from an unauthorized perspective.

Low (L): Attackers should possess standard user privileges to exploit the vulnerability successfully. The exploitation in this case usually affects files and settings owned by a user or non-sensitive assets.

High (H): Attackers should possess admin-level privileges to exploit the vulnerability successfully. The exploitation in this case usually affects the entire vulnerable system.

### User Interaction

Shows if attackers can successfully exploit the vulnerability on their own or user interaction is required.

None (N): Attackers can successfully exploit the vulnerability independently.

Required (R): A user should take some action before the attackers can successfully exploit the vulnerability.

### Scope

Shows if successful exploitation of the vulnerability can affect components other than the affected one.

Unchanged (U): Successful exploitation of the vulnerability affects the vulnerable component or affects resources managed by the same security authority.

Changed (C): Successful exploitation of the vulnerability can affect components other than the affected one or resources beyond the scope of the affected component's security authority.

### Confidentiality

Shows how much the vulnerable component's confidentiality is affected upon successfully exploiting the vulnerability. Confidentiality limits information access and disclosure to authorized users only and prevents unauthorized users from accessing information.

None (N): The confidentiality of the vulnerable component does not get impacted.

Low (L): The vulnerable component will experience some loss of confidentiality upon successful exploitation of the vulnerability. In this case, the attackers do not have control over what information is obtained.

High (H): The vulnerable component will experience total (or serious) loss of confidentiality upon successfully exploiting the vulnerability. In this case, the attackers have total (or some) control over what information is obtained.

### Integrity

Shows how much the vulnerable component's integrity is affected upon successfully exploiting the vulnerability. Integrity refers to the trustworthiness and veracity of information.

None (N): The integrity of the vulnerable component does not get impacted.

Low (L): Attackers can modify data in a limited manner on the vulnerable component upon successfully exploiting the vulnerability. Attackers do not have control over the consequence of a modification, and the vulnerable component does not get seriously affected in this case.

High (H): Attackers can modify all or critical data on the vulnerable component upon successfully exploiting the vulnerability. Attackers have control over the consequence of a modification, and the vulnerable component will experience a total loss of integrity.

### Availability

Shows how much the vulnerable component's availability is affected upon successfully exploiting the vulnerability. Availability refers to the accessibility of information resources in terms of network bandwidth, disk space, processor cycles, etc.

None (N): The availability of the vulnerable component does not get impacted.

Low (L): The vulnerable component will experience some loss of availability upon successfully exploiting the vulnerability. The attacker does not have complete control over the vulnerable component's availability and cannot deny the service to users, and performance is just reduced.

High (H): The vulnerable component will experience total (or severe) availability loss upon successfully exploiting the vulnerability. The attacker has complete (or significant) control over the vulnerable component's availability and can deny the service to users. Performance is significantly reduced.

***

## Examples

Find below some examples of using CVSS 3.1 to communicate the severity of vulnerabilities.

<table><thead><tr><th width="239"></th><th></th></tr></thead><tbody><tr><td><code>Title:</code></td><td>Cisco ASA Software IKEv1 and IKEv2 Buffer Overflow Vulnerability (CVE-2016-1287)</td></tr><tr><td><code>CVSS 3.1 Score:</code></td><td>9.8 (Critical)</td></tr><tr><td><code>Attack Vector:</code></td><td>Network - The Cisco ASA device was exposed to the internet since it was used to facilitate connections to the internal network through VPN.</td></tr><tr><td><code>Attack Complexity:</code></td><td>Low - All the attacker has to do is execute the available exploit against the device</td></tr><tr><td><code>Privileges Required:</code></td><td>None - The attack could be executed from an unauthenticated/unauthorized perspective</td></tr><tr><td><code>User Interaction:</code></td><td>None - No user interaction is required</td></tr><tr><td><code>Scope:</code></td><td>Unchanged - Although you can use the exploited device as a pivot, you cannot affect other components by exploiting the buffer overflow vulnerability.</td></tr><tr><td><code>Confidentiality:</code></td><td>High - Successful exploitation of the vulnerability results in unrestricted access in the form of a reverse shell. Attackers have total control over what information is obtained.</td></tr><tr><td><code>Integrity:</code></td><td>High - Successful exploitation of the vulnerability results in unrestricted access in the form of a reverse shell. Attackers can modify all or critical data on the vulnerable component.</td></tr><tr><td><code>Availability:</code></td><td>High - Successful exploitation of the vulnerability results in unrestricted access in the form of a reverse shell. Attackers can deny the service to users by powering the device off</td></tr></tbody></table>

***

<table><thead><tr><th width="239"></th><th></th></tr></thead><tbody><tr><td><code>Title:</code></td><td>Stored XSS in an admin panel (Malicious Admin -> Admin)</td></tr><tr><td><code>CVSS 3.1 Score:</code></td><td>5.5 (Medium)</td></tr><tr><td><code>Attack Vector:</code></td><td>Network - The attack can be mounted over the internet.</td></tr><tr><td><code>Attack Complexity:</code></td><td>Low - All the attacker (malicious admin) has to do is specify the XSS payload that is eventually stored in the database.</td></tr><tr><td><code>Privileges Required:</code></td><td>High - Only someone with admin-level privileges can access the admin panel.</td></tr><tr><td><code>User Interaction:</code></td><td>None - Other admins will be affected simply by browsing a specific (but regularly visited) page within the admin panel.</td></tr><tr><td><code>Scope:</code></td><td>Changed - Since the vulnerable component is the webserver and the impacted component is the browser</td></tr><tr><td><code>Confidentiality:</code></td><td>Low - Access to DOM was possible</td></tr><tr><td><code>Integrity:</code></td><td>Low - Through XSS, we can slightly affect the integrity of an application</td></tr><tr><td><code>Availability:</code></td><td>None - We cannot deny the service through XSS</td></tr></tbody></table>
