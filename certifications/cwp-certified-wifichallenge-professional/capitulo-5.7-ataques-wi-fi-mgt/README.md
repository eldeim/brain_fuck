# Capítulo 5.7: Ataques Wi-Fi - MGT

## 802.1X Mecanismos de Autenticación

802.1X es un estándar de IEEE para controlar el acceso a redes. Utiliza tres componentes principales: el suplicante (cliente), el autenticador (punto de acceso) y el servidor de autenticación (comúnmente un servidor RADIUS). El 802.1X proporciona mecanismos robustos para la autenticación de usuarios y dispositivos en redes cableadas e inalámbricas. Esto permite a las redes MGT utilizar una autenticación única para cada cliente, permitiendo a cada usuario acceder con su usuario y contraseña o con su certificado propio.

En el siguiente diagrama se puede ver las partes de una conexión MGT.

<figure><img src="../../../.gitbook/assets/image (22) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Cliente (Supplicant): Cliente que intenta conectarse a la red. En la ilustración, se representa como un ordenador junto a una figura humana.
* autenticador (Authenticator): El autenticador es el dispositivo de red intermedio (en el caso de redes Wi-Fi es el punto de acceso) que recibe las solicitudes de autenticación del cliente. Está representado por un dispositivo con flechas rojas que indican el flujo de tráfico.
* Servidor de Autenticación (Authentication Server): El servidor de autenticación valida las credenciales del cliente. Está representado por un dispositivo al que el autenticador se conecta para verificar la identidad del suplicante. Este servidor esta normalmente conectado a un Directorio Activo para que los usuarios tengan SSO (Single Sign-on), lo que permite a los clientes acceder a la red Wi-Fi con sus credenciales de dominio, igual que a su equipo corporativo y a su mail.

**Proceso de Autenticación:**

* **EAPOL (EAP over LAN)**: El Protocolo de Autenticación Extensible sobre LAN (EAPOL) se usa para la comunicación inicial entre el cliente y el autenticador.
* **RADIUS/Diameter**: Después de que el autenticador recibe las credenciales del cliente, las reenvía al servidor de autenticación usando RADIUS o Diameter (protocolos de autenticación y autorización).
* **Acceso a la Red**: Si el servidor de autenticación valida exitosamente las credenciales del cliente, el autenticador le permitirá el acceso a la red, incluyendo recursos de LAN e internet, como se indica en el diagrama con la nube.

### EAP

EAP, o Protocolo de Autenticación Extensible, es un marco de autenticación que soporta múltiples métodos de autenticación. A continuación, se presentan algunos de los métodos de autenticación más comunes utilizados en redes.

* PA&#x50;**, o Protocolo de Autenticación por Contraseña**, es un método simple de autenticación de dos vías que envía contraseñas en texto claro. Es uno de los métodos menos seguros debido a su falta de cifrado.
* CHA&#x50;**, o Protocolo de Autenticación de Challenge-Response**, es más seguro que PAP porque utiliza un proceso de desafío y respuesta para autenticar al usuario. Las contraseñas no se transmiten en texto claro.
* MSCHAPv2 es una versión mejorada del Protocolo de Autenticación por Desafío de Microsoft. Proporciona autenticación mutua entre el cliente y el servidor y utiliza un algoritmo más seguro para el cifrado de contraseñas.
* GT&#x43;**, o Texto Generado por el Usuario**, es un método simple de autenticación que permite al cliente enviar un nombre de usuario y una contraseña en texto claro.
* Certificado de cliente, es el método más seguro. En este caso, tanto el servidor como el cliente utilizan certificados digitales para autenticarse entre sí. Este método es el único disponible en EAP-TLS y encuentra en EAP-TTLS y algunas variaciones de PEAP.

### EAP Handshakes

Los Handshake de EAP son procesos de autenticación que pueden dividirse en dos fases. A continuación, se presentan varios tipos comunes de Handshake EAP:

* EAP-MD5 es uno de los métodos EAP más simples. Se basa en un hash MD5 para autenticar a los usuarios, pero es vulnerable a varios ataques. **(**&#x4F;BSOLET&#x4F;**)**
* LEAP, o Protocolo de Autenticación Ligera Extensible, fue desarrollado por Cisco. Utiliza una combinación de contraseñas y cifra las credenciales, pero ha sido vulnerable a ataques de diccionario. **(**&#x4F;BSOLET&#x4F;**)**
* EAP-TLS utiliza certificados de cliente y servidor para autenticar a ambas partes de forma segura. Es uno de los métodos EAP más robustos y seguros, pero también uno de los más complejos de implementar debido a la necesidad de una infraestructura de PKI.
* EAP-FAST, desarrollado por Cisco, es similar a EAP-TLS pero utiliza un PAC (Paquete de Capacidad) en lugar de certificados para la autenticación. Esto reduce la complejidad de implementación mientras mantiene un buen nivel de seguridad.
* EAP-TTLS crea un canal seguro similar a EAP-TLS pero permite el uso de métodos de autenticación más simples dentro del túnel seguro, como PAP, CHAP, o MSCHAPv2 además de TLS.
* PEAP, o Protocolo de Autenticación Extensible Protegida, es similar a EAP-TTLS y permite el uso de métodos de autenticación más simples dentro de un túnel TLS seguro.

### Resumen

Para simplificar, es posible diferenciar dos grandes tipos de métodos EAP:

* EAP con Autenticación mediante Certificado de Cliente (por ejemplo, EAP-TLS, PEAPv0 (EAP-TLS)):

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* EAP con Autenticación por Credenciales (por ejemplo, LEAP, PEAPv0 (MSCHAPv2), EAP-TTLS (MSCHAPv2), etc.). Los procesos de autenticación EAP se pueden desglosar en 2 fases:
  * Fase 1: El servidor se autentica utilizando un certificado (o un PAC en EAP-FAST) y se establece un túnel TLS seguro.
  * Fase 2: El cliente se autentica utilizando un método de autenticación basado en credenciales como MSCHAPv2, CHAP, PAP, GTC, etc.

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Comparación de Métodos EAP <a href="#comparacion-de-metodos-eap" id="comparacion-de-metodos-eap"></a>

| Característica                                            | EAP-TLS	                                    | EAP-TTLS	                                   | PEAPv0 (EAP-MSCHAPv2)	                      | PEAPv0 (EAP-TLS)	 | PEAPv1 (EAP-GTC)                            |
| --------------------------------------------------------- | ------------------------------------------- | ------------------------------------------- | ------------------------------------------- | ----------------- | ------------------------------------------- |
| **Certificado de Servidor**                               | Sí                                          | Sí                                          | Sí                                          | Sí                | Sí                                          |
| **Certificado de Cliente**                                | Sí, obligatorio (también soporta smartcard) | Opcional                                    | No                                          | Sí                | Opcional                                    |
| **Autenticación del Cliente Soportada**                   | Via Certif./Smartcard                       | PAP, CHAP, MSCHAPv2, GTC, Certif.           | MSCHAPv2                                    | Certif.           | GTC                                         |
| **Autenticación Mutua**                                   | Sí                                          | Sí                                          | Sí                                          | Sí                | Sí                                          |
| **Protección de Identidad del Usuario**                   | Sí (anónima)                                | Sí (cifrado TLS)                            | Sí (cifrado TLS)                            | Sí (cifrado TLS)  | Sí (cifrado TLS)                            |
| **Autenticación del Cliente en texto claro**              | No                                          | No                                          | No                                          | No                | Sí, - Permite contraseñas en texto claro    |
| **Autenticación de Cliente Vulnerable a Ataques Offline** | No                                          | Herramienta Asleap (para MSCHAPv2)          | Herramienta Asleap                          | No                | Texto claro (dentro del túnel TLS)          |
| **AtaqueEvil Twin Posible?**                              | No                                          | Sí si no se valida certificado del servidor | Sí si no se valida certificado del servidor | No                | Sí si no se valida certificado del servidor |

