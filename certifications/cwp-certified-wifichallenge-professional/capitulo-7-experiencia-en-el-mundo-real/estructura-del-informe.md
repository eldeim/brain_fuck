# Estructura del Informe

### 1. Portada

* **Título del Informe:** Informe de Pentesting de Redes Wi-Fi
* **Cliente:** Nombre de la empresa o persona para quien se hizo el trabajo
* **Fecha:** Fecha de entrega del informe
* **Autor:** Nombre del pentester o equipo que hizo el trabajo

### 2. Índice

Un índice detallado para que sea fácil encontrar lo que buscas en el informe.

### 3. Resumen Ejecutivo

* **Propósito del Pentesting:** Una breve explicación de por qué se hizo el pentesting.
* **Alcance del Pentesting:** Qué partes de la red Wi-Fi se evaluaron, incluyendo ubicaciones y segmentos específicos.
* **Metodología:** Un resumen de las técnicas y herramientas que se usaron.
* **Principales Hallazgos:** Un resumen de las vulnerabilidades más importantes que se encontraron.
* **Análisis General:** Una evaluación general del estado de seguridad de la red Wi-Fi.
* **Recomendaciones:** Un resumen de las acciones sugeridas para corregir los problemas.

### 4. Alcance y Metodología

* **Alcance del Trabajo:** Detalles sobre las redes Wi-Fi que se evaluaron, incluyendo SSIDs y ubicaciones.
* **Metodología Utilizada:** Una descripción detallada de las técnicas y herramientas empleadas, tales como:
*
  * Escaneo de redes
  * Captura de paquetes
  * Análisis de configuraciones de seguridad
  * Pruebas de autenticación y cifrado

Para este pentesting, se utilizó la metodología OSSTMM ( Open Source Security Testing Methodology Manual). Esta metodología proporciona un marco estructurado para evaluar la seguridad de redes y sistemas, asegurando que las pruebas sean exhaustivas, replicables y basadas en estándares reconocidos. Utilizar OSSTMM garantiza que el análisis sea riguroso y que las recomendaciones sean precisas y efectivas para mejorar la seguridad de la red Wi-Fi evaluada.

### 5. Detalles de los Hallazgos

Cada hallazgo debe ser documentado de manera detallada, incluyendo:

* **Descripción de la Vulnerabilidad:** Una explicación clara de la vulnerabilidad encontrada.
* **Impacto Potencial:** Qué tan grave podría ser si alguien explotara esta vulnerabilidad.
* **Evidencia:** Capturas de pantalla, logs y otros datos que demuestren el hallazgo.
* **Recomendaciones:** Acciones sugeridas para solucionar el problema, clasificadas por prioridad.

Ejemplo de formato para un hallazgo:

| **Vulnerabilidad**    | **Contraseña Débil WPA2-PSK**                                                                       |
| --------------------- | --------------------------------------------------------------------------------------------------- |
| **Descripción**       | La red Wi-Fi **wifi-mobile** usa una contraseña débil.                                              |
| **Impacto Potencial** | Un atacante podría romper la contraseña con fuerza bruta, obteniendo acceso no autorizado a la red. |
| **Evidencia**         | Captura de pantalla del intento exitoso de crackeo.                                                 |
| **Recomendaciones**   | Cambiar a una contraseña más fuerte y utilizar WPA3 si es posible.                                  |

### 6. Recomendaciones Generales

* **Mejoras en la Configuración:** Sugerencias para mejorar las configuraciones de seguridad de la red Wi-Fi.
* **Capacitación y Concienciación:** Recomendaciones sobre la capacitación del personal en prácticas de seguridad.
* **Políticas de Seguridad:** Sugerencias para implementar o mejorar políticas de seguridad de redes Wi-Fi.

### 7. Anexos

* **Herramientas Utilizadas:** Lista de herramientas de pentesting utilizadas, como aircrack-ng, Wireshark, etc.
* **Referencias:** Documentación adicional, guías de configuración y recursos utilizados para el informe.
* **Glosario de Términos:** Definiciones de términos técnicos utilizados en el informe.

### Mejores Prácticas para la Creación del Informe

* **Claridad y Concisión:** Usa un lenguaje claro y evita jerga técnica innecesaria.
* **Evidencia Visual:** Incluye capturas de pantalla, diagramas y logs para respaldar los hallazgos.
* **Prioriza las Recomendaciones:** Clasifica las recomendaciones según la gravedad de las vulnerabilidades.
* **Revisiones y Validaciones:** Realiza revisiones internas del informe para asegurar su precisión y completitud.
* **Confidencialidad:**&#x41;segúrate de que el informe sea tratado de manera confidencial y solo accesible para personas autorizadas.
