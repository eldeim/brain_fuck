# 📶 CWP-Certified WiFiChallenge Professional

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## Ver en formato Web <a href="#view-in-web-format" id="view-in-web-format"></a>

{% embed url="https://eldeim.gitbook.io/brain_fuck/checklists/certifications/cwp-certified-wifichallenge-professional" %}

***

## Estructura y Apartados

### 01. Teoría de Redes Wi-Fi

**Descripción:** Este apartado se enfoca en proporcionar las bases teóricas necesarias para entender los ataques Wi-Fi que se abordarán en módulos posteriores, evitando profundizar en detalles técnicos innecesarios.

**Contenido:**

* **Diagrama básico de Wi-Fi:** Introducción a la estructura básica de una red Wi-Fi doméstica y corporativa.
* **IEEE 802.11:** Explicación de los estándares de la familia IEEE 802.11 y sus principales características.
* **Terminología básica:** Definición de términos clave utilizados en redes Wi-Fi.
* **Estándares Wi-Fi:** Evolución de los estándares Wi-Fi desde su creación hasta las versiones más recientes.
* **Ancho de Banda del Wi-Fi:** Explicación de los anchos de banda HT20, HT40, HT80 y HT160 y su impacto en el rendimiento de la red.
* **Sondas, autenticación y asociación:** Descripción del proceso de conexión de un dispositivo a una red Wi-Fi, incluyendo mensajes de Beacon, solicitud de Probe, autenticación y asociación.
* **802.11w (MFP):** Detalles sobre la Protección de Tramas de Gestión y su importancia en la seguridad de redes Wi-Fi.
* **Tipos de Redes por Cifrado:** Diferentes tipos de redes Wi-Fi según su método de cifrado y autenticación.

**Indicaciones:** Este apartado se centra en proporcionar una comprensión básica pero suficiente de los conceptos y términos esenciales para los ejercicios prácticos de hacking Wi-Fi que se realizarán en los módulos posteriores.

### 02. WiFiChallenge Lab - Primeros pasos

**Descripción:**&#x45;ste apartado se enfoca en la configuración e introducción a WiFiChallenge Lab, un entorno de laboratorio creado en una máquina virtual (VM) para realizar prácticas y ejercicios de hacking Wi-Fi de manera segura.

**Contenido:**

* Qué es WiFiChallenge Lab y su funcionalidad.
* Instalación y configuración de la VM en VirtualBox.
* Primeros ejercicios y validación de configuración.
* Solución de problemas comunes durante la instalación.
* Instalación adicional de herramientas (como Hashcat en Windows).

**Indicaciones:**&#x45;ste apartado es crucial para preparar el entorno de trabajo que utilizaremos en los ejercicios prácticos del curso. Sigue cada paso cuidadosamente para asegurar que la VM se instale correctamente y esté lista para su uso.

### 03. Fundamentos de Linux - Cómo usar Linux

**Descripción:** Este apartado proporciona una introducción detallada al uso de Linux, enfocándose en la interfaz de línea de comandos (CLI), indispensable para hacking Wi-Fi. Se utiliza una distribución Ubuntu dentro de la VM del laboratorio WiFiChallenge Lab para todos los ejercicios.

**Contenido:**

* Introducción a Linux: Breve explicación del sistema operativo Linux y su importancia en el hacking Wi-Fi.
* Comenzando con la Terminal: Cómo abrir y usar la terminal en Ubuntu, incluyendo el uso del prompt de comandos y comandos básicos de navegación y gestión de archivos.
* Comprender su y sudo: Explicación de los comandos su y sudo para cambiar de usuario y ejecutar comandos con privilegios de administrador, respectivamente.
* Comandos Avanzados: Uso de herramientas avanzadas como tmux para gestionar múltiples sesiones de terminal desde una sola pantalla, gestión de permisos de archivos y directorios, búsqueda y localización de archivos, configuración y uso de túneles SSH, y el uso de herramientas de cracking como Hashcat, John the Ripper y aircrack-ng.

**Indicaciones:** Este apartado es fundamental para aprender a utilizar Linux de manera efectiva en el contexto del hacking Wi-Fi. Sigue cuidadosamente cada ejercicio en la VM del laboratorio para familiarizarte con los comandos y herramientas esenciales.

### 04. Redes Wi-Fi en Linux

**Descripción:** Este apartado aborda la gestión de redes Wi-Fi en Linux utilizando herramientas de línea de comandos. Se cubren las configuraciones básicas y avanzadas para la administración de interfaces Wi-Fi, escaneo de redes, modo monitor, monitoreo de tráfico y cambios de dirección MAC.

**Contenido:**

* Información de las Interfaces: Uso de los comandos iw, iwconfig, y ip para obtener información detallada de las interfaces Wi-Fi, y esquemas de nombres clásicos y modernos para las interfaces de red.
* Gestión de Interfaces Wi-Fi: Comandos para activar y desactivar interfaces Wi-Fi, escaneo de redes Wi-Fi utilizando iw y nmcli, y configuración de la interfaz en un canal específico con iwconfig.
* Modo Monitor: Procedimiento para habilitar y deshabilitar el modo monitor en una interfaz Wi-Fi, y uso del comando tcpdump para capturar y analizar paquetes TCP/IP.
* Cambiar la Dirección MAC: Procedimiento para cambiar la dirección MAC de una interfaz utilizando macchanger o ifconfig.
* Conexión a Redes Wi-Fi: Uso de nmcli y wpa\_supplicant para conectarse a redes Wi-Fi abiertas (OPN), OWE, WEP, WPA/WPA2-PSK, WPA3-SAE y WPA/WPA2/WPA3-Enterprise.
* Creación de Redes Wi-Fi (Access Points): Configuración de un punto de acceso Wi-Fi usando hostapd para diferentes tipos de redes (OPN, OWE, WEP, PSK, SAE, MGT), y generación de certificados y configuración avanzada para redes seguras.
* aircrack-ng: Instalación y uso básico de la suite aircrack-ng para auditorías de seguridad de redes inalámbricas, procedimientos para activar el modo monitor, inyección de paquetes y romper contraseñas de redes Wi-Fi.
* Wireshark: Instalación y uso básico de Wireshark para capturar y analizar paquetes de red, y procedimiento para ejecutar Wireshark con y sin privilegios de root.

**Indicaciones:** Este apartado es fundamental para aprender a manejar las interfaces Wi-Fi en Linux y realizar tareas de auditoría y pruebas de penetración. Sigue cuidadosamente cada ejercicio en la VM del laboratorio para familiarizarte con los comandos y herramientas esenciales.

### 05.0. Recon Wi-Fi Ofensivo

**Descripción:** Este apartado se centra en las técnicas y herramientas utilizadas para el reconocimiento Wi-Fi ofensivo. Se explica cómo configurar interfaces en modo monitor, capturar y analizar datos de redes inalámbricas y cómo identificar y procesar redes ocultas y clientes asociados.

**Contenido:**

* Introducción: Definición y objetivos del reconocimiento Wi-Fi ofensivo. Herramientas principales: airodump-ng y wifi\_db.
* Modo Monitor y Captura de Paquetes: Comandos para habilitar el modo monitor en una interfaz inalámbrica. Uso de airodump-ng para iniciar la captura de datos en todos los canales, filtrado por banda y canal, y opciones avanzadas para ordenar, seleccionar y colorear puntos de acceso y clientes. Detalle de la información mostrada en las secciones de APs y Clientes.
* Analizando los Datos Capturados: Guardado de paquetes capturados para análisis posterior. Uso de wifi\_db para importar y analizar datos de airodump-ng en una base de datos SQLite. Procedimiento para utilizar wifi\_db con y sin Docker. Acceso y análisis de la base de datos usando SQLitebrowser y sqlite3.
* Análisis de Redes Ocultas: Comandos para detectar redes ocultas utilizando iwlist y airodump-ng. Técnicas de fuerza bruta para descubrir ESSIDs ocultos usando mdk4. Uso de ataques de deautenticación para forzar la reconexión de clientes y capturar el ESSID.
* Herramientas Hacking Wi-Fi: Lista de herramientas útiles para auditorías y ataques a redes Wi-Fi, incluyendo aircrack-ng, hostapd-wpe, eaphammer, wifi\_db, airgeddon, Reaver, entre otras. Explicación breve de cada herramienta y su propósito.
* WiFiChallenge Lab: Desafíos prácticos que incluyen la identificación del canal utilizado por un AP específico (wifi-global), la identificación de la MAC del cliente asociado a la red wifi-IT, la identificación del probe de un cliente específico con formato wifi-, y el descubrimiento del ESSID de un AP oculto mediante fuerza bruta.

**Indicaciones:** Este apartado es esencial para aprender a realizar reconocimiento Wi-Fi ofensivo de manera efectiva. Sigue cuidadosamente cada ejercicio en la VM del laboratorio para familiarizarte con las herramientas y técnicas descritas.

### 05.1. Ataques Wi-Fi - OPN (Redes Públicas Abiertas)

**Descripción:** Este apartado explora técnicas y herramientas para explotar redes Wi-Fi públicas abiertas (OPN). Se abordan métodos para eludir portales cautivos, suplantación de MAC e IP, robo de credenciales, y creación de puntos de acceso falsos. También se detalla cómo visualizar el tráfico no cifrado de usuarios en estas redes.

**Contenido:**

* Teoría: Explicación de las redes OPN y su vulnerabilidad a diversos ataques.
* Bypass del Portal Cautivo: Métodos para eludir portales cautivos mediante suplantación de MAC e IP, y análisis de vulnerabilidades en portales cautivos no seguros.
* Punto de Acceso Falso (Evil Twin): Configuración y uso de puntos de acceso falsos para interceptar tráfico y realizar ataques de phishing.
* Visualización del Tráfico de Usuarios Sin Cifrar: Uso de herramientas como Wireshark y tcpdump para capturar y analizar tráfico en redes OPN.

**Indicaciones:** Este apartado es crucial para entender cómo se pueden explotar las vulnerabilidades en redes públicas abiertas. Sigue cada ejercicio en la VM del laboratorio para familiarizarte con los ataques y técnicas descritas.

### 05.2. Ataques Wi-Fi - OWE (Opportunistic Wireless Encryption)

**Descripción:** Este apartado explora las técnicas y herramientas para realizar ataques en redes Wi-Fi OWE (Opportunistic Wireless Encryption). Aunque OWE ofrece una mayor privacidad al cifrar el tráfico, se presentan métodos ofensivos para explotar estas redes mediante ingeniería social, creación de puntos de acceso falsos y ataques Man-in-the-Middle.

**Contenido:**

* **Teoría**: Introducción a OWE, su propósito y beneficios. Comparación con redes OPN y explicación de cómo OWE proporciona cifrado sin necesidad de una contraseña compartida.
* Enfoque Ofensivo: Descripción de varias tácticas ofensivas aplicables a redes OWE:
*
  * Creación de un punto de acceso falso (Evil Twin) utilizando herramientas como hostapd y airbase-ng.
  * Realización de ataques de ingeniería social mediante portales cautivos falsos usando herramientas como SET (Social Engineering Toolkit).
  * Ejecución de ataques Man-in-the-Middle (MitM) para interceptar y alterar la comunicación entre el usuario y el servidor utilizando técnicas como ARP spoofing y DNS spoofing con herramientas como Ettercap, Wireshark, y MITMf.

**Indicaciones:** Este apartado es esencial para entender las vulnerabilidades de las redes OWE y cómo pueden ser explotadas mediante técnicas avanzadas. Sigue cuidadosamente cada ejercicio y práctica las técnicas descritas en la VM del laboratorio para familiarizarte con los ataques y herramientas.

### 05.3. Ataques Wi-Fi - WEP (Wired Equivalent Privacy)

**Descripción:** Este apartado explora las vulnerabilidades y técnicas para realizar ataques en redes Wi-Fi protegidas con WEP (Wired Equivalent Privacy). Se abordan conceptos teóricos de WEP, métodos de captura y descifrado de tráfico, y técnicas para acelerar la recopilación de paquetes IV.

**Contenido:**

* **Teoría**: Introducción a WEP y sus vulnerabilidades. Explicación de conceptos clave como el Vector de Inicialización (IV), el cifrado de flujo RC4 y el Valor de Comprobación de Integridad (ICV).
* Captura y Descifrado: Uso de herramientas como airodump-ng para capturar tráfico y aircrack-ng para analizar y descifrar la clave WEP. Importancia de capturar un gran número de paquetes para aumentar la probabilidad de encontrar IVs repetidos.
* Aceleración de recopilación de paquetes IV: Técnicas como la autenticación falsa, ataques de repetición de solicitudes ARP (arpreplay) y ataques de desautenticación para generar tráfico adicional y acelerar la recolección de IVs.

**Indicaciones:** Este apartado es crucial para entender cómo explotar las vulnerabilidades de las redes WEP y aplicar técnicas avanzadas para descifrar sus claves. Sigue cuidadosamente cada ejercicio y práctica las técnicas descritas en la VM del laboratorio para familiarizarte con los ataques y herramientas.

### 05.4. Ataques Wi-Fi - PSK (Pre Shared Key)

**Descripción:** Este apartado detalla los métodos de ataque y defensa para redes Wi-Fi protegidas con Clave Precompartida (PSK). Se abordan los procesos de captura de handshakes, ataques PMKID, técnicas NoAP, vulnerabilidades WPS, ataques Evil Twin, descifrado de tráfico, y optimización de cracking de contraseñas.

**Contenido:**

* **Teoría**: Explicación del método PSK y su seguridad basada en la complejidad de la contraseña. Descripción de cómo los ataques pueden aprovechar configuraciones incorrectas de WPS, vulnerabilidades antiguas y técnicas de Evil Twin.
* Handshake: Descripción del proceso de Handshake en redes WPA/WPA2-PSK y cómo capturar y utilizar los mensajes para descifrar la contraseña. Ejemplo de ataque usando airodump-ng, aireplay-ng, aircrack-ng y hashcat.
* PMKID: Explicación del ataque PMKID que permite capturar el identificador de la clave maestra sin necesidad de un handshake completo. Ejemplo de uso con hcxdumptool y hashcat.
* NoAP: Descripción del ataque NoAP que explota los probes de los clientes para capturar handshakes. Este ataque también es conocido como half-handshake o medio-handshake Ejemplo de creación de un AP falso con hostapd y captura de handshakes.
* WPS Pin: Explicación del método WPS y su vulnerabilidad a ataques de fuerza bruta en configuraciones antiguas. Ejemplo de uso con wash y reaver.
* KRACK Attack: Descripción del ataque KRACK que explota una vulnerabilidad en el proceso de Handshake de WPA2. Referencia a herramientas y configuración para explotar esta vulnerabilidad.
* FragAttacks: Descripción de los FragAttacks que permiten fragmentar o agregar paquetes maliciosos en una red WPA/WPA2-PSK. Referencia a herramientas y configuración para explotar esta vulnerabilidad.
* Evil Twin y MiTM: Descripción del ataque Evil Twin para crear un punto de acceso falso y realizar un ataque MiTM. Ejemplo de configuración de hostapd y dnsmasq para crear el AP falso y redirigir tráfico.
* Evil Twin OPN y portal cautivo: Explicación de cómo crear un portal cautivo con un AP falso OPN para capturar credenciales de usuarios. Ejemplo de uso con airgeddon.
* Descifrado: Explicación de cómo descifrar tráfico capturado conociendo la clave PSK. Ejemplo de uso con airdecap-ng y Wireshark.
* Optimización cracking de contraseñas: Comparación de métodos de cracking de contraseñas utilizando GPU, ataques de tablas arcoíris y generación de diccionarios personalizados. Ejemplo de uso de hashcat y herramientas para generar diccionarios.

**Indicaciones:** Este apartado es fundamental para comprender y practicar técnicas avanzadas de ataque a redes Wi-Fi protegidas con PSK. Sigue cuidadosamente cada ejercicio en la VM del laboratorio para familiarizarte con las herramientas y técnicas descritas.

### 05.5. Ataques Wi-Fi - SAE (Simultaneous Authentication of Equals)

**Descripción:** Este apartado analiza los aspectos teóricos y prácticos de los ataques a redes Wi-Fi protegidas por el protocolo SAE (Simultaneous Authentication of Equals), utilizado en WPA3. Se detallan las vulnerabilidades y métodos para explotar configuraciones erróneas o realizar ataques activos.

**Contenido:**

* **Teoría:** Explicación del protocolo SAE en WPA3, su uso de criptografía de curva elíptica y su resistencia a ataques de diccionario offline.
* **Ataques a redes WPA2/WPA3 transitional:** Detalle de cómo las redes que soportan WPA2 y WPA3 simultáneamente son vulnerables debido a la posibilidad de obtener un handshake WPA2 y realizar ataques como si fuese una red WPA2.
* **Ataques de Degradación a WPA2:** Proceso para forzar a los dispositivos a conectarse usando WPA2 en lugar de WPA3, haciéndolos vulnerables a los ataques tradicionales de WPA2.
* **Ataques de Fuerza Bruta Online:** Descripción de cómo realizar ataques de fuerza bruta online utilizando herramientas específicas, destacando la importancia de políticas de contraseñas fuertes.
* **Ataques deEvil Twin:** Creación de un punto de acceso falso con el mismo ESSID y contraseña que la red objetivo para realizar ataques MiTM o de envenenamiento.
* **Ataques deEvil Twin OPN:** Uso de un punto de acceso sin contraseña para realizar ataques MiTM o establecer un portal cautivo.
* **Vulnerabilidades de Dragonblood:** Descripción de las vulnerabilidades Dragonblood, incluyendo ataques basados en caché, de canal lateral basados en tiempo y de reflexión, y medidas de mitigación como actualizaciones de firmware y configuraciones adecuadas.

**Indicaciones:** Es fundamental entender los diferentes ataques posibles contra SAE y las medidas de mitigación correspondientes. Practica los ejercicios y técnicas descritas en la VM del laboratorio para familiarizarte con los ataques y herramientas utilizados en estos contextos.

### 05.6. Ataques Wi-Fi - Recon MGT

**Descripción:** El reconocimiento previo a un ataque es una fase crucial que permite recolectar información sobre la red y los dispositivos conectados antes de lanzar un ataque más elaborado. Este apartado presenta técnicas y herramientas frecuentemente utilizadas para realizar un reconocimiento efectivo en redes MGT (WPA/WPA2/WPA3-Enterprise).

**Contenido:**

* **Análisis de Identidades de Usuarios:** Recolección de identidades (nombres de usuario) transmitidas en texto claro antes del establecimiento del túnel TLS en redes Wi-Fi MGT. Esta información se puede capturar mediante monitorización pasiva.
* **Análisis de Certificados de los AP:** Captura y análisis de los certificados enviados en texto claro durante el establecimiento del túnel TLS entre un cliente y un servidor (AP). Esta información es útil para preparar ataques como RogueAP y obtener información sobre dominios corporativos.
* **Métodos de Autenticación EAP:** Prueba de diversos métodos de autenticación EAP soportados por el AP utilizando herramientas especializadas para identificar qué métodos de autenticación están disponibles y evaluar la seguridad general de la red.

**Indicaciones:** Es fundamental realizar un reconocimiento detallado para obtener información valiosa sobre la red objetivo antes de intentar cualquier ataque. Practica las técnicas descritas en la VM del laboratorio para familiarizarte con las herramientas y métodos de reconocimiento en redes MGT.

### 05.7. Ataques Wi-Fi - MGT

**Descripción:** Este apartado explica la teoría básica sobre redes MGT y los ataques posibles sobre ellas. Se destacan los principales mecanismos de autenticación y los métodos EAP más comunes, así como los ataques específicos que se pueden realizar.

**Contenido:**

* **802.1X Mecanismos de Autenticación:** Descripción del estándar 802.1X para el control de acceso a redes, que utiliza un cliente, un autenticador y un servidor de autenticación para validar usuarios y dispositivos en redes cableadas e inalámbricas.
* **EAP (Protocolo de Autenticación Extensible):** Explicación de varios métodos de autenticación EAP utilizados en redes MGT, incluyendo PAP, CHAP, MSCHAPv2, GTC y autenticación por certificado de cliente.
* **EAP Handshakes:** Descripción de los diferentes tipos de handshakes EAP, como EAP-MD5, LEAP, EAP-TLS, EAP-FAST, EAP-TTLS y PEAP, y sus características.
* **Resumen de Métodos EAP:** Comparación de diferentes métodos EAP, destacando sus características, ventajas y vulnerabilidades.
* **AtaqueRogue AP:** Proceso para crear un punto de acceso no autorizado (Rogue AP) y capturar credenciales mediante el uso de herramientas como eaphammer y hostapd-mana.
* **Ataque de Reenvío (RelayAttack):** Explicación del ataque de reenvío, que permite al atacante reenviar credenciales desde un cliente legítimo a un AP legítimo para obtener acceso a la red.
* **Fuerza Bruta en Redes MGT:** Uso de herramientas como air-hammer para realizar ataques de fuerza bruta sobre credenciales en redes MGT.
* **Password Spraying en Redes MGT:** Técnica de password spraying para probar una contraseña en múltiples cuentas de usuario, minimizando el riesgo de bloqueo de cuentas.
* **Phishing+RogueAP (Captive-Portal):** Combinación de un ataque Rogue AP con un portal cautivo de phishing para capturar credenciales de usuario.
* **Crear AP con CA Robada:** Uso de una autoridad de certificación (CA) robada para crear un AP que utilice certificados legítimos y engañe a los clientes para que se conecten.
* **Generar Certificado de Cliente con CA Robada:** Proceso para emitir un certificado de cliente usando una CA robada y acceder a una red protegida.

**Indicaciones:** Comprender y practicar estos ataques es esencial para evaluar la seguridad de las redes MGT y desarrollar contramedidas efectivas. Sigue cuidadosamente cada ejercicio y utiliza las herramientas descritas en la VM del laboratorio para familiarizarte con los diferentes métodos y técnicas de ataque.

### 05.8. Sistema de Detección de Intrusiones Inalámbricas (WIDS)

**Descripción:** Este apartado explica qué es un Sistema de Detección de Intrusiones Inalámbricas (WIDS), cómo funciona y su importancia en la seguridad de redes inalámbricas. Se describen las funcionalidades clave de WIDS y su papel en mejorar la seguridad, cumplimiento y visibilidad de la red.

**Contenido:**

* **¿Qué es WIDS?**: Explicación del propósito de un Sistema de Detección de Intrusiones Inalámbricas, diseñado para monitorear, detectar y alertar sobre posibles amenazas o vulnerabilidades en redes Wi-Fi.
* **¿Cómo Funciona WIDS?**: Descripción del funcionamiento de WIDS a través del despliegue de sensores que analizan el tráfico inalámbrico para detectar actividades sospechosas o comportamientos anómalos.
* **Detección de Dispositivos Clandestinos**: Identificación de puntos de acceso y dispositivos no autorizados en la red.
* **Detección de Ataques de Suplantación**: Detección de intentos de suplantar dispositivos legítimos o APs.
* **Monitoreo del Espectro Inalámbrico**: Escaneo en busca de actividades inalámbricas no autorizadas.
* **Aplicación de Políticas**: Aseguramiento del cumplimiento de las políticas de seguridad de la organización.
* **Alertas y Reportes**: Notificación de amenazas y provisión de reportes detallados para análisis posterior.
* **Importancia de WIDS**: Beneficios de WIDS en mejorar la seguridad, cumplir con regulaciones, ofrecer visibilidad de la red y permitir una gestión proactiva de amenazas.

**Indicaciones:** La implementación de un WIDS es crucial para mantener la seguridad de las redes inalámbricas. Practica el uso de herramientas como Nzyme en la VM del laboratorio para monitorear y recibir alertas sobre posibles ataques, asegurándote de familiarizarte con las funcionalidades y capacidades del sistema.

### 06. Ataques Avanzados e Indirectos en Wi-Fi Enterprise

**Descripción:** Este apartado cubre varios ataques avanzados dirigidos a redes Wi-Fi empresariales, utilizando vectores de acceso indirectos. Se enfoca en atacar a los clientes de las redes o a otras redes relacionadas y en técnicas modernas de ataques que se actualizarán regularmente.

**Contenido:**

* Ataques Indirectos en Wi-Fi Enterprise: Uso de Rogue APs que imitan redes gratuitas o domésticas, así como técnicas de ESSID Stripping para capturar credenciales y realizar ataques adicionales.
* 6 GHz: Configuración y monitorización de redes en la banda de 6 GHz, utilizando WPA3 SAE y airodump-ng para escanear el espectro.

**Indicaciones:** Estos ataques avanzados requieren un profundo conocimiento de las redes Wi-Fi y sus mecanismos de seguridad. Practica estos ataques en un entorno controlado utilizando las herramientas descritas para familiarizarte con los métodos y técnicas utilizados en escenarios de Red Team y Pentesting.

### 07. Experiencia en el mundo real

**Descripción**: Este capítulo se centra en la elección y configuración de adaptadores Wi-Fi y controladores para 2024, destacando modelos recomendados que soportan bandas de 2.4 GHz, 5 GHz y 6 GHz, y cubriendo su instalación en Kali Linux y técnicas de wardriving.

**Contenido**:

* Adaptadores Wi-Fi y Controladores: Revisión de adaptadores como Alfa AWUS036AXML y Panda PAU09 N600, con recomendaciones según precio y prestaciones.
* Antena Direccional: Uso de antenas direccionales como Alfa APA-M25 para mejorar el alcance de la señal.
* Instalación de Controladores en Kali Linux: Guía para instalar controladores de adaptadores Wi-Fi en Kali Linux.
* Uso de airodump-ng para Escanear: Configuración del modo monitor y uso de airodump-ng para escanear diversas bandas.
* GPS para Wardriving: Implementación de dispositivos GPS para mejorar la detección de redes durante el wardriving.
* Lista de No Recomendados: Adaptadores Wi-Fi no recomendados por su inestabilidad o limitaciones.
* Consejos útiles: Recomendaciones prácticas para la gestión de certificados y uso adecuado del modo monitor.

**Indicaciones**: Sigue las guías de instalación y configuración para evitar problemas y maximizar la efectividad de las técnicas descritas, mejorando tus habilidades en el análisis de redes Wi-Fi.

### 08. Securización de redes Wi-Fi

**Descripción**: Este capítulo proporciona estrategias para proteger redes Wi-Fi, como el uso de contraseñas fuertes, aislamiento de clientes, y configuración de redes de invitados. También se ofrecen recomendaciones específicas para diferentes tipos de redes.

**Contenido**:

* Medidas Generales: Utiliza contraseñas fuertes, aísla clientes, configura redes de invitados, cambia el SSID predeterminado, habilita el filtrado de direcciones MAC, ajusta la potencia de transmisión, monitorea la red regularmente y habilita MFP.
* Redes OPN (Open): Migra de OPN a OWE (WPA3-OWE) para cifrar datos en redes abiertas.
* Redes WEP (Wired Equivalent Privacy): Desactiva WEP y actualiza a WPA2 o WPA3 por mayor seguridad.
* Redes PSK (Pre-Shared Key): Desactiva WPS, migra a WPA3-SAE y rota claves regularmente.
* Redes MGT Enterprise: Usa certificados digitales, identidades anónimas, verifica certificados en clientes y segmenta la red.

**Indicaciones**: Implementa estas medidas y adaptaciones específicas para fortalecer la seguridad de tu red Wi-Fi, protegiendo los datos y mitigando vulnerabilidades.
