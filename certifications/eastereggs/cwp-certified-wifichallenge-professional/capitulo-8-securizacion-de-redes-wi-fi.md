# Capítulo 8: Securización de redes Wi-Fi

## Securización de redes Wi-Fi

### Medidas Generales

* **Uso de Contraseñas Fuertes y Únicas**: Es crucial usar contraseñas robustas y únicas para la red inalámbrica y cambiar la contraseña predeterminada que viene con el router.
* **Aislamiento de Clientes**: Permitir que los dispositivos conectados a la misma red Wi-Fi no se comuniquen entre sí directamente, previniendo la propagación de ataques.
* **Configurar una Red de Invitados**: Es aconsejable configurar una red de invitados separada de la red principal para limitar el acceso a los recursos internos.
* **Uso de un SSID no Predeterminado**: Cambiar el SSID (Identificador de Conjunto de Servicios) predeterminado a algo único y menos identificable mejora la seguridad y evita que los atacantes exploten configuraciones predeterminadas comunes. No se debe identificar a uno mismo o a la empresa en el nombre a menos que sea necesario.
* **Filtrado de Direcciones MAC**: Habilitar el filtrado de direcciones MAC para permitir que solo dispositivos específicos se conecten a la red mediante la creación de una lista blanca de sus direcciones de hardware.
* **Reducción de la Potencia de Transmisión**: Ajustar la potencia de transmisión de la señal Wi-Fi al mínimo requerido para cubrir el área deseada puede reducir el rango en el cual los atacantes podrían intentar atacar la red desde el exterior de las instalaciones.
* **Monitoreo Regular de la Red**: Implementar un sistema de monitoreo regular para detectar y responder a actividades sospechosas en la red.
* **Habilitar MFP (Management Frame Protection)**: Proteger los marcos de gestión (Management Frames) es esencial para evitar varios tipos de ataques a la administración de la red inalámbrica. Se debe habilitar MFP en los routers compatibles.
* **Transmisión inalámbrica autorizada**: Asegurarse de configurar correctamente todos los puntos de acceso con el código de país correspondiente para emitir de forma legal solo en canales autorizados en la banda de 5 Ghz sin interferir en ningún canal DFS.
* **Software y firmware actualizado**: Actualizar el software y el firmware de los dispositivos que actúen como puntos de acceso para verificar que tienen las últimas actualizaciones y parches contra posibles vulnerabilidades en los diferentes protocolos.

### Redes OPN (Open)

* **Migración de OPN a OWE (WPA3-OWE)**: el cifrado OWE (Opportunistic Wireless Encryption) ofrece cifrado en redes abiertas, evitando que los atacantes capturen datos transmitidos entre dispositivos. Se recomienda actualizar las redes abiertas de OPN (Open) a OWE para mejorar la seguridad.

### Redes WEP (Wired Equivalent Privacy)

* **Desactivar WEP**: WEP es un protocolo obsoleto y vulnerable a múltiples tipos de ataques. Se recomienda desactivar WEP y actualizar a protocolos más seguros como WPA2 o WPA3.

### Redes PSK (Pre-Shared Key)

* **Desactivar WPS (Wi-Fi Protected Setup)**: Se debe desactivar WPS debido a sus conocidas vulnerabilidades que permiten a los atacantes acceder a la red Wi-Fi mediante ataques de fuerza bruta.
* **Migración de WPA/WPA2-PSK a WPA3-SAE**: WPA3-SAE (Simultaneous Authentication of Equals) proporciona una autenticación más robusta y resistencia a los ataques de fuerza bruta. Se recomienda cambiar de WPA/WPA2-PSK a WPA3-SAE.
* **Rotación de Claves Regular**: Implementar una política de rotación de claves regularmente para minimizar el riesgo de que las claves PSK sean comprometidas.

### Redes MGT Enterprise

* **Utilización de Certificados**: Implementar el uso de certificados digitales para autenticar dispositivos y usuarios en la red, proporcionando un nivel adicional de seguridad.
* **Forzar Identidad Anónima**: Configurar la red para que los usuarios deban autenticarse utilizando una identidad anónima, lo cual añade una capa de privacidad.
* **Verificación de Certificados en los Clientes**: Asegurarse de que los dispositivos clientes verifiquen los certificados del servidor para prevenir ataques de intermediario (Man-in-the-Middle).
* **Implementación de Segmentación de Red**: Implementar la segmentación de red para limitar la propagación de ataques dentro de la red y mejorar la gestión del tráfico.
