# LAB - Sensitive Information Leakage

In this lab environment, you will have GUI access to a Debian machine. An application named **HeyDoc** is available on the Android Emulator.

**Objective:** Identify sensitive information leaked by the HeyDoc app by intercepting and analyzing network requests using Burp Suite.

**Note:** Allow the app to make and manage phone calls by granting the required permission when prompted.

The valid credentials for **HeyDoc** app are as follows:

* **Username:** alice
* **Password:** Bazinga@12345#

> **Note:** You can start the emulator using the script located on the Desktop. Additionally, check the **Tools** directory located on the Desktop for available tools.

***

Frist, ejecute the APK and see that que APP make and manage the Phone Calls, after accept, we have in front of us a login panel, and insert the credentials to log in-->

<figure><img src="../../../.gitbook/assets/image (1287).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1288).png" alt=""><figcaption></figcaption></figure>

With it, configure the Burp Porixy and navagete into the APK to intercep all peticions -->

<figure><img src="../../../.gitbook/assets/image (1289).png" alt=""><figcaption></figcaption></figure>

Now, or configure the Wifi proxy same as burp of do it about command line -->

```
adb shell settings put global http_proxy <host-ip>:8080
```

<figure><img src="../../../.gitbook/assets/image (1290).png" alt=""><figcaption></figcaption></figure>

With it made, navegate on the APK -->

<figure><img src="../../../.gitbook/assets/image (1291).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1292).png" alt=""><figcaption></figcaption></figure>

1. `Authorization: Bearer token_101_20250602080848`: This header typically includes an access token prefixed with "Bearer", which is used to authenticate the request.
2. `_ygs: SU1FST0zNTgyNDAwNTExMTExMTA7VXNlcm5hbWU9YWxpY2U=`: This appears to be a base64 encoded data

So, decode Base64 the \_ygs paraphers -->

```
echo "SU1FST0zNTgyNDAwNTExMTExMTA7VXNlcm5hbWU9YWxpY2U=" | base64 --decode
```

<figure><img src="../../../.gitbook/assets/image (1293).png" alt=""><figcaption></figcaption></figure>

We have found that an **IMEI number** and the **logged-in username** are being sent with the request in an inconspicuous parameter within the request header.
