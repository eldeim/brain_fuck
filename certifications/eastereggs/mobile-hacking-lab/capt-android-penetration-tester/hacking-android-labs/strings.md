---
description: https://www.mobilehackinglab.com/course/lab-strings
---

# Strings

Outline\
The challenge will give you a clear idea of how intents and intent filters work on android also you will get a hands-on experience using Frida APIs.

Objective\
Exploit the application to obtain the flag.

Skills Required\
Understanding of Android app components.\
Familiarity with Frida\
Android reverse engineering.

***

## Reconocimiento

Al abrir la app solo vemos "Hello from C++". Sin botones, sin interacción, sin nada más visible.

El siguiente paso es analizar el APK estáticamente con jadx-gui.

<figure><img src="../../../../../.gitbook/assets/image (1474).png" alt=""><figcaption></figcaption></figure>

## Análisis Estático con JADX

> AndroidManifest.xml

Lo primero que miras es el Manifest. Encuentras dos Activities exportadas (exportadas = accesibles desde fuera de la app, cualquier app o comando puede lanzarlas):

com.mobilehackinglab.challenge.MainActivity → exportada com.mobilehackinglab.challenge.Activity2 → exportada + Deep Link

<figure><img src="../../../../../.gitbook/assets/image (1478).png" alt=""><figcaption></figcaption></figure>

Activity2 tiene configurado un intent-filter con:

* action: android.intent.action.VIEW
* scheme: mhl
* host: labs

Esto significa que cualquier URL con formato mhl://labs/... abrirá Activity2.

<figure><img src="../../../../../.gitbook/assets/image (1484).png" alt=""><figcaption></figcaption></figure>

### MainActivity

Lo importante es que dentro de <mark style="background-color:$warning;">MainActivity</mark> hay un método llamado KLOW que nunca se invoca en ningún sitio del código. Está definido pero nadie lo llama. Su función es:

<figure><img src="../../../../../.gitbook/assets/image (1485).png" alt=""><figcaption></figcaption></figure>

### Activity2

Aquí está toda la lógica del reto. Su onCreate hace una cadena de validaciones. Si alguna falla, la app se cierra forzosamente:

Validación 1 — Fecha en SharedPreferences:

```java
String stored = getSharedPreferences("DAD4", 0).getString("UUU0133", "");
if (!stored.equals(cd())) finish(); // cd() = fecha hoy dd/MM/yyyy
```

> Busca en DAD4.xml la clave UUU0133 y compara con la fecha de hoy. Si no existe o no coincide → cierra.

Validación 2 — Intent action:

```java
if (!getIntent().getAction().equals("android.intent.action.VIEW")) finish();
```

> Verifica que fue lanzada con el intent correcto (el que produce un Deep Link).

Validación 3 — Deep Link válido:

```java
Uri uri = getIntent().getData();
if (!uri.getScheme().equals("mhl") || !uri.getHost().equals("labs")) finish();
```

Validación 4 — Secreto en Base64:

```java
String lastSegment = uri.getLastPathSegment(); // coge último segmento del link
String decoded = new String(Base64.decode(lastSegment, 0)); // decodifica Base64
String decrypted = decrypt("bqGrDKdQ8zo26HflRsGvVA=="); // descifra AES
if (!decoded.equals(decrypted)) finish();
```

> Coge el último segmento del Deep Link, lo decodifica de Base64, y lo compara con un valor descifrado AES. Si no coinciden → cierra.

Si todo es correcto:

```java
System.loadLibrary("flag");
String flag = getflag(); // función nativa de libflag.so
Toast.makeText(this, flag, Toast.LENGTH_LONG).show();
```

> Carga libflag.so, obtiene el flag y lo muestra como Toast — pero en la práctica el Toast no es visible.

## Descifrar el Secreto

Del código de Activity2 extraes:

* Ciphertext: bqGrDKdQ8zo26HflRsGvVA==
* Algoritmo: AES/CBC/PKCS5Padding
* Key: your\_secret\_key\_1234567890123456
* IV: lo encuentras buscando fixedIV en jadx

<figure><img src="../../../../../.gitbook/assets/image (1476).png" alt=""><figcaption></figcaption></figure>

Lo descifras en CyberChef con AES Decrypt → resultado: mhl\_secret\_1337

<figure><img src="../../../../../.gitbook/assets/image (1480).png" alt=""><figcaption></figcaption></figure>

### Construir el Deep Link

Activity2 espera el secreto en Base64 dentro del link (porque ella misma lo decodifica para comparar). Entonces tienes que codificarlo:

<figure><img src="../../../../../.gitbook/assets/image (1481).png" alt=""><figcaption></figcaption></figure>

> mhl\_secret\_1337 → Base64 → bWhsX3NlY3JldF8xMzM3

Deep Link final:&#x20;mhl://labs/bWhsX3NlY3JldF8xMzM3

## Explotación Completa

Paso 1 — Lanzar la app con Frida

```
frida -U -f com.mobilehackinglab.challenge
```

Paso 2 — Conectar Objection (otra terminal)

```
objection -g com.mobilehackinglab.challenge explore
```

Paso 3 — Invocar KLOW manualmente

> KLOW nunca se llama solo, hay que forzarlo desde el heap de la app:>

```
android heap search instances com.mobilehackinglab.challenge.MainActivity
```

Te devuelve un hash (ej: 207990455). Úsalo para ejecutar KLOW:

```
android heap execute 207990455 KLOW
```

Esto crea DAD4.xml con UUU0133 = fecha de hoy → satisface la validación de Activity2.

Paso 4 — Disparar el Deep Link

```
adb shell am start -a android.intent.action.VIEW -d "mhl://labs/bWhsX3NlY3JldF8xMzM3" -n com.mobilehackinglab.challenge/.Activity2
```

> Activity2 valida todo, carga libflag.so y pone el flag en memoria. Aparece un Toast pero no se ve

Paso 5 — Memory Scan

Dentro de objection:

```
memory search "MHL{" --string
```
