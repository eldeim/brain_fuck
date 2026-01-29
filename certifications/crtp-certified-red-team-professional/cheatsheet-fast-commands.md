# 🏁 Cheatsheet - Fast Commands

## Invisible Shells

<table><thead><tr><th width="134">Herramienta</th><th width="159">Para qué sirve</th><th width="410">Ejemplos de comandos</th></tr></thead><tbody><tr><td><strong>Invisi-Shell</strong></td><td>PowerShell stealth (AMSI + logging bypass)</td><td><p><code>C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat</code> </p><p><code>C:\AD\Tools\InviShell\RunWithPathAsAdmin.bat</code></p></td></tr><tr><td><strong>PowerView</strong></td><td>Enumeración ofensiva de Active Directory</td><td><p><code>. C:\AD\Tools\PowerView.ps1</code> </p><blockquote><p><code>powershell Get-DomainUser powershell Get-DomainGroup</code> </p><p><code>powershell Find-InterestingDomainAcl</code> </p><p><code>powershell Get-DomainObjectAcl -Identity administrador -ResolveGUIDs</code></p></blockquote></td></tr><tr><td><strong>ADModule</strong></td><td>Módulo oficial de Microsoft para administrar AD</td><td><p><code>Import-Module C:\AD\Tools\ADModulemaster\Microsoft.ActiveDirectory.Management.dll</code> </p><p><code>Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1</code> </p><blockquote><p><code>powershell Get-ADUser -Filter * powershell Get-ADGroup -Filter *</code></p></blockquote></td></tr></tbody></table>

### Using Invisi-Shell

> • With admin privileges:\
> RunWithPathAsAdmin.bat\
> • With non-admin privileges:\
> RunWithRegistryNonAdmin.bat\
> • Type exit from the new PowerShell session to complete the clean-up.

```
cd \AD\Tools
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```

```
. C:\AD\Tools\PowerView.ps1
```
