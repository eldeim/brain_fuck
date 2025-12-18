# LAB - Mobile Vulnerabilities: SQLi in Android

In this lab environment, you will get access to a Debian machine, which has all the required tools installed on it for this lab, along with an Android emulator. To start the Android emulator, run the "startemulator.sh" script present at "/root/Desktop."

**Objective:** Identify and exploit the SQLi vulnerability in the vulnerable APK.

The following Android application can be useful:

* allsafe.apk: Intentionally vulnerable Android application. (Pre-installed on the emulator).

***

After execute the APK, we can see a login and into this we can try SQLi, execute and get credentials confirm that the SQLi exist -->

<figure><img src="../../../.gitbook/assets/image (16) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

After that, we can inspect the source code and see the variable to execute it login -->

<figure><img src="../../../.gitbook/assets/image (17) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
