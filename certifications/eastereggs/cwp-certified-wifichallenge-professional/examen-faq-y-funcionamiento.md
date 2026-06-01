# Examen FAQ y funcionamiento

100% de los retos del examen son como los de los ejercicios.

* **¿Cuánto tiempo se dispone para realizar el examen y cuál es el objetivo?**
  * Dispones de 6 horas para hackear al menos 4 de los 5 APs.
* **¿Cómo funciona el examen del CWP?**
  * Al registrarte para el examen, recibirás un correo electrónico con toda la información necesaria. Quince minutos antes de que comience el examen, recibirás otro correo electrónico con instrucciones detalladas, incluyendo la dirección IP y las credenciales de inicio de sesión para acceder al servidor remoto a través de SSH. El examen se lleva a cabo en un entorno aislado para garantizar la equidad. Se te proporcionará una configuración controlada donde deberás usar tus habilidades para completar las tareas asignadas dentro del tiempo establecido. Tu objetivo es explotar al menos 4 de las 5 redes Wi-Fi disponibles en el entorno. Asegúrate de documentar minuciosamente tus pasos y resultados para la presentación del informe final.
* **¿Cuánto tiempo tengo para reservar mi examen de certificación una vez que compro el curso?**
  * Tu cupón está "activo" durante 12 meses desde la fecha de compra; ese es tu plazo para asegurar una fecha de examen. Elige cualquier fecha disponible que se ajuste a tu plan de estudio antes de que venza. (Y no te preocupes si las cosas cambian: una vez que hayas reservado tu plaza, seguirás teniendo la flexibilidad habitual para modificar la cita según las políticas estándar de exámenes).\
    Se ha establecido este límite de un año para mantener el impulso: los alumnos que fijan una fecha con antelación tienen muchas más probabilidades de presentarse y obtener la certificación.
* **¿Qué evidencias son obligatorias en el informe?**
  * Es indispensable incluir una evidencia que muestre tanto el wpa\_supplicant conectado a la red como la flag en el servidor web con la URL.
* **¿Es necesario crackear contraseñas en el examen?**
  * Sí, y es necesario disponer de las herramientas necesarias (aircrack-ng, hashcat, etc.) funcionando en local ya que en el servidor no es posible realizar el cracking.
* **¿Qué requisitos previos se recomiendan antes de comenzar?**
  *   Se recomienda:

      Verificar la comprensión de todos los ataques en el laboratorio.

      Comprender los ataques avanzados presentes en el laboratorio o la documentación proporcionada.
* **¿Hay plantillas para redactar el walkthrough?**
  *   Sí, hay plantilla en Word y en Markdown.

      [Plantilla Word](https://import.cdn.thinkific.com/937577/0FmGFTC6T1GwBSu2GMPH_Plantilla%20Walkthrough%20CWP.docx)
  * [Plantilla Markdown](https://import.cdn.thinkific.com/937577/uSH6SfMfQkCKtBAa0Lpk_Plantilla%20Walkthrough%20CWP.md)
* **¿Hay algún examen de prueba?**
  * No, no existe un examen de práctica como tal. Sin embargo, tienes a tu disposición una versión del laboratorio con los datos cambiados (ESSID, MAC, BSSID, etc.) para que puedas repetir los ejercicios y reforzar tus habilidades antes del examen.
* **¿Dónde se encuentran las contraseñas necesarias para las fuerzas brutas?**
  * Todas las contraseñas para descifrar están dentro del archivo rockyou ubicado en el directorio /root.
* **¿Es necesario utilizartúneles SSH?**
  * Sí, es necesario tener comprensión sobre el túnel SSH para conectarse al examen.
* **¿Cómo se debe enviar el informe final?**
  * Se debe subir a la web del curso en el apartado de Envio de Examen en formato PDF máximo 24 horas después de la finalización del examen.
* **¿Hay feedback del examen si suspendo?**
  * Sí, si no te ha llegado feedback tras entregar el examen puedes mandar un mail a para recibir una respuesta con feedback de tu examen.
* **¿En qué idiomas se puede hacer el reporte?**
  * Se puede escribir en español o en inglés.
* **¿Cómo reinicio o gestiono el servidor del examen?**
  * Una vez comience el examen te llegará un mail con toda la información necesaria, así como un link a una web para gestionar el examen. Tienes 3 opciones:
  * **Reiniciar**: Reinicia la máquina del examen completamente, tanto los APs, clientes como el atacante al que estás conectado. Similar al script del lab.
* **Resetear**: Destruye el entorno y lo vuelve a crear. En este caso se destruye también el atacante, por lo que perderás cualquier información que haya en el equipo remoto.
* **Finalizar**: Apaga el examen y finaliza todo.
* **¿Cuándo recibo el código para el examen**?
  * Marca como completada la siguiente lección (**Plantillas Walkthrough Examen**) y en un plazo máximo de 5 minutos te debería llegar un mail con el código.
* **¿Es posible cancelar o reprogramar el examen?**
  * Sí, puedes hacerlo hasta 2 horas antes de la hora programada del examen.
* **¿Por qué cuando accedo al router en los retos me aparece un error y me redirecciona a HTTPS?**
  * Esto puede pasar porque el servidor se ha caído y lo mejor es ejecutar el script de reinicio o reiniciar la VM o porque ha habido un problema con la IP, esto se puede solucionar quitando el DHCP con dhclient wlanX -r y ejecutándolo de nuevo normalmente.
* **¿Cuánto tiempo es válido el cupón de examen después de la compra?**
  * El cupón de examen es válido por 1 año a partir de la fecha de compra (a menos que hayas comprado el curso en la preventa). Asegúrate de reservar tu examen antes de que expire el cupón.
* ¿Cuál es el formato de las flags en el examen?
  * Las flags en el examen siempre siguen este formato: **flag{XXXXXXXXXXXXXXXXXXXX}**. La parte dentro de las llaves representa una cadena de que será única para cada reto o problema.
* **¿Cuánto se tarda en obtener la corrección del examen?**
  * Normalmente se envía la corrección del examen en un plazo de 2 días laborables. Si se alarga puedes mandar un mail a support@wifichallenge.com
