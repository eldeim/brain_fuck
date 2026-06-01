# Otras funcionalidades MGT

## Explicación de Fast Roaming

El Wi-Fi Fast Roaming, común en redes empresariales, permite que un cliente cambie rápidamente entre APs sin realizar una reauthentication completa cada vez. Esto es crítico en entornos con alta movilidad, como VoIP, videollamadas o automatización industrial. Protocolos como 802.11r facilitan esto al habilitar el key caching y la pre-authentication.

En lugar de realizar un 4-way handshake completo con cada nuevo AP, el cliente reutiliza credenciales previas mediante PMKID caching o FT.

### Implicaciones de seguridad

Depender de claves almacenadas reduce la sobrecarga de autenticación pero abre vectores de ataque:

* **Key Reuse Risk:** Cached keys (como PMK) pueden ser interceptadas si no se protegen adecuadamente.
* **FT Handshake Attacks:** Un atacante podría falsificar FT requests para hacerse pasar por un AP o cliente.
* **Weak PMKID Protection:** Si el PMKID se expone, permite ataques de diccionario offline.
* **Lack of Full Handshake:** Sin un handshake completo, algunos pasos de verificación se omiten, facilitando ataques MITM.

### Fast Roaming en hostapd (802.11r)

Configura 802.11r para habilitar la pre-authentication y el key caching, reduciendo drásticamente la association latency cuando los clientes hacen roam entre APs:

```
ieee80211r=1
mobility_domain=4f57     # Hex-encoded domain ID (two octets)
ft_over_ds=1             # Use Distribution System for FT
ft_psk_generate_local=1  # Generate local PSK for FT
r0_key_lifetime=10000    # Lifetime of R0 key in seconds
r1_key_holder=1
disable_pmksa_caching=0  # Keep PMKSA cache enabled
```

Equilibra rendimiento y seguridad: usa WPA3 donde sea posible y monitoriza rogue APs.

## Band Steering

El Band Steering anima a los clientes compatibles a conectarse a APs de 5 GHz en lugar de 2.4 GHz, mejorando el throughput y reduciendo la co-channel interference. El controlador retiene temporalmente las respuestas en 2.4 GHz para "empujar" al cliente a 5 GHz.

#### Beneficios y consideraciones

* **Reduced Congestion:** Alivia tráfico de la banda de 2.4 GHz.
* **Improved Performance:** Más canales no solapados y mayores data rates en 5 GHz.
* **Compatibility Issues:** Algunos dispositivos legacy pueden ignorar el steering y volver a 2.4 GHz.
* **Complexity:** Ajustar umbrales de RSSI para evitar client flapping.

Ajusta los umbrales con cuidado: si son muy agresivos, los clientes pueden perder conexión antes de asociarse al AP de 5 GHz.

## Airtime Fairness

La Airtime Fairness evita que clientes lentos monopolizan el medio RF. El AP asigna a cada cliente una "porción" basada en tiempo, asegurando que los high-speed clients no se vean ralentizados por slow clients.

#### Cuándo habilitar

* **Mixed Client Speeds:** Dispositivos 802.11b/g y 802.11n/ac/ax simultáneos.
* **High-Density Deployments:** Aulas, auditorios o conferencias con muchos usuarios.
* **QoS Sensitivity:** Entornos con VoIP o streaming en real time.

Ten en cuenta que, aunque mejora el rendimiento global, puede aumentar la latency para slow clients.

## Load Balancing

El Load Balancing reparte clientes entre APs vecinos para evitar que uno solo se sobrecargue. Basado en thresholds de client count o channel utilization, el controlador redirige nuevas associations a APs menos saturados.

### Detalles de implementación

* **Threshold-Based:** Define el máximo de associations simultáneas por AP.
* **Utilization-Based:** Mide la channel utilization para balancear no solo el client count sino el real tráfico.
* **Sticky Clients:** Desactiva temporalmente el balancing para critical devices (p.ej., security cameras).

Configura perfiles de sticky clients para evitar redirecciones repetidas y packet loss.

### PMKSA Caching & Reassociation

Habilitar PMKSA Caching permite a los clientes reutilizar sus PMKs al moverse entre APs, eliminando la necesidad de un EAP-TLS handshake completo en cada roam; no se envían intercambios adicionales de identidad o certificados tras la conexión inicial, aunque el 4-way handshake (o FT handshake) sigue ocurriendo y debe protegerse con una strong key.

### Configuración en hostapd

```
pmeksa=1                # 0 = disabled, 1 = enabled
max_num_sta=200          # Max PMKSA cache entries
reassociation_deadline=1000  # Time ms to complete reassociation
```

## Protected Management Frames (MFP)

Exigir 802.11w aplica protección criptográfica en management frames como deauthentication y disassociation, bloqueando ataques de spoofing e injection, y mejorando la resistencia ante disconnect-based DoS y MITM; los clientes sin soporte para 802.11w no podrán conectarse hasta su actualización o reemplazo.

### Configuración en hostapd

```
ieee80211w=2    # 1 = optional, 2 = required
pmk_r1_push=1   # Push FT R1 key to AP for faster handoff
```

## Segmentación VLAN

Dynamic VLAN assignment aísla el tráfico de cada cliente en la Data Link layer (Layer 2) mapeándolo en VLANs separadas, conteniendo cualquier ataque broadcast o compromiso dentro de esa VLAN y previniendo lateral movement, aunque una configuración incorrecta de VLANs o bridges puede exponer redes sensibles si no se gestiona con cuidado.

### Configuración en hostapd

```
dynamic_vlan=1
vlan_file=/etc/hostapd.vlan
vlan_tagged_interface=wlan0
vlan_bridge=br_mgt
```

## Protected Access Credential (PAC)

En EAP-FAST (RFC 4851), un Protected Access Credential (PAC) es una credencial opaca que acelera el establecimiento de túneles TLS tras el primer provisioning. Al usar un PAC, no es necesario repetir el TLS handshake completo en cada autenticación subsecuente.

### PAC Provisioning

1. Durante la primera autenticación de EAP-FAST, el servidor EAP entrega el PAC al supplicant junto con metadata como el server ID y el PAC validity period.
2. La entrega ocurre dentro de un frame EAP-FAST-Response, encapsulado en un frame EAPoL (management).
3. El PAC contiene derived keys y tunnel parameters, cifrados y autenticados con un message authentication code (MAC).

### PAC Format and Storage

* **Binary structure:** \[version | length | encrypted\_data | MAC], donde encrypted\_data incluye session keys y parámetros para la PRF (pseudo-random function).
* **Server storage:** encrypted file (p. ej., /etc/hostapd/fast\_pac\_list).
* **Client storage:** operating-system keystore o protected on-disk file.
* **Default expiration:** los PAC expiran por defecto tras 604800 segundos (7 días) en hostapd; esto puede ajustarse con pac\_key\_lifetime (hard limit) y pac\_key\_refresh\_time (soft refresh threshold).

### Subsequent Authentications with PAC

* El supplicant incluye el campo PAC-opaque en su EAP-FAST-Request, omitiendo así el TLS handshake completo.
* Las session keys se derivan mediante una optimized PRF operation sobre el PAC, reduciendo latencia y uso de CPU.
* Solo se intercambian mensajes de EAP-FAST-Protect (PAC verification y tunnel context); no aparecen certificados ni ClientHello.

### Configuring PAC in hostapd

```
# hostapd.conf
eap_server=1
eap_user_file=/etc/hostapd/fast_user_file
fast_pac_file=/etc/hostapd/fast_pac_list
fast_provisioning=1
fast_prov_batch=1
logger_syslog_level=3
# Optional PAC lifetime settings:
#pac_key_lifetime=604800        # hard limit (7 days)
#pac_key_refresh_time=86400     # soft refresh threshold (1 day)
```

El archivo fast\_user\_file debe contener líneas con formato \<Identity> \<Password> \<PAC-Alias>. Por ejemplo:

```
# identity   password     pac-alias
user1        secretpass   user1-pac
```

Si fast\_pac\_list no existe, se crea automáticamente en el primer provisioning. Alternativamente, puede precargarse con fast\_prov\_list o mediante hostapd\_cli.

#### Capturing Identity and Handshake after a Deauthentication using PAC&#x20;

* **Injecting deauth:** aireplay-ng --deauth 1 -a \<BSSID\_AP> -c \<MAC\_client> wlan0
*   **Collecting Identity:**&#x43;opiar

    ```
    eap.code == 1 || eap.code == 2
    ```

    Los frames EAP-Request/Identity y EAP-Response/Identity se capturan en cleartext.
* **Full TLS handshake:** Ocurre solo si el PAC está absent o ha expired. Se observan ClientHello, ServerHello y el server Certificate; no se usa client certificate.
* **Fast reauth with valid PAC:** Solo se intercambian mensajes de PAC verification y key derivation; no aparecen certificados ni ClientHello.
