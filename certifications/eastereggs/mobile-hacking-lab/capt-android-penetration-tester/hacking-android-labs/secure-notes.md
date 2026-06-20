# Secure Notes

#### Objective <a href="#el_1700338310850_408" id="el_1700338310850_408"></a>

* Retrieve a PIN code from a secured content provider in an Android application.

#### Hints <a href="#el_1700338310831_386" id="el_1700338310831_386"></a>

* Investigate the use of ContentResolver.query() for data interaction.
* Examine the Android manifest for any improperly exported content providers.
* Employ adb for enhanced interaction with the application.

***

## Analice - Android Manifest

* `AndroidManifest.xml`: to identify exported components and permissions.
* `MainActivity`: to observe how the app queries the provider and displays results.
* `SecretDataProvider`: to understand how the provider validates input, derives the key and decrypts the secret.
* `assets/config.properties`: where the encrypted secret and crypto parameters are stored.

We can observe difrerents exported recourses -->

<figure><img src="../../../../../.gitbook/assets/image (1504).png" alt=""><figcaption></figcaption></figure>

Jum...

<figure><img src="../../../../../.gitbook/assets/image (1505).png" alt=""><figcaption></figcaption></figure>

***

### Main Activity

<figure><img src="../../../../../.gitbook/assets/image (1506).png" alt=""><figcaption></figcaption></figure>

We can see a PIN code and the Error message -->

<figure><img src="../../../../../.gitbook/assets/image (1507).png" alt=""><figcaption></figcaption></figure>

In mainActivity see that... well well, seach by Secret -->

<figure><img src="../../../../../.gitbook/assets/image (1508).png" alt=""><figcaption></figcaption></figure>

We can found the code of decription pin... so... we can read here something like: 1. check the config.properties file loaded in local, 2. try to decode it pin 3. test

Lest try to read it local file in the app -->

<figure><img src="../../../../../.gitbook/assets/image (1509).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (1510).png" alt=""><figcaption></figcaption></figure>

```
encryptedSecret=bTjBHijMAVQX+CoyFbDPJXRUSHcTyzGaie3OgVqvK5w=
salt=m2UvPXkvte7fygEeMr0WUg==
iv=L15Je6YfY5owgIckR9R3DQ==
iterationCount=10000
```

Well... Now we Know two thinks!

### decryptSecret()

The decryptSecret() method decrypts the stored Base64-encoded secret using the key derived from the user PIN. This confirms that gaining access to the correct PIN directly reveals the protected flag or secret.

### generateKeyFromPin()

Upon reviewing generateKeyFromPin(), I identified that it uses the provided PIN along with the defined salt and iteration count to generate a cryptographic key via PBKDF2. This implementation ties the strength of the encryption entirely to the PIN’s complexity, further justifying the feasibility of brute-forcing weak PINs.

<figure><img src="../../../../../.gitbook/assets/image (1511).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (1512).png" alt=""><figcaption></figcaption></figure>

***

## Create a bruteforce code

Know it four parameters... We can make a burte force python code -->

```python
import base64
import hashlib
from Crypto.Cipher import AES

## Parameters taken from assets/config.properties
ENCRYPTED = base64.b64decode("bTjBHijMAVQX+CoyFbDPJXRUSHcTyzGaie3OgVqvK5w=")
SALT      = base64.b64decode("m2UvPXkvte7fygEeMr0WUg==")
IV        = base64.b64decode("L15Je6YfY5owgIckR9R3DQ==")
ITER      = 10000          # iterationCount
KEY_LEN   = 32             # 32 bytes = AES-256


def decrypt(pin: str):

    """Derive the AES key from the PIN and try to decrypt the secret.
    Return the plaintext only if the padding is valid AND the result is
    printable ASCII (avoids the ~1/256 random valid-padding false positives)."""
    
    key = hashlib.pbkdf2_hmac("sha1", pin.encode(), SALT, ITER, dklen=KEY_LEN)
    pt = AES.new(key, AES.MODE_CBC, IV).decrypt(ENCRYPTED)
    pad = pt[-1]                              # last byte = padding length
    if not (1 <= pad <= 16 and pt[-pad:] == bytes([pad]) * pad):
        return None                           # invalid PKCS7 padding
    text = pt[:-pad]
    if all(32 <= b < 127 for b in text):      # printable ASCII -> real secret
        return text
    return None


def main():
    # Try every 4-digit PIN, "0000" to "9999" (zero-padded).
    for n in range(10000):
        pin = f"{n:04d}"
        secret = decrypt(pin)
        if secret:
            print(f"[+] PIN found: {pin}")
            print(f"[+] Secret:    {secret.decode(errors='replace')}")
            return
    print("[-] PIN not found in the 4-digit range.")


if __name__ == "__main__":
    main()
```

RESULT! -->

<figure><img src="../../../../../.gitbook/assets/image (1513).png" alt=""><figcaption></figcaption></figure>

```
[+] PIN found: 2580
[+] Secret: CTF{D1d_y0u_gu3ss_1t!1?}
```

***

### Resume

```python
  encryptedSecret=bTjBHijMAVQX+...   # ← el flag, pero CIFRADO
  salt=m2UvPXkvte7fygEeMr0WUg==      # parámetro público de PBKDF2
  iv=L15Je6YfY5owgIckR9R3DQ==        # parámetro público de AES-CBC
  iterationCount=10000               # parámetro público de PBKDF2
```

> PIN ──PBKDF2(salt, 10000)──► clave AES-256 ──AES-CBC(iv)──► descifra encryptedSecret

