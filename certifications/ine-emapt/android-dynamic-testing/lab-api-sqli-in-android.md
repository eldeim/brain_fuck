# LAB - API SQLi in Android

In this lab environment, you will get access to a Debian machine, which has all the required tools installed on it for this lab, along with an Android emulator. To start the Android emulator, run the "startemulator.sh" script present at "Desktop."

**Objective:** Identify and exploit a SQL Injection (SQLi) vulnerability.

The following Android application can be useful:

* NovaTech.apk: Intentionally vulnerable Android application. (Pre-installed on the emulator).

The following credentials can be useful:

```
Username: alice
Password: pass
```

***

<figure><img src="../../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

After exucte the app, we can see a login panel and we can input credentials to login. Then, we can see a search "Schoolmate"  -->

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can to try set a basic SLQi -->

```
%'/**/OR/**/1=1-- 
```

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

But, the app dosent give us nothing response, so... we can try to intercep the traffic with burpsuite and read some content -->

#### Configurate Bupsuite

Frist, locate us local IP, wit it, go to Wi-Fi device and set the proxy of burpsuite, in this case: 10.138.0.36:8080 (because it is me IP address)

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

With it, we can up the proxy of burp, and finish to configurate the proxy of burpsite

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now, modify the peticion and set the SQLi -->

<figure><img src="../../../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
