# Extracting Passwords from the Network

## Credential Hunting in Network Traffic

The table below lists several common protocols alongside their encrypted counterparts. While it is now more common to encounter the secure versions, there was a time when plaintext protocols were widely used.

| Unencrypted Protocol | Encrypted Counterpart      | Description                                                                 |
| -------------------- | -------------------------- | --------------------------------------------------------------------------- |
| `HTTP`               | `HTTPS`                    | Used for transferring web pages and resources over the internet.            |
| `FTP`                | `FTPS/SFTP`                | Used for transferring files between a client and a server.                  |
| `SNMP`               | `SNMPv3 (with encryption)` | Used for monitoring and managing network devices like routers and switches. |
| `POP3`               | `POP3S`                    | Retrieves emails from a mail server to a local client.                      |
| `IMAP`               | `IMAPS`                    | Accesses and manages email messages directly on the mail server.            |
| `SMTP`               | `SMTPS`                    | Sends email messages from client to server or between mail servers.         |
| `LDAP`               | `LDAPS`                    | Queries and modifies directory services like user credentials and roles.    |
| `RDP`                | `RDP (with TLS)`           | Provides remote desktop access to Windows systems.                          |
| `DNS (Traditional)`  | `DNS over HTTPS (DoH)`     | Resolves domain names into IP addresses.                                    |
| `SMB`                | `SMB over TLS (SMB 3.0)`   | Shares files, printers, and other resources over a network.                 |
| `VNC`                | `VNC with TLS/SSL`         | Allows graphical remote control of another computer.                        |

### Wireshark

[Wireshark](https://www.wireshark.org/) is a well-known packet analyzer that comes pre-installed in nearly all penetration testing Linux distributions. It features a powerful [filter engine](https://www.wireshark.org/docs/man-pages/wireshark-filter.html) that allows for efficient searching through both live and captured network traffic. Some basic but useful filters include:

| Wireshark filter                                  | Description                                                                                                                                                                          |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `ip.addr == 56.48.210.13`                         | Filters packets with a specific IP address                                                                                                                                           |
| `tcp.port == 80`                                  | Filters packets by port (HTTP in this case).                                                                                                                                         |
| `http`                                            | Filters for HTTP traffic.                                                                                                                                                            |
| `dns`                                             | Filters DNS traffic, which is useful to monitor domain name resolution.                                                                                                              |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0`        | Filters SYN packets (used in TCP handshakes), useful for detecting scanning or connection attempts.                                                                                  |
| `icmp`                                            | Filters ICMP packets (used for Ping), which can be useful for reconnaissance or network issues.                                                                                      |
| `http.request.method == "POST"`                   | Filters for HTTP POST requests. In the case that POST requests are sent over unencrypted HTTP, it may be the case that passwords or other sensitive information is contained within. |
| `tcp.stream eq 53`                                | Filters for a specific TCP stream. Helps track a conversation between two hosts.                                                                                                     |
| `eth.addr == 00:11:22:33:44:55`                   | Filters packets from/to a specific MAC address.                                                                                                                                      |
| `ip.src == 192.168.24.3 && ip.dst == 56.48.210.3` | Filters traffic between two specific IP addresses. Helps track communication between specific hosts.                                                                                 |

For example, in the image below we are filtering for unencrypted `HTTP` traffic.

![Network packet capture showing HTTP requests with source, destination, protocol, length, and info details.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/308/img/Net_2.png)

In Wireshark, it's possible to locate packets that contain specific bytes or strings. One way to do this is by using a display filter such as `http contains "passw"`. Alternatively, you can navigate to `Edit > Find Packet` and enter the desired search query manually. For example, you might search for packets containing the string `"passw"`:

![Network packet capture showing HTTP requests with details. Highlighted POST request includes HTML form data with username and password fields.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/308/img/Net_3.png)

It's worth familiarizing yourself with the syntax of Wireshark's filtering engine, especially if you ever need to perform network traffic analysis.

### Pcredz

[Pcredz](https://github.com/lgandx/PCredz) is a tool that can be used to extract credentials from live traffic or network packet captures. Specifically, it supports extracting the following information:

* Credit card numbers
* POP credentials
* SMTP credentials
* IMAP credentials
* SNMP community strings
* FTP credentials
* Credentials from HTTP NTLM/Basic headers, as well as HTTP Forms
* NTLMv1/v2 hashes from various traffic including DCE-RPC, SMBv1/2, LDAP, MSSQL, and HTTP
* Kerberos (AS-REQ Pre-Auth etype 23) hashes

In order to run `Pcredz`, one may either clone the repository and install all dependencies, or use the provided Docker container detailed in the [Install](https://github.com/lgandx/PCredz?tab=readme-ov-file#install) portion of the README file.

The following command can be used to run `Pcredz` against a packet capture file:

```shell-session
[!bash!]$ ./Pcredz -f demo.pcapng -t -v

Pcredz 2.0.2
Author: Laurent Gaffie
Please send bugs/comments/pcaps to: laurent.gaffie@gmail.com
This script will extract NTLM (HTTP,LDAP,SMB,MSSQL,RPC, etc), Kerberos,
FTP, HTTP Basic and credit card data from a given pcap file or from a live interface.

CC number scanning activated

Unknown format, trying TCPDump format

[1746131482.601354] protocol: udp 192.168.31.211:59022 > 192.168.31.238:161
Found SNMPv2 Community string: s3cr...SNIP...

[1746131482.601640] protocol: udp 192.168.31.211:59022 > 192.168.31.238:161
Found SNMPv2 Community string: s3cr...SNIP...

<SNIP>

[1746131482.658938] protocol: tcp 192.168.31.243:55707 > 192.168.31.211:21
FTP User: le...SNIP...
FTP Pass: qw...SNIP...

demo.pcapng parsed in: 1.82 seconds (File size 15.5 Mo).
```

### Lab - Questions

Download the attached `credential-hunting-in-network-traffic` and extract the `demo.pcapng` file, then use `Wireshark` or `PCredz` to answer the following questions.

{% embed url="https://academy.hackthebox.com/storage/modules/147/credential-hunting-in-network-traffic.zip" %}

* The packet capture contains cleartext credit card information. What is the number that was transmitted?

<figure><img src="../../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

* What is the SNMPv2 community string that was used?

> Git clone and install Pcredz

<figure><img src="../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

* What is the password of the user who logged into FTP?

<figure><img src="../../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

* What file did the user download over FTP?

<figure><img src="../../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

***

## Credential Hunting in Network Shares - Windows

#### **Common credential patterns**

Before diving into specialized tools, it's important to understand the types of patterns and file formats that often reveal sensitive information. This was covered in earlier sections, so we won't repeat it in detail here. But as a quick reminder, here are some general tips:

* Look for keywords within files such as `passw`, `user`, `token`, `key`, and `secret`.
* Search for files with extensions commonly associated with stored credentials, such as `.ini`, `.cfg`, `.env`, `.xlsx`, `.ps1`, and `.bat`.
* Watch for files with "interesting" names that include terms like `config`, `user`, `passw`, `cred`, or `initial`.
* If you're trying to locate credentials within the `INLANEFREIGHT.LOCAL` domain, it may be helpful to search for files containing the string `INLANEFREIGHT\`.
* Keywords should be localized based on the target; if you are attacking a German company, it's more likely they will reference a `"Benutzer"` than a `"User"`.
* Pay attention to the shares you are looking at, and be strategic. If you scan ten shares with thousands of files each, it's going to take a signifcant amount of time. Shares used by `IT employees` might be a more valuable target than those used for company photos.

With all of this in mind, you may want to begin with basic command-line searches (e.g., `Get-ChildItem -Recurse -Include *.ext \\Server\Share | Select-String -Pattern ...`) before scaling up to more advanced tools. Let's take a look at how we can use `MANSPIDER`, `Snaffler`, `SnafflePy`, and `NetExec` to automate and enhance this credential hunting process.

### **Snaffler**

The first tool we will cover is [Snaffler](https://github.com/SnaffCon/Snaffler). This is a C# program that, when run on a `domain-joined` machine, automatically identifies accessible network shares and searches for interesting files. The `README` file in the Github repository describes the numerous configuration options in great detail, however a basic search can be carried out like so:

```cmd-session
c:\Users\Public>Snaffler.exe -s

 .::::::.:::.    :::.  :::.    .-:::::'.-:::::':::    .,:::::: :::::::..
;;;`    ``;;;;,  `;;;  ;;`;;   ;;;'''' ;;;'''' ;;;    ;;;;'''' ;;;;``;;;;
'[==/[[[[, [[[[[. '[[ ,[[ '[[, [[[,,== [[[,,== [[[     [[cccc   [[[,/[[['
  '''    $ $$$ 'Y$c$$c$$$cc$$$c`$$$'`` `$$$'`` $$'     $$""   $$$$$$c
 88b    dP 888    Y88 888   888,888     888   o88oo,.__888oo,__ 888b '88bo,
  'YMmMY'  MMM     YM YMM   ''` 'MM,    'MM,  ''''YUMMM''''YUMMMMMMM   'W'
                         by l0ss and Sh3r4 - github.com/SnaffCon/Snaffler


[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:42Z [Info] Parsing args...
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Info] Parsed args successfully.
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Info] Invoking DFS Discovery because no ComputerTargets or PathTargets were specified
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Info] Getting DFS paths from AD.
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Info] Found 0 DFS Shares in 0 namespaces.
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Info] Invoking full domain computer discovery.
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Info] Getting computers from AD.
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Info] Got 1 computers from AD.
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Info] Starting to look for readable shares...
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Info] Created all sharefinder tasks.
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Black}<\\DC01.inlanefreight.local\ADMIN$>()
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\ADMIN$>(R) Remote Admin
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Black}<\\DC01.inlanefreight.local\C$>()
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\C$>(R) Default share
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\Company>(R)
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\Finance>(R)
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\HR>(R)
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\IT>(R)
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\Marketing>(R)
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\NETLOGON>(R) Logon server share
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\Sales>(R)
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\SYSVOL>(R) Logon server share
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:51Z [File] {Red}<KeepPassOrKeyInCode|R|passw?o?r?d?>\s*[^\s<]+\s*<|2.3kB|2025-05-01 05:22:48Z>(\\DC01.inlanefreight.local\ADMIN$\Panther\unattend.xml) 5"\ language="neutral"\ versionScope="nonSxS"\ xmlns:wcm="http://schemas\.microsoft\.com/WMIConfig/2002/State"\ xmlns:xsi="http://www\.w3\.org/2001/XMLSchema-instance">\n\t\t\ \ <UserAccounts>\n\t\t\ \ \ \ <AdministratorPassword>\*SENSITIVE\*DATA\*DELETED\*</AdministratorPassword>\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ </UserAccounts>\n\ \ \ \ \ \ \ \ \ \ \ \ <OOBE>\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ <HideEULAPage>true</HideEULAPage>\n\ \ \ \ \ \ \ \ \ \ \ \ </OOBE>\n\ \ \ \ \ \ \ \ </component
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:53Z [File] {Yellow}<KeepDeployImageByExtension|R|^\.wim$|29.2MB|2022-02-25 16:36:53Z>(\\DC01.inlanefreight.local\ADMIN$\Containers\serviced\WindowsDefenderApplicationGuard.wim) .wim
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:58Z [File] {Red}<KeepPassOrKeyInCode|R|passw?o?r?d?>\s*[^\s<]+\s*<|2.3kB|2025-05-01 05:22:48Z>(\\DC01.inlanefreight.local\C$\Windows\Panther\unattend.xml) 5"\ language="neutral"\ versionScope="nonSxS"\ xmlns:wcm="http://schemas\.microsoft\.com/WMIConfig/2002/State"\ xmlns:xsi="http://www\.w3\.org/2001/XMLSchema-instance">\n\t\t\ \ <UserAccounts>\n\t\t\ \ \ \ <AdministratorPassword>\*SENSITIVE\*DATA\*DELETED\*</AdministratorPassword>\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ </UserAccounts>\n\ \ \ \ \ \ \ \ \ \ \ \ <OOBE>\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ <HideEULAPage>true</HideEULAPage>\n\ \ \ \ \ \ \ \ \ \ \ \ </OOBE>\n\ \ \ \ \ \ \ \ </component
<SNIP>
```

All of the tools covered in this section output a `large amount of information`. While they assist with automation, a fair amount of manual review is typically required, as many matches may turn out to be `"false positives"`. Two useful parameters that can help refine Snaffler's search process are:

* `-u` retrieves a list of users from Active Directory and searches for references to them in files
* `-i` and `-n` allow you to specify which shares should be included in the search

### **PowerHuntShares**

Another tool that can be used is [PowerHuntShares](https://github.com/NetSPI/PowerHuntShares), a PowerShell script that doesn't necessarily need to be run on a domain-joined machine. One of its most useful features is that it generates an `HTML report` upon completion, providing an easy-to-use UI for reviewing the results:

![Summary report from PowerHuntShares showing findings: 5 critical, 0 high, 0 medium, 2 low. Data exposure includes 21 interesting, 2 sensitive, 2 secrets files.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/308/img/Net_1.png)

We can run a basic scan using `PowerHuntShares` like so:

```powershell-session
PS C:\Users\Public\PowerHuntShares> Invoke-HuntSMBShares -Threads 100 -OutputDirectory c:\Users\Public

 ===============================================================
 INVOKE-HUNTSMBSHARES
 ===============================================================
  This function automates the following tasks:

  o Determine current computer's domain
  o Enumerate domain computers
  o Check if computers respond to ping requests
  o Filter for computers that have TCP 445 open and accessible
  o Enumerate SMB shares
  o Enumerate SMB share permissions
  o Identify shares with potentially excessive privileges
  o Identify shares that provide read or write access
  o Identify shares thare are high risk
  o Identify common share owners, names, & directory listings
  o Generate last written & last accessed timelines
  o Generate html summary report and detailed csv files

  Note: This can take hours to run in large environments.
 ---------------------------------------------------------------
 |||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
 ---------------------------------------------------------------
 SHARE DISCOVERY
 ---------------------------------------------------------------
 [*][05/01/2025 12:51] Scan Start
 [*][05/01/2025 12:51] Output Directory: c:\Users\Public\SmbShareHunt-05012025125123
 [*][05/01/2025 12:51] Successful connection to domain controller: DC01.inlanefreight.local
 [*][05/01/2025 12:51] Performing LDAP query for computers associated with the inlanefreight.local domain
 [*][05/01/2025 12:51] -  computers found
 [*][05/01/2025 12:51] - 0 subnets found
 [*][05/01/2025 12:51] Pinging  computers
 [*][05/01/2025 12:51] -  computers responded to ping requests.
 [*][05/01/2025 12:51] Checking if TCP Port 445 is open on  computers
 [*][05/01/2025 12:51] - 1 computers have TCP port 445 open.
 [*][05/01/2025 12:51] Getting a list of SMB shares from 1 computers
 [*][05/01/2025 12:51] - 11 SMB shares were found.
 [*][05/01/2025 12:51] Getting share permissions from 11 SMB shares
<SNIP>
```

## Credential Hunting in Network Shares - Linux

### **MANSPIDER**

If we don’t have access to a domain-joined computer, or simply prefer to search for files remotely, tools like [MANSPIDER](https://github.com/blacklanternsecurity/MANSPIDER) allow us to scan SMB shares from Linux. It's best to run `MANSPIDER` using the official Docker container to avoid dependency issues. Like the other tools, `MANSPIDER` offers many parameters that can be configured to fine-tune the search. A basic scan for files containing the string `passw` can be run as follows:

```shell-session
eldeim@htb[/htb]$ docker run --rm -v ./manspider:/root/.manspider blacklanternsecurity/manspider 10.129.234.121 -c 'passw' -u 'mendres' -p 'Inlanefreight2025!'

[+] MANSPIDER command executed: /usr/local/bin/manspider 10.129.234.121 -c passw -u mendres -p Inlanefreight2025!
[+] Skipping files larger than 10.00MB
[+] Using 5 threads
[+] Searching by file content: "passw"
[+] Matching files will be downloaded to /root/.manspider/loot
[+] 10.129.234.121: Successful login as "mendres"
[+] 10.129.234.121: Successful login as "mendres"
<SNIP>
```

### **NetExec**

In addition to its many other uses, `NetExec` can also be used to search through network shares using the `--spider` option. This functionality is described in great detail on the [official wiki](https://www.netexec.wiki/smb-protocol/spidering-shares). A basic scan of network shares for files containing the string `"passw"` can be run like so:

```shell-session
eldeim@htb[/htb]$ nxc smb 10.129.234.121 -u mendres -p 'Inlanefreight2025!' --spider IT --content --pattern "passw"

SMB         10.129.234.121  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:inlanefreight.local) (signing:True) (SMBv1:False)
SMB         10.129.234.121  445    DC01             [+] inlanefreight.local\mendres:Inlanefreight2025! 
SMB         10.129.234.121  445    DC01             [*] Started spidering
SMB         10.129.234.121  445    DC01             [*] Spidering .
<SNIP>
```

***

### Lab - Questions

Use the credentials `mendres:Inlanefreight2025!` to connect to the target either by RDP or WinRM, then use the tools and techniques taught in this section to answer the questions below. For your convenience, `Snaffler` and `PowerHuntShares` can be found in `C:\Users\Public`.

* One of the shares mendres has access to contains valid credentials of another domain user. What is their password?

Fristly, i will connect via rdp:

```
xfreerdp /v:10.129.1.79 /u:mendres /p:Inlanefreight2025! /clipboard
```

<figure><img src="../../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

* As this user, search through the additional shares they have access to and identify the password of a domain administrator. What is it?

With it new credential, will be to connecto at it users via RDP

```
xfreerdp /v:10.129.1.79 /u:jbader /p:ILovePower333### /clipboard
```

<figure><img src="../../../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

Now, search by "Administrator" using Powershell

```
Get-ChildItem -Path 'C:\HR\' -Recurse -File |
    Select-String -Pattern 'Administrator' |
    ForEach-Object { "$($_.Path):$($_.LineNumber):$($_.Line)" }
```

<figure><img src="../../../.gitbook/assets/image (13) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (14) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (15) (1).png" alt=""><figcaption></figcaption></figure>
