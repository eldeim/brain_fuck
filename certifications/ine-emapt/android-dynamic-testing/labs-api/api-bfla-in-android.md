# API BFLA in Android

In this lab environment, you will get access to a Debian machine, which has all the required tools installed on it for this lab, along with an Android emulator. To start the Android emulator, run the "startemulator.sh" script present at "Desktop."

**Objective:** Identify and exploit a Broken Function Level Authorization (BFLA) vulnerability.

The following Android application can be useful:

* NovaTech.apk: Intentionally vulnerable Android application. (Pre-installed on the emulator).

The following credentials can be useful:

```
Username: alice
Password: pass
```

***

The frist thing we do is exec the android emulator and login with the credentials getting -->

<figure><img src="../../../../.gitbook/assets/image (1298).png" alt=""><figcaption></figcaption></figure>

We have logged into the user profile of "Alice," where we can view the associated user data and account details.

Click on the "Dashboard" button.

<figure><img src="../../../../.gitbook/assets/image (1300).png" alt=""><figcaption></figcaption></figure>

We are presented with the user dashboard overview. Here we can see some more user details.

<figure><img src="../../../../.gitbook/assets/image (1301).png" alt=""><figcaption></figcaption></figure>

Open a new terminal and check the system IP, and set the global HTTP proxy on the Android device to the system IP address.

```
## View us IP
ip addr
## Set Proxy with us IP
adb shell settings put global http_proxy <IP>:8080
```

Now, config the Burp Proxy

<figure><img src="../../../../.gitbook/assets/image (1294).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (1295).png" alt=""><figcaption></figcaption></figure>

With it, we can see that the proxy its woking weel. So... Now intercept "Dashboard"peticions -->

<figure><img src="../../../../.gitbook/assets/image (1296).png" alt=""><figcaption></figcaption></figure>

We can manipulate the user\_id, and see information about others user (IDOR)

<figure><img src="../../../../.gitbook/assets/image (1297).png" alt=""><figcaption></figcaption></figure>
