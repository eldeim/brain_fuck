# LAB - Insecure LogIn

In this lab environment, you will have GUI access to a Debian machine. The **InsecureBankv2** application is available on the Android Emulator.

**Objective:** Identify the insecure logging vulnerability in the InsecureBankv2 application by monitoring log outputs for exposed sensitive data such as usernames and passwords.

The valid credentials for InsecureBankv2 are as follows:

* **Username:** dinesh
* **Password:** Dinesh@123$

> **Note:** You can start the emulator using the script located on the Desktop. Additionally, check the **/root/Tools** directory for available tools.

***

First, execute the android emulator and open the APK. After this, we can see and configurate a server IP into preferences, secute python app web server an set us IP and Port -->

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Before it, we can try to login and we can see the credentials send -->

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now, Open a new terminal and run the following command to find the process ID (PID) of the target app.

```
adb shell ps | grep bank
```

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> The PID of the target app is 12543. Please note, this value may differ for you.

Next, run the following command to monitor the device log related to the InsecureBank app:

```
adb logcat | grep 12543
```

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

You will observe that the credentials are logged, leading to sensitive information leakage



Let's take a look at the vulnerable source code.

First, determine the location of the target apk file and pull it your host machine using the following commands:

```
## Search APK
adb shell pm list packages -f | grep "insecurebank"
## Download APK
adb pull /data/app/com.android.insecurebankv2-xu-xOuIlKClfvJjYmNC-Jw==/base.apk .
## Open APK
jadx-gui base.apk
```

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This logs a message with the username and password of the successfully logged-in user. It uses the `Log.d()` method to print debug-level information, which is commonly used for development and debugging.

Next, navigate to **Source code > com > android.insecurebankv2 > MyBroadCastReceiver**.

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> During the password change, the highlighted line above logs both the phone number and the password to the console, potentially leaking sensitive information.
