---
hidden: true
---

# 📋 Índice

## Indice 2 - Mio

Comentar que todo lo que ves es sacado de mi reporsitorio de git, no puede hacer un git pull o subir lo que sea nunca a no ser que yo t elo diga. Ademas, yo me encargare de hacer un git pull manuealmente para tener todo actualizado par ti, ya que yo uso el gitbook para tomar apuntes. Tambien decir que no todo de esta crtf crtp tiene que estar 100% terminado. Los ejercicios que yo voy sigueindo y toamndo notas estan totalmente realizados en el folder que se encutra bajo esta misma ruta del indice.md llamado LabManual\_files. Mi objetivo con esto es que me ayudes a ir realizandos loe ejercicos, preguntarte dudas que me ayudes con estas y posibles errores y ademas, cuando empiece el examen puede preguntarte e ir tomando notas yo de todo y me ayudes a avanzar y pwnear el examen si me trabo, gracias

## Indice CRTP — Certified Red Team Professional

> Busca por fase de ataque, herramienta, o tecnica. Todos los links son relativos.

***

### Por Fase de Ataque

#### 1. Evasion & Setup (bypass controles PS / AV)

* [LO-1](learning-objectives/learning-objetive-1.md) — InviShell, AMSI bypass, Script Block Logging bypass
* [LO-3](learning-objectives/learning-objetive-3.md) — InviShell setup detallado
* [Class Notes 1](class-notes/class-one-notes.md) — PowerShell tradecraft completo
* [Class Notes 2](class-notes/class-two-notes.md) — .NET tradecraft, loaders in-memory
* [Cheatsheet Enumeration](cheatsheet-fast-commands-enumeration.md) — InviShell + PowerView + ADModule setup rapido
* [Cheatsheet AppLocker](cheatsheet-fast-commands-explotation/cheetsheet-fast-commands-applocker-policy.md) — AppLocker bypass via registry

#### 2. Enumeracion AD

* [LO-1](learning-objectives/learning-objetive-1.md) — Enumeracion basica con InviShell activo
* [LO-2](learning-objectives/learning-objetive-2.md) — ACLs del grupo Domain Admins (PowerView)
* [LO-3](learning-objectives/learning-objetive-3.md) — Enumeracion continuada
* [LO-4](learning-objectives/learning-objetive-4.md) — Forest trust enumeration, todos los dominios del forest
* [Cheatsheet Enumeration](cheatsheet-fast-commands-enumeration.md) — comandos rapidos PowerView / ADModule / BloodHound

#### 3. Local Privilege Escalation

* [LO-5](learning-objectives/learning-objetive-5.md) — Service abuse en student VM (PowerUp, Jenkins)
* [Cheatsheet PrivEsc](cheatsheet-fast-commands-privilege-escalation.md) — whoami /all, net localgroup, PowerUp, WinPEAS

#### 4. Domain Privilege Escalation

* [LO-6](learning-objectives/learning-objetive-6.md) — GPO abuse, Group Policy attribute modification
* [LO-7](learning-objectives/learning-objtetive-7.md) — Kerberoasting, NTLM hash extraction de service accounts
* [LO-14](learning-objectives/learning-objetive-14.md) — Kerberoasting de SQL Server service account
* [LO-15](learning-objectives/learning-objetive-15.md) — Unconstrained Delegation + Printer Bug → Enterprise Admin
* [LO-16](learning-objectives/learning-objetive-16.md) — Constrained Delegation (usuarios y cuentas maquina), TGT/TGS
* [LO-17](learning-objectives/learning-objetive-17.md) — ACL Write permissions abuse en computer objects
* [Cheatsheet Domain/Forest PrivEsc](cheatsheet-fast-commands-domain-forest-privilege-escalation.md) — Trust Key attack, evasive-silver, asktgs

#### 5. Domain Persistence

* [LO-8](learning-objectives/learning-objetive-8.md) — Golden Ticket (extrae hashes DC, crea golden ticket)
* [LO-9](learning-objectives/learning-objetive-9.md) — Silver Ticket (HTTP y WMI command execution)
* [LO-10](learning-objectives/learning-objetive-10.md) — Diamond Ticket attack
* [LO-11](learning-objectives/learning-objetive-11.md) — DSRM persistence (DSRM admin credential abuse)
* [LO-12](learning-objectives/learning-objetive-12.md) — DCSync (replication rights, pull krbtgt hash)
* [LO-13](learning-objectives/learning-objetive-13.md) — Security descriptor modification (WMI/PSRemoting sin DA)
* [Cheatsheet Persistence](cheatsheet-fast-commands-persistence.md) — DSRM remote attack, tickets

#### 6. Cross-Trust / Forest Attacks

* [LO-4](learning-objectives/learning-objetive-4.md) — Enumerar trusts inter-forest
* [LO-18](learning-objectives/learning-objetive-18.md) — Escalacion via trust key (DA → Enterprise Admin)
* [LO-19](learning-objectives/learning-objetive-19.md) — Escalacion via krbtgt hash, inter-realm TGT
* [Cheatsheet Domain/Forest PrivEsc](cheatsheet-fast-commands-domain-forest-privilege-escalation.md) — Trust Key attack completo

#### 7. Lateral Movement

* [Cheatsheet Lateral Movement](cheatsheet-fast-commands-lateral-movement.md) — Find-PSRemotingLocalAdminAccess, WinRM, pivoting

#### 8. Post-Exploitation

* [Cheatsheet Post-Exploitation](cheatsheet-fast-commands-post-explotation.md) — tabla comparativa Golden/Silver/Diamond, comandos post-DA

***

### Por Herramienta

| Herramienta                         | Archivos relevantes                                                                                                                                                                                                                                                                                                           |
| ----------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **InviShell**                       | [LO-1](learning-objectives/learning-objetive-1.md), [LO-3](learning-objectives/learning-objetive-3.md), [Cheat Enum](cheatsheet-fast-commands-enumeration.md), [Class 1](class-notes/class-one-notes.md)                                                                                                                      |
| **PowerView**                       | [LO-2](learning-objectives/learning-objetive-2.md), [LO-4](learning-objectives/learning-objetive-4.md), [LO-17](learning-objectives/learning-objetive-17.md), [Cheat Enum](cheatsheet-fast-commands-enumeration.md)                                                                                                           |
| **ADModule**                        | [LO-4](learning-objectives/learning-objetive-4.md), [Cheat Enum](cheatsheet-fast-commands-enumeration.md)                                                                                                                                                                                                                     |
| **BloodHound**                      | [Cheat Enum](cheatsheet-fast-commands-enumeration.md)                                                                                                                                                                                                                                                                         |
| **Rubeus**                          | [LO-7](learning-objectives/learning-objtetive-7.md), [LO-8](learning-objectives/learning-objetive-8.md), [LO-9](learning-objectives/learning-objetive-9.md), [LO-15](learning-objectives/learning-objetive-15.md), [LO-16](learning-objectives/learning-objetive-16.md), [LO-19](learning-objectives/learning-objetive-19.md) |
| **Loader.exe**                      | [Class 2](class-notes/class-two-notes.md), [LO-19](learning-objectives/learning-objetive-19.md)                                                                                                                                                                                                                               |
| **SafetyKatz**                      | [LO-7](learning-objectives/learning-objtetive-7.md), [LO-8](learning-objectives/learning-objetive-8.md)                                                                                                                                                                                                                       |
| **PowerUp**                         | [LO-5](learning-objectives/learning-objetive-5.md), [Cheat PrivEsc](cheatsheet-fast-commands-privilege-escalation.md)                                                                                                                                                                                                         |
| **WinPEAS / PrivEscCheck**          | [Cheat PrivEsc](cheatsheet-fast-commands-privilege-escalation.md)                                                                                                                                                                                                                                                             |
| **Find-PSRemotingLocalAdminAccess** | [Cheat Lateral](cheatsheet-fast-commands-lateral-movement.md)                                                                                                                                                                                                                                                                 |

***

### Por Tecnica / Keyword

| Tecnica / Keyword                                 | Archivos                                                                                                                                                                 |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **AMSI bypass**                                   | [LO-1](learning-objectives/learning-objetive-1.md), [Class 1](class-notes/class-one-notes.md), [Cheat Enum](cheatsheet-fast-commands-enumeration.md)                     |
| **Script Block Logging bypass**                   | [LO-1](learning-objectives/learning-objetive-1.md), [Class 1](class-notes/class-one-notes.md)                                                                            |
| **Constrained Language Mode (CLM)**               | [LO-1](learning-objectives/learning-objetive-1.md), [Class 1](class-notes/class-one-notes.md)                                                                            |
| **AppLocker bypass**                              | [Cheat AppLocker](cheatsheet-fast-commands-explotation/cheetsheet-fast-commands-applocker-policy.md)                                                                     |
| **ACL enumeration**                               | [LO-2](learning-objectives/learning-objetive-2.md), [Cheat Enum](cheatsheet-fast-commands-enumeration.md)                                                                |
| **ACL abuse / Write permissions**                 | [LO-17](learning-objectives/learning-objetive-17.md)                                                                                                                     |
| **Forest trust enumeration**                      | [LO-4](learning-objectives/learning-objetive-4.md), [Cheat Enum](cheatsheet-fast-commands-enumeration.md)                                                                |
| **GPO abuse**                                     | [LO-6](learning-objectives/learning-objetive-6.md)                                                                                                                       |
| **Service abuse (local privesc)**                 | [LO-5](learning-objectives/learning-objetive-5.md), [Cheat PrivEsc](cheatsheet-fast-commands-privilege-escalation.md)                                                    |
| **Kerberoasting**                                 | [LO-7](learning-objectives/learning-objtetive-7.md), [LO-14](learning-objectives/learning-objetive-14.md)                                                                |
| **AS-REP Roasting**                               | [LO-7](learning-objectives/learning-objtetive-7.md)                                                                                                                      |
| **Golden Ticket**                                 | [LO-8](learning-objectives/learning-objetive-8.md), [Cheat Post](cheatsheet-fast-commands-post-explotation.md), [Cheat Persist](cheatsheet-fast-commands-persistence.md) |
| **Silver Ticket**                                 | [LO-9](learning-objectives/learning-objetive-9.md), [Cheat Post](cheatsheet-fast-commands-post-explotation.md)                                                           |
| **Diamond Ticket**                                | [LO-10](learning-objectives/learning-objetive-10.md), [Cheat Post](cheatsheet-fast-commands-post-explotation.md)                                                         |
| **DSRM persistence**                              | [LO-11](learning-objectives/learning-objetive-11.md), [Cheat Persist](cheatsheet-fast-commands-persistence.md)                                                           |
| **DCSync**                                        | [LO-12](learning-objectives/learning-objetive-12.md), [LO-16](learning-objectives/learning-objetive-16.md)                                                               |
| **Security descriptor / WMI / PSRemoting sin DA** | [LO-13](learning-objectives/learning-objetive-13.md)                                                                                                                     |
| **Unconstrained Delegation**                      | [LO-15](learning-objectives/learning-objetive-15.md)                                                                                                                     |
| **Printer Bug (SpoolSample)**                     | [LO-15](learning-objectives/learning-objetive-15.md)                                                                                                                     |
| **Constrained Delegation**                        | [LO-16](learning-objectives/learning-objetive-16.md)                                                                                                                     |
| **Trust Key attack (DA → EA)**                    | [LO-18](learning-objectives/learning-objetive-18.md), [Cheat Domain/Forest](cheatsheet-fast-commands-domain-forest-privilege-escalation.md)                              |
| **Inter-realm TGT / krbtgt hash**                 | [LO-19](learning-objectives/learning-objetive-19.md)                                                                                                                     |
| **In-memory execution / .NET Loaders**            | [Class 2](class-notes/class-two-notes.md)                                                                                                                                |
| **MDE evasion**                                   | [Class 2](class-notes/class-two-notes.md)                                                                                                                                |
| **Lateral movement / WinRM**                      | [Cheat Lateral](cheatsheet-fast-commands-lateral-movement.md)                                                                                                            |

***

### Todos los Archivos

#### Learning Objectives

| Archivo                                              | Descripcion                                                                |
| ---------------------------------------------------- | -------------------------------------------------------------------------- |
| [LO-1](learning-objectives/learning-objetive-1.md)   | PowerShell bypass: InviShell, AMSI, Script Block Logging, CLM              |
| [LO-2](learning-objectives/learning-objetive-2.md)   | Enumerate ACLs Domain Admins group con PowerView                           |
| [LO-3](learning-objectives/learning-objetive-3.md)   | InviShell setup + continuacion enumeracion AD                              |
| [LO-4](learning-objectives/learning-objetive-4.md)   | Forest trust enumeration, todos los dominios del forest                    |
| [LO-5](learning-objectives/learning-objetive-5.md)   | Local Privilege Escalation: service abuse, Jenkins                         |
| [LO-6](learning-objectives/learning-objetive-6.md)   | GPO abuse: modificacion de atributos Group Policy                          |
| [LO-7](learning-objectives/learning-objtetive-7.md)  | Kerberoasting: svcadmin hash, NTLM extraction (dcorp-mgmt/adminsrv)        |
| [LO-8](learning-objectives/learning-objetive-8.md)   | Golden Ticket: extraer hashes DC, crear y usar ticket                      |
| [LO-9](learning-objectives/learning-objetive-9.md)   | Silver Ticket: command execution via HTTP y WMI                            |
| [LO-10](learning-objectives/learning-objetive-10.md) | Diamond Ticket attack con DA privileges                                    |
| [LO-11](learning-objectives/learning-objetive-11.md) | DSRM persistence: abusing DSRM administrator credential                    |
| [LO-12](learning-objectives/learning-objetive-12.md) | DCSync: replication rights abuse, pull krbtgt hash                         |
| [LO-13](learning-objectives/learning-objetive-13.md) | Security descriptor mod en dcorp-dc (WMI/PSRemoting sin DA)                |
| [LO-14](learning-objectives/learning-objetive-14.md) | Kerberoasting: SQL Server service account, crack password                  |
| [LO-15](learning-objectives/learning-objetive-15.md) | Unconstrained Delegation + Printer Bug → Enterprise Admin                  |
| [LO-16](learning-objectives/learning-objetive-16.md) | Constrained Delegation: users y machines, TGT→TGS→DCSync                   |
| [LO-17](learning-objectives/learning-objetive-17.md) | ACL Write permissions abuse en computer objects                            |
| [LO-18](learning-objectives/learning-objetive-18.md) | Cross-forest: DA → Enterprise Admin via trust key                          |
| [LO-19](learning-objectives/learning-objetive-19.md) | Cross-forest: EA via krbtgt hash + inter-realm TGT (Rubeus evasive-golden) |
| [LO-20](learning-objectives/learning-objetive-20.md) | (vacio)                                                                    |

#### Cheatsheets

| Archivo                                                                                              | Descripcion                                                |
| ---------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| [Cheat Enumeration](cheatsheet-fast-commands-enumeration.md)                                         | InviShell + PowerView + ADModule: comandos rapidos de enum |
| [Cheat PrivEsc](cheatsheet-fast-commands-privilege-escalation.md)                                    | whoami, local privesc, PowerUp, WinPEAS                    |
| [Cheat Lateral Movement](cheatsheet-fast-commands-lateral-movement.md)                               | Find-PSRemotingLocalAdminAccess, WinRM, pivoting           |
| [Cheat Persistence](cheatsheet-fast-commands-persistence.md)                                         | DSRM remote attack, tickets                                |
| [Cheat Domain/Forest PrivEsc](cheatsheet-fast-commands-domain-forest-privilege-escalation.md)        | Trust Key attack, evasive-silver, asktgs                   |
| [Cheat Post-Exploitation](cheatsheet-fast-commands-post-explotation.md)                              | Tabla comparativa Golden/Silver/Diamond, comandos post-DA  |
| [Cheat AppLocker](cheatsheet-fast-commands-explotation/cheetsheet-fast-commands-applocker-policy.md) | AppLocker bypass via registry keys en dcorp-adminsrv       |

#### Class Notes

| Archivo                                         | Descripcion                                                       |
| ----------------------------------------------- | ----------------------------------------------------------------- |
| [Class Notes 1](class-notes/class-one-notes.md) | PowerShell tradecraft: InviShell, AMSI, Script Block Logging, CLM |
| [Class Notes 2](class-notes/class-two-notes.md) | .NET tradecraft: loaders, in-memory execution, MDE bypass         |
