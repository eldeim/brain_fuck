# LAB - JWT Token Manipulation

In this lab environment, you will have GUI access to a Debian machine. An application named **Playme** is available on the Android Emulator.

**Objective:** Manipulate the JWT token to impersonate an admin and retrieve the flag.

The regular user credentials for the **Playme** app are:

* **Username:** alice
* **Password:** Qwerty@1234567

> **Note:** You can start the emulator using the script located on the Desktop. Additionally, check the **/root/Tools** directory for available tools.

***

<figure><img src="../../../../.gitbook/assets/image (430).png" alt=""><figcaption></figcaption></figure>

After execute the app, we can see a login panel. Set credentials here -->

<figure><img src="../../../../.gitbook/assets/image (431).png" alt=""><figcaption></figcaption></figure>

Now, with it, configurate local proxy and burpproxy -\_>

```
## View us IP 
hostname -I
## Set local proxy
adb shell settings put global http_proxy <host-ip>:8080
```

<figure><img src="../../../../.gitbook/assets/image (434).png" alt=""><figcaption></figcaption></figure>

With it do, intercept the login peticion -->

<figure><img src="../../../../.gitbook/assets/image (432).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (433).png" alt=""><figcaption></figcaption></figure>

We can get a login token, and we can see three points so... copy and put in into jwio -\_>

<figure><img src="../../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

Know it, we can decode of base64 the content "white selection" and manipulate it. For example, change the role to admin -->

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
