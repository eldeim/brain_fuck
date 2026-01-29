# Capitulo 2: WiFiChallenge Lab - Primeros Pasos

## Cómo instalar la VM

### Docker vs VM

Las diferencias entre Docker y máquinas virtuales (VM)se centran en su enfoque de aislamiento y utilización de recursos. Docker utiliza contenedores para ejecutar aplicaciones, haciéndolos livianos y portables. En contraste, las VM encapsulan sistemas operativos completos, incluyendo el Kernel, lo que puede llevar a un mayor consumo de recursos. Los contenedores comparten el Kernel del sistema anfitrión, lo que los hace más eficientes en términos de uso de recursos y más rápidos de iniciar que las VM. Esto hace que Docker sea una excelente opción para el despliegue de aplicaciones, donde se necesitan múltiples entornos replicables de manera rápida y eficiente. Sin embargo, las VM proporcionan un nivel de aislamiento más profundo y son más adecuadas para ejecutar aplicaciones con diferentes requisitos de sistemas operativos. En la siguiente imagen podemos ver una comparación entre máquinas virtuales y Docker donde se puede ver como las VM duplican en vez de compartir partes del Sistema Operativo.

<figure><img src="../../.gitbook/assets/image (292).png" alt=""><figcaption></figcaption></figure>

### Instalación usando VirtualBox o VMWare

Una vez hecho esto, ya podemos descargar la máquina virtual desde Proton Drive, que aproximadamente ocupa 5 Gigabytes. Para ello, la mejor opción es ir a y crear una cuenta allí. En esta página encontrarás todos los desafíos y la VM para todos los ejercicios prácticos. Puedes encontrar el enlace en la página README del sitio web.

<figure><img src="../../.gitbook/assets/image (293).png" alt=""><figcaption></figcaption></figure>

Ahora pulsamos en la carpeta de VirtualBox y hacemos clic en Download o Scan & Download. Esto descargará un archivo ZIP con los 2 archivos. Si se pulsa Download sin marcar nada, se descargará tanto la VM para VirtualBox como para el resto de los sistemas, descargando el triple de ficheros.

<figure><img src="../../.gitbook/assets/image (294).png" alt=""><figcaption></figcaption></figure>

También puedes descargar directamente la VM desde aquí: [WiFiChallenge Lab - Proton Drive](https://drive.proton.me/urls/Q4WPB23W7R#Qk4nxMH8Q4oQ)

Una vez que tenemos el archivo ZIP descargado, debemos descomprimirlo. Haz Clic derecho y selecciona Extract All. Esto creará una carpeta con los 2 archivos.

<figure><img src="../../.gitbook/assets/image (295).png" alt=""><figcaption></figcaption></figure>

Lo mejor es verificar el archivo. Para hacerlo, puedes verificar el hash de tu archivo para asegurar que se descargó correctamente. Esto se puede hacer usando la utilidad 7zip (www.7-zip.org/download.html) si está instalado en Windows o con sum256 en Linux. Como puedes ver en la siguiente captura.

<figure><img src="../../.gitbook/assets/image (296).png" alt=""><figcaption></figcaption></figure>

Con 7zip instalado podemos pulsar botón derecho → 7-zip → CRC SHA y Test archive: Checksum:

<figure><img src="../../.gitbook/assets/image (298).png" alt=""><figcaption></figcaption></figure>

Si aparece There are no errors todo bien, si muestra algún error habría que descargar de nuevo la VM.

<figure><img src="../../.gitbook/assets/image (299).png" alt=""><figcaption></figcaption></figure>

Una vez descargado y verificado el archivo, podemos hacer doble clic en el archivo OVA y abrir el programa VirtualBox para importarlo directamente. Un archivo OVA es una imagen virtualizada para VirtualBox. Aquí podemos configurar la dirección donde queremos que se almacene la máquina virtual y la importamos.

Una vez importada, podemos ir a configuración para especificar la memoria (RAM) y el número de procesadores (CPU) que queremos que utilice para ajustar la RAM y la CPU.

En el caso del proyecto, viene con 4 GB de RAM y 4 CPU, que es lo óptimo para que funcione. Pero si tenemos un equipo más potente, podemos darle más recursos. Mi recomendación es dar la mitad de RAM y CPU que tiene tu equipo. O, si no, dejarlo por defecto y todo debería funcionar perfectamente.

<figure><img src="../../.gitbook/assets/image (300).png" alt=""><figcaption></figcaption></figure>

Una vez hecho, podemos ejecutarlo y veremos una pantalla de inicio donde tenemos que poner el nombre y la contraseña. En este caso, tanto el usuario como la contraseña es user:

<figure><img src="../../.gitbook/assets/image (301).png" alt=""><figcaption></figcaption></figure>

Y ya está listo. Ya tenemos la máquina virtual instalada y funcionando.

<figure><img src="../../.gitbook/assets/image (302).png" alt=""><figcaption></figcaption></figure>

### Solucionar posibles errores durante la instalación

En el caso de que se muestre el siguiente error: `Oracle VM VirtualBox 7.0.18 needs the Microsoft Visual C++ 2019 Redistributable Package being installed first.`

<figure><img src="../../.gitbook/assets/image (303).png" alt=""><figcaption></figcaption></figure>

Esto se puede solucionar instalando la versión de Microsoft Visual C++ Redistributable más reciente desde la web de Microsoft (https://learn.microsoft.com/es-es/cpp/windows/latest-supported-vc-redist?view=msvc-170) o directamente desde este link: `https://aka.ms/vs/17/release/vc_redist.x64.exe`

***

## Gestionar la VM y posibles errores

### Instrucciones Importantes

Para iniciar sesión en el sistema, utiliza el usuario y la contraseña predeterminados: _**user/user**_. En caso de que ocurra algún error, puedes utilizar el script restartWiFi.sh para reiniciar y restaurar la configuración de Wi-Fi. Todas las herramientas necesarias se encuentran en la carpeta /root/tools.

Para los ataques de fuerza bruta, puedes utilizar el archivo rockyou ubicado en /root/rockyou-top100000.txt. Debes tener en cuenta que el tiempo máximo de espera entre automatizaciones, conexiones y otros procesos es de aproximadamente **3 a 5 minutos**. Aunque los desafíos están numerados, esta numeración es solo indicativa, lo que significa que si un desafío está desbloqueado, puedes realizarlo sin seguir un orden específico. Aunque todas las herramientas necesarias para el laboratorio están preinstaladas, puedes instalar herramientas adicionales si lo consideras necesario. En el contexto de una auditoría o Red Team real, el alcance incluye las siguientes redes:

* wifi-guest
* Red oculta: \<longitud: 9>
* wifi-mobile
* wifi-offices
* wifi-old
* wifi-management
* wifi-IT
* wifi-corp
* wifi-regional
* wifi-regional-tablets
* wifi-global

### Preguntas Frecuentes

* **¿Cuál es el formato de las Flags?**

Las flags son contraseñas o cadenas de texto que siguen el formato flag{XXXXXXX}.

* **¿Desde dónde se realizan los ataques?**

Todos los ataques se llevan a cabo desde la máquina virtual de Ubuntu.

* **¿Dónde están las herramientas?**

Todas las herramientas están ubicadas en la carpeta /root/tools.

* **¿Debo hacer algo al iniciar la VM?**

No, todo está configurado para ejecutarse automáticamente al iniciar la VM.

* **¿Qué hacer si hay problemas con algún AP o cliente?**

Si experimentas problemas con algún punto de acceso o cliente, ejecuta el script /root/restartWiFi.sh para reiniciar los APs y los clientes Wi-Fi. Si el problema persiste, puedes pedir ayuda en los canales de Discord o Telegram.

* **¿Todas las flags son de algún reto?**

No, existen flags señuelo que no están asociadas a ningún reto específico.

* **¿Por qué cuando accedo al router en los retos me aparece un error y me redirecciona a HTTPS?**

Esto puede pasar porque el servidor se ha caído y lo mejor es ejecutar el script de reinicio o reiniciar la VM o porque ha habido un problema con la IP, esto se puede solucionar quitando el DHCP con dhclient wlanX -r y ejecutándolo de nuevo normalmente.

***

## Extra: Instalar Hashcat en Windows

Para descargar Hashcat únicamente hay que ir a la página web pulsando y aparecerá la última versión, para descargar, en este caso la versión v6.2.6. Para descargar la última versión sería simplemente pulsar Download en la primera fila para descargar directamente los ejecutables.

<figure><img src="../../.gitbook/assets/image (304).png" alt=""><figcaption></figcaption></figure>

En el caso de que queramos descargar una versión antigua, como en el caso de necesitar crackear un hash con formato 2500 de Handshake de hostapd-mana podemos descargar la versión más moderna que dispone de esa versión, que es la versión v6.0.0.

<figure><img src="../../.gitbook/assets/image (305).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (306).png" alt=""><figcaption></figcaption></figure>

Una vez tenemos el fichero descargado, es un fichero 7z, por lo que podemos descomprimirlo con 7zip que hemos instalado para verificar el hash de WiFiChallenge Lab. En la siguiente imagen se puede ver cómo se descomprime el fichero.

Esto genera una carpeta con el mismo nombre que el fichero descargado. Dentro, podemos encontrar un gran número de ficheros, siendo el más importante hashcat.exe.

<figure><img src="../../.gitbook/assets/image (307).png" alt=""><figcaption></figcaption></figure>

Este programa es solo CLI, es decir, se tiene que ejecutar desde una terminal como la de Linux pero en Windows. La forma más fácil de abrir una terminal en la carpeta actual en Windows es pulsando Shift y Clic derecho en una parte en blanco del explorador del fichero, es decir, sin seleccionar ningún fichero.

<figure><img src="../../.gitbook/assets/image (308).png" alt=""><figcaption></figcaption></figure>

Una vez tenemos la PowerShell abierta, podemos ejecutar .\hashcat.exe para ejecutar el programa. La ventaja de ejecutarlo en el host es que si tenemos una tarjeta gráfica instalada con los drivers configurados podemos utilizar esta tarjeta gráfica para sacar toda la potencia a Hashcat, pudiendo crackear más de 1000 veces más rápido.

<figure><img src="../../.gitbook/assets/image (309).png" alt=""><figcaption></figcaption></figure>

La única diferencia entre los comandos para PowerShell y los que van a aparecer en el curso es el nombre del binario para ejecutarlo, por lo que si estamos en dicha carpeta y ejecutamos .\hashcat.exe donde aparece Hashcat en los ejercicios, todo lo demás es igual.

***

## WiFiChallenge Lab - Challenges capítulo 02

En este capítulo vamos a realizar los siguientes retos:

* Challenge 0 - ¿Cuál es el contenido del fichero /root/flag.txt en la máquina virtual?

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
