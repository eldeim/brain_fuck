# LAB - Insecure Token Management

In this lab environment, you will have GUI access to a Debian machine. An application named **HeyDoc** is available on the Android Emulator.

**Objective:** Your task is to find issues with HeyDoc's access token management and understand how they can be exploited to gain unauthorized access.

The valid credentials for the **HeyDoc** app are as follows:

* **Username:** alice
* **Password:** Bazinga@12345#

**HeyDoc's** app backend code is available at the following location for analysis:

* /home/student/Desktop/heydoc-backend

***

<figure><img src="../../../../.gitbook/assets/image (1307).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (1308).png" alt=""><figcaption></figcaption></figure>

Frist, run the andorid emultor and up the APP. After it, we can see a Reset Password Option, so... configurate the proxi -->

```
## View us IP 
hostname -I
## Set local proxy
adb shell settings put global http_proxy <host-ip>:8080
```

Now intercept the reset password peticion -->

<figure><img src="../../../../.gitbook/assets/image (1309).png" alt=""><figcaption></figcaption></figure>

This app know we are alice because we have a token, if we can enumate other token, maybe we can change the password of other user

But I can see one stranger thing into the token: "token\_number", if i change 101 by 102, i can change the password of others users -->

<figure><img src="../../../../.gitbook/assets/image (1310).png" alt=""><figcaption></figcaption></figure>
