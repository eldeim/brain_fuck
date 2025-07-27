# Optimización cracking de contraseñas

### Cracking con GPU

El cracking de contraseñas utilizando GPUs (Unidades de Procesamiento Gráfico) es mucho más eficiente que métodos tradicionales que usan CPUs o tablas aircoiris (rainbow tables). Las GPUs están diseñadas para manejar operaciones paralelas masivas, lo que hace que pueden realizar una gran cantidad de cálculos hash simultáneamente.

#### Por qué es más eficiente el cracking con GPU utilizando hashcat:

* **Paralelización masiva**: Las GPUs tienen miles de núcleos que pueden procesar hashes en paralelo, aumentando significativamente la velocidad.
* **Velocidad de cálculo**: Las GPUs son mucho más rápidas en cálculos repetitivos, esenciales para el cracking de contraseñas.
* **Optimización de software**: Herramientas como hashcat están específicamente diseñadas para aprovechar la arquitectura de las GPUs.
* **Coste-eficiencia**: Aunque las GPUs pueden ser costosas, ofrecen una mejor relación costo/eficiencia en términos de hashes por segundo comparado con las CPUs.
* **Escalabilidad**: Hashcat puede usar múltiples GPUs en un solo sistema, aumentando la capacidad de cracking de manera eficiente.
* **Productividad mejorada**: El uso de GPUs reduce drásticamente el tiempo necesario para descifrar contraseñas, permitiendo evaluaciones más rápidas.

Comparativamente, las tablas rainbow son menos prácticas debido a su enorme requerimiento de espacio de almacenamiento y su falta de flexibilidad y escalabilidad para nuevas configuraciones o algoritmos hash. En conclusión, **crackear contraseñas con GPU** utilizando herramientas como hashcat es más rápido, práctico y escalable que otros métodos como el uso de CPUs o tablas rainbow.

Como alternativa a hashcat también se puede utilizar Pyrit ([https://github.com/JPaulMora/Pyrit](https://github.com/JPaulMora/Pyrit)), que es capaz de crackear utilizando tanto CPU como GPU.

### Ataques de Tablas Arcoíris (Rainbow Tables)

Los ataques de tablas arcoíris utilizan tablas precomputadas que contienen los valores hash de posibles contraseñas, reduciendo drásticamente el tiempo necesario para romper un PSK después de capturar el handshake de la red.

* **Explicación Técnica**: En lugar de calcular los valores hash en tiempo real (como con la fuerza bruta), las tablas arcoíris almacenan valores hash precomputados para combinaciones de SSIDs y posibles contraseñas. Cuando se captura un handshake, un atacante puede rápidamente comparar el hash del handshake capturado contra las entradas de la tabla.
* **Herramientas**: rainbowcrack y otras herramientas especializadas se utilizan para generar y utilizar tablas arcoíris para romper redes Wi-Fi.

### Generación de Diccionarios Personalizados

Crear diccionarios personalizados adaptados a objetivos específicos puede mejorar significativamente la eficiencia del descifrado de PSK. Herramientas como CeWL ayudan a generar listas de palabras contextuales que aumentan las posibilidades de descubrir contraseñas legítimas.

```
cewl -d 2 -m 5 -w docswords.txt https://example.com
```

* -d 2 Esta opción especifica la profundidad de rastreo. En este caso, 2 indica que la herramienta seguirá los enlaces en el sitio web hasta dos niveles de profundidad.
* -m 5 Esta opción establece la longitud mínima de las palabras que se incluirán en la lista. Aquí, 5 significa que solo se recopilarán palabras que tengan al menos cinco caracteres.
* -w docswords.txt Esta opción indica el nombre del archivo donde se guardarán las palabras recopiladas. En este caso, las palabras se guardarán en un archivo llamado docswords.txt.
* https://example.com Esta es la URL del sitio web del cual CeWL extraerá las palabras.
