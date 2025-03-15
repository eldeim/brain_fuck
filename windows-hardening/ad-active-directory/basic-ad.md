# Basic AD

<figure><img src="../../.gitbook/assets/Pasted image 20250214143841.png" alt=""><figcaption></figcaption></figure>

### Ventajas de un AD

Las principales ventajas de un AD son:

* **Gestión centralizada de identidades**: Todos los usuarios de la red se pueden configurar desde Active Directory con el mínimo esfuerzo.
* **Gestión de políticas de seguridad**: Se pueden configurar políticas de seguridad directamente desde Active Directory y aplicarlas a los usuarios y equipos de toda la red según sea necesario.

### Los grupos mas importantes en un AD

|   Security Group   | Description                                                                                                                                                                     |
| :----------------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|    Domain Admins   | Los usuarios de este grupo tienen privilegios administrativos sobre todo el dominio. Por defecto, pueden administrar cualquier equipo del dominio, incluidos los DC.            |
|  Server Operators  | Los usuarios de este grupo pueden administrar los Controladores de Dominio. No pueden cambiar ninguna pertenencia a grupos administrativos.                                     |
|  Backup Operators  | Los usuarios de este grupo pueden acceder a cualquier archivo, sin tener en cuenta sus permisos. Se utilizan para realizar copias de seguridad de los datos de los ordenadores. |
|  Account Operators | Los usuarios de este grupo pueden crear o modificar otras cuentas del dominio.                                                                                                  |
|    Domain Users    | Incluye todas las cuentas de usuario existentes en el dominio.                                                                                                                  |
|  Domain Computers  | Incluye todos los ordenadores existentes en el dominio.                                                                                                                         |
| Domain Controllers | Incluye todos los DC existentes en el dominio.                                                                                                                                  |

### Security Groups vs OUs

* **Las OUs** son útiles para aplicar políticas a usuarios y ordenadores, que incluyen configuraciones específicas que pertenecen a conjuntos de usuarios dependiendo de su rol particular en la empresa. Recuerda que un usuario sólo puede ser miembro de una única OU a la vez, ya que no tendría sentido intentar aplicar dos conjuntos diferentes de políticas a un único usuario.
* **Los Grupos de Seguridad**, por otro lado, se utilizan para otorgar permisos sobre los recursos. Por ejemplo, utilizarás grupos si quieres permitir que algunos usuarios accedan a una carpeta compartida o a una impresora de red. Un usuario puede formar parte de muchos grupos, lo que es necesario para conceder acceso a múltiples recursos.

### Authehication Methos

Se pueden utilizar dos protocolos para la autenticación de red en un AD de Windows:

* **Kerberos:** Utilizado por **cualquier versión reciente de Windows**. Es el protocolo por defecto en cualquier dominio reciente.
* **NetNTLM:** Protocolo de autenticación heredado que se mantiene por motivos de compatibilidad. Aunque es un protocolo antiguo, todavía se usa en algunos sistemas por **compatibilidad con versiones anteriores**.

### Kerberos Authentication

1º El usuario envía su nombre de usuario y una marca de tiempo cifrada mediante una clave derivada de su contraseña al Centro de Distribución de Claves (KDC) (un servicio instalado normalmente en el Controlador de Dominio encargado de crear los tiques Kerberos en la red)

<figure><img src="../../.gitbook/assets/Pasted image 20250223104639.png" alt=""><figcaption></figcaption></figure>



&#x20;El KDC creará y devolverá un Ticket Granting Ticket (TGT), que permitirá al usuario solicitar tickets adicionales para acceder a servicios específicos. La necesidad de un ticket para obtener más tickets puede sonar un poco rara, pero permite a los usuarios solicitar tickets de servicio sin tener que pasar sus credenciales cada vez que quieran conectarse a un servicio.

2º Cuando un usuario desea conectarse a un servicio de la red, como un recurso compartido, un sitio web o una base de datos, utilizará su TGT para solicitar al KDC un Ticket Granting Service (TGS). Para solicitar un TGS, el usuario enviará su nombre de usuario y una marca de tiempo cifrada utilizando la Clave de Sesión, junto con el TGT y un Nombre Principal de Servicio (SPN), que indica el servicio y el nombre del servidor al que pretendemos acceder.

<figure><img src="../../.gitbook/assets/Pasted image 20250223105130.png" alt=""><figcaption></figcaption></figure>

El TGS se cifra utilizando una clave derivada del hash del propietario del servicio. El Propietario del Servicio es la cuenta de usuario o máquina bajo la que se ejecuta el servicio.

3º El TGS puede enviarse al servicio deseado para autenticar y establecer una conexión. El servicio utilizará el hash de la contraseña de su cuenta configurada para descifrar el TGS y validar la Clave de Sesión de Servicio.

<figure><img src="../../.gitbook/assets/Pasted image 20250223105419.png" alt=""><figcaption></figcaption></figure>

### NetNTLM Authentication

NetNTLM funciona mediante un mecanismo de desafío-respuesta:

1º El cliente envía una solicitud de autenticación al servidor al que quiere acceder. 2º El servidor genera un número aleatorio y lo envía como desafío al cliente. 3º El cliente combina el hash de su contraseña NTLM con el desafío (y otros datos conocidos) para generar una respuesta al desafío y la envía de vuelta al servidor para su verificación. 4º El servidor reenvía el desafío y la respuesta al controlador de dominio para su verificación. 5º El controlador de dominio utiliza el desafío para recalcular la respuesta y la compara con la respuesta original enviada por el cliente. Si ambas coinciden, se autentica al cliente; de lo contrario, se deniega el acceso. 6º El resultado de la autenticación se devuelve al servidor. 7º El servidor reenvía el resultado de la autenticación al cliente.

<figure><img src="../../.gitbook/assets/Pasted image 20250223105638.png" alt=""><figcaption></figcaption></figure>
