# Checklist

### Checklist Antes de comenzar

* Obtener los permisos necesarios
* Verificar Adaptadores Wi-Fi y sus Drivers. Generalmente se utilizan 2 adaptadores externos y el incorporado en el ordenador portátil, ya sea mediante una USB en vivo o directamente con Linux.
* Configurar y probar todos los equipos y herramientas
* Asegurarse de que las baterías y fuentes de alimentación estén completamente cargadas en el caso de que se vaya a hacer más orientado a Red Team (sin apoyo del cliente)

### Checklist de Vulnerabilidades de Configuración y Soluciones

A continuación, se presenta una lista de posibles vulnerabilidades en la configuración de las redes Wi-Fi, junto con las soluciones recomendadas para mitigarlas. Esta lista está dividida en diferentes categorías según el tipo de red y las tecnologías empleadas.

#### Vulnerabilidades Generales para Todas las Redes Wi-Fi

* **Protección de Tramas de Gestión Deshabilitada (MFP):**
*
  * **Vulnerabilidad:** Tramas de des autenticación y des asociación no protegidas.
  * **Solución:** Habilitar la Protección de Tramas de Gestión (MFP) en el punto de acceso para proteger contra estos tipos de ataques. Esto ayuda a evitar la desconexión no autorizada de clientes legítimos de la red.
* **Exceso de Rango de Cobertura:**
*
  * **Vulnerabilidad:** Señal Wi-Fi extendida más allá de las áreas necesarias, permitiendo a atacantes potenciales conectarse desde fuera de las instalaciones.
  * **Solución:** Ajustar la potencia de transmisión de los puntos de acceso para limitar la cobertura solo a las áreas necesarias. Realizar un estudio de sitio para optimizar la cobertura.
* **Redes Ocultas:**
*
  * **Vulnerabilidad:** Redes Wi-Fi ocultas que no transmiten su SSID pueden ser detectadas y atacadas por métodos avanzados.
  * **Solución:** Evaluar la necesidad de ocultar el SSID y, si se decide ocultarlo, combinarlo con otras medidas de seguridad robustas como WPA3 para dificultar los ataques.
* **Falta de Aislamiento de Clientes (**&#x43;lient Isolatio&#x6E;**):**
*
  * **Vulnerabilidad:** Usuarios pueden comunicarse directamente entre sí dentro de la misma red, lo que puede llevar a ataques internos.
  * **Solución:** Habilitar la función de aislamiento de clientes en el punto de acceso para evitar la comunicación directa entre dispositivos, reduciendo así el riesgo de ataques entre usuarios de la misma red.
* **Clientes con Probes Peligrosos:**
*
  * **Vulnerabilidad:** Dispositivos que envían solicitudes de probe a múltiples SSID, exponiéndose a ataques de rogue AP (punto de acceso falso).
  * **Solución:** Configurar dispositivos para evitar probes a SSID no conocidos y educar a los usuarios sobre la importancia de conectarse solo a redes de confianza.
* **SSID Genéricos:**
*
  * **Vulnerabilidad:** Uso de SSID predeterminados o comunes que pueden ser objetivos fáciles para atacantes.
  * **Solución:** Utilizar SSID únicos y no reveladores de la naturaleza o ubicación de la red para dificultar su identificación por parte de atacantes.
* **Ausencia de WIDS (**&#x57;ireless Intrusion Detection Syste&#x6D;**):**
*
  * **Vulnerabilidad:** Ausencia de un sistema de detección de intrusiones que monitoree y alerte sobre actividades sospechosas en la red.
  * **Solución:** Implementar y configurar un WIDS para monitorear el tráfico y detectar posibles intrusiones o comportamientos anómalos, proporcionando una capa adicional de seguridad.

#### Vulnerabilidades en Redes OPN (Open Networks)&#x20;

* **Red Abierta sin Cifrado (OPN):**
*
  * **Vulnerabilidad:** Los clientes de la red envían la información sin cifrar hacia el AP, lo que permite que cualquier atacante pueda ver la información en texto claro.
  * **Solución:** Si es necesario que la red no tenga autenticación, implementar OWE para cifrar la comunicación de los clientes. Si se pueden utilizar credenciales, implementar WPA3 con cifrado SAE o, como mínimo, WPA2 con PSK para asegurar que solo usuarios autorizados puedan acceder.
* **Información Sensible en Mensajes sin Cifrar:**
*
  * **Vulnerabilidad:** Tráfico sensible de red sin cifrar.
  * **Solución:** Implementar OWE y utilizar protocolos seguros como TLS o HTTPS para proteger la información sensible transmitida.
* **Portal cautivo sin TLS:**
*
  * **Vulnerabilidad:** Tráfico del login del portal cautivo sin cifrar.
  * **Solución:** Utilizar protocolos seguros como TLS o HTTPS para proteger la información sensible transmitida.
* **Acceso a Otros Segmentos de Red:**
*
  * **Vulnerabilidad:** Usuarios no autorizados pueden acceder a otros segmentos de la red, especialmente desde la red de invitados a la red interna.
  * **Solución:** Segmentar la red con VLANs y aplicar reglas estrictas de firewall para controlar el acceso entre segmentos, asegurando que los usuarios de la red de invitados no puedan acceder a recursos internos.
* **Debilidades del Portal Cautivo:**
*
  * **Vulnerabilidad:** Fallos en la implementación de portales cautivos pueden permitir el acceso no autorizado explotando el portal web.
  * **Solución:** Asegurar que el portal cautivo esté correctamente configurado y actualizado con las últimas medidas de seguridad. Realizar auditorías periódicas para identificar y corregir vulnerabilidades.
* **Bypass de Portal Cautivo:**
*
  * **Vulnerabilidad:** Técnicas que permiten a los usuarios evitar la autenticación del portal cautivo a nivel de red.
  * **Solución:** Implementar medidas adicionales de autenticación y monitoreo para detectar y prevenir intentos de bypass.

#### Vulnerabilidades en Redes OWE (Opportunistic Wireless Encryption)

* **Compatibilidad de Dispositivos:**
*
  * **Vulnerabilidad:** Dispositivos que no soportan OWE, forzando a caer a una red sin cifrar.
  * **Solución:** Asegurarse de que todos los dispositivos en la red soporten OWE y actualizar o reemplazar los dispositivos que no lo hagan para mantener la integridad de la seguridad de la red.

#### Vulnerabilidades en Redes WEP (Wired Equivalent Privacy)

* **Configuración de Claves WEP Estáticas y Débiles:**
*
  * **Vulnerabilidad:** Claves de cifrado fácilmente crackeables con herramientas de fuerza bruta.
  * **Solución:** Migrar a un protocolo de seguridad más robusto como WPA3 o WPA2 para garantizar una protección adecuada.
* **Uso Continuo de WEP:**
*
  * **Vulnerabilidad:** Uso de WEP, que es inherentemente inseguro.
  * **Solución:** Actualizar a WPA3 o WPA2 lo antes posible, ya que WEP no ofrece un nivel de seguridad aceptable en la actualidad.

#### Vulnerabilidades en Redes PSK (Pre-Shared Key) y SAE (Simultaneous Authentication of Equals)

* **Clave Precompartida Débil (PSK):**
*
  * **Vulnerabilidad:** Uso de contraseñas fáciles de adivinar o predecibles.
  * **Solución:** Implementar una política de contraseñas fuertes que incluya una combinación de letras, números y caracteres especiales. Cambiar las contraseñas regularmente para mantener la seguridad.
* **No Rotación de Claves (PSK):**
*
  * **Vulnerabilidad:** Uso prolongado de la misma clave sin cambios periódicos.
  * **Solución:** Establecer un calendario regular para cambiar las claves precompartidas y notificar a los usuarios de antemano para evitar interrupciones en el servicio.
* **Configuración Incorrecta de SAE:**
*
  * **Vulnerabilidad:** Errores en la configuración del protocolo SAE.
  * **Solución:** Verificar y asegurar que la configuración de WPA3 utiliza el protocolo SAE correctamente, revisando la documentación del fabricante y realizando pruebas de seguridad.
* **Compatibilidad de Dispositivos (SAE):**
*
  * **Vulnerabilidad:** Dispositivos que no soportan correctamente SAE.
  * **Solución:** Asegurarse de que todos los dispositivos en la red soporten SAE y actualizar o reemplazar los dispositivos que no lo hagan, garantizando la compatibilidad con los estándares de seguridad más recientes.
* **Filtrado de MAC Deshabilitado:**
*
  * **Vulnerabilidad:** No se está utilizando el filtrado de direcciones MAC para restringir el acceso a la red.
  * **Solución:** Implementar el filtrado de direcciones MAC para permitir solo dispositivos autorizados en la red, mejorando el control de acceso.
* **Rogue AP para Phishing de Contraseñas:**
*
  * **Vulnerabilidad:** Puntos de acceso falsos utilizados para engañar a los usuarios y obtener sus contraseñas.
  * **Solución:** Implementar un WIDS para detectar y alertar sobre puntos de acceso falsos. Educar a los usuarios sobre los peligros de conectarse a redes no verificadas para prevenir ataques de phishing.
* **Obtención de Información Sensible en Descifrado de Tráfico:**
*
  * **Vulnerabilidad:** Tráfico descifrado que puede contener información sensible.
  * **Solución:** Utilizar cifrado de extremo a extremo (como HTTPS) para proteger la información sensible transmitida sobre la red Wi-Fi, asegurando que los datos permanezcan seguros durante su tránsito.

#### Vulnerabilidades en Redes MGT (Management Frames Protection)

* **Bloqueo de Cuentas por Fuerza Bruta o Password Spraying:**
*
  * **Vulnerabilidad:** No hay un mecanismo para bloquear cuentas después de múltiples intentos fallidos.
  * **Solución:** Implementar políticas de bloqueo de cuentas después de un número determinado de intentos fallidos, reduciendo el riesgo de ataques de fuerza bruta.
* **No Uso de Identidades Anónimas:**
*
  * **Vulnerabilidad:** Uso de identidades reales en lugar de anónimas puede llevar a la exposición de información sensible.
  * **Solución:** Configurar identidades anónimas para el uso en autenticación de red cuando sea posible, protegiendo la privacidad de los usuarios.
* **Relay de Credenciales:**
*
  * **Vulnerabilidad:** Ataques de retransmisión de credenciales para obtener acceso no autorizado.
  * **Solución:** Utilizar autenticación mutua y cifrado fuerte para evitar ataques de retransmisión, asegurando que las credenciales no puedan ser reutilizadas por atacantes.
* **Rogue AP ya que los Clientes no Verifican CA:**
*
  * **Vulnerabilidad:** Clientes que no verifican la autenticidad del certificado del punto de acceso pueden conectarse a rogue APs.
  * **Solución:** Configurar clientes para que verifiquen siempre los certificados de los puntos de acceso y utilicen certificados firmados por una CA confiable para garantizar la autenticidad de las conexiones.
* **Phishing de Credenciales Mediante APs Falsos:**
*
  * **Vulnerabilidad**: APs falsos utilizados para engañar a los usuarios y obtener sus credenciales de acceso.
  * **Solución**: Implementar WIDS y educar a los usuarios sobre los peligros de conectarse a redes no verificadas. Utilizar autenticación multifactor para mitigar el riesgo de phishing.
* **Realización de ataques ESSID Stripping**:
*
  * **Vulnerabilidad**: Los atacantes pueden realizar ataques de stripping del ESSID para engañar a los clientes y hacer que se conecten a puntos de acceso falsos.
  * **Solución**: Configurar los clientes y puntos de acceso para que verifiquen siempre la autenticidad del ESSID y utilicen certificados firmados por una CA confiable. Implementar medidas de seguridad adicionales como WPA3 y educar a los usuarios sobre la importancia de conectarse solo a redes de confianza.
* **Interfaces de Administración por Defecto:**
*
  * **Vulnerabilidad:** Uso de configuraciones de administración por defecto que pueden ser conocidas por atacantes.
  * **Solución:** Cambiar las credenciales de administración por defecto y deshabilitar accesos innecesarios, asegurando que solo personal autorizado pueda realizar cambios en la configuración de la red.
