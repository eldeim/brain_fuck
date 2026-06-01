# LAB - Insecure Biometric Failover

In this lab environment, you will have GUI access to a Debian machine. The **QuickPay** application is available on the Android Emulator.

**Objective:** Identify the vulnerability in the QuickPay app’s biometric failover mechanism and leverage it to gain access to the dashboard.

> **Note:** You can start the emulator using the script located on the Desktop. Additionally, check the **/root/Tools** directory for available tools.

### Tools

The best tools for this lab are:

* ADB
* Jadx-GUI
* Emulator Extended Controls

***

Frist open the android emulator and open the "QuickPay" app, after that, we can see a biometric login

<figure><img src="../../../../.gitbook/assets/image (1236).png" alt=""><figcaption></figcaption></figure>

Now, in another terminal, use adb to get the apk file, frist search by he name and then, run this path to get apk

```
## Search app
adb shell pm list packages -f | grep "quickpay"
## Get APK
adb pull /data/app/com.example.quickpay-w99vl4pFDB3yTvqIV4jGtA==/base.apk .
```

<figure><img src="../../../../.gitbook/assets/image (1237).png" alt=""><figcaption></figcaption></figure>

Now use jadx-gui to see the source code and go to "MainActivity" to search the function about biometric login -->

```
jadx-gui base.apk
```

<figure><img src="../../../../.gitbook/assets/image (1238).png" alt=""><figcaption></figcaption></figure>

The class maintains a private integer `failureCount` to keep track of how many times biometric authentication has failed, starting at zero.

> Scroll down to line 37.

<figure><img src="../../../../.gitbook/assets/image (1239).png" alt=""><figcaption></figcaption></figure>

In the highighted code above, the `onAuthenticationFailed()` method is called whenever a biometric authentication attempt fails. It increments the `failureCount` by invoking the synthetic `access$008` method, which increases the number of failed attempts by one. . After incrementing, it checks if the failure count has reached or exceeded the maximum allowed attempts (which is 5). If so, it triggers a fallback by launching the **PIN authentication activity** to let the user verify their identity through an alternative method

Now, we can see into Navigate to **Source code > com > example.quickpay > PinActivity**

<figure><img src="../../../../.gitbook/assets/image (1240).png" alt=""><figcaption></figcaption></figure>

The primary vulnerability in this PinActivity code is the hardcoding of the PIN directly in the source as a plain string (`123456`), which makes it easily discoverable through reverse engineering or decompiling the app. This allows anyone to bypass biometric authentication simply by entering the known PIN.

Now, On the emulator, click the three dots to open the **Extended Controls** window.

<figure><img src="../../../../.gitbook/assets/image (1241).png" alt=""><figcaption></figcaption></figure>

Navigate to the Fingerprint section. We assume that we don't have the correct fingerprint, so our objective is to trigger the fallback mechanism (PinActivity)

<figure><img src="../../../../.gitbook/assets/image (1242).png" alt=""><figcaption></figcaption></figure>

We notice that the biometric authentication has failed and we are left with 4 more attempts.

<figure><img src="../../../../.gitbook/assets/image (1243).png" alt=""><figcaption></figcaption></figure>

Now, enter the hardcoded PIN (123456) that we discovered and click on Submit.
