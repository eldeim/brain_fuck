# LAB - Insecure Data Storage

In this lab environment, you will get access to a Debian machine, which has all the required tools installed on it for this lab, along with an Android emulator. To start the Android emulator, run the "startemulator.sh" script present at "/root/Desktop."

**Objective:** Find and exploit the insecure data storage vulnerability.

The following Android application can be useful:

* **Salesperson.apk**: A "Salesperson" app that stores orders in a world-readable file. (Pre-installed on the emulator.)
* **Readorders.apk**: A malicious app which can read the orders stored by the "Salesperson" app. (Pre-installed on the emulator.)

### Tools

* **adb:** Android Debug Bridge is a versatile command-line tool that allows developers and users to interact with Android devices and emulators. It's part of the Android SDK Platform-Tools package and provides a range of functionalities for debugging, testing, and managing Android devices.
* **Jadx:** Jadx is a popular open-source tool used for decompiling and analyzing Android application packages (APKs). It allows developers and security researchers to reverse-engineer APK files to understand their inner workings, extract resources such as source code, images, and other assets, and analyze the app's behavior.

***

Frist ejecuta the emulator and open the app and test -->

<figure><img src="../../../.gitbook/assets/image (1253).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1254).png" alt=""><figcaption></figcaption></figure>

So, i can create an new order and now, i can seach by ID... it smells like IDOR... xd

<figure><img src="../../../.gitbook/assets/image (1257).png" alt=""><figcaption></figcaption></figure>

Now, i extract the apk file

<pre><code><strong>## Search package
</strong><strong>adb shell pm list packages -f "salesperson"
</strong>## Get APK
adb pull /data/app/com.litesh.salesperson-1n_UBhYLrPkOIsKm5YFQWg==/base.apk /root/Desktop/
</code></pre>

<figure><img src="../../../.gitbook/assets/image (1255).png" alt=""><figcaption></figcaption></figure>

The apk file has been decompiled. Now, we can use this jadx-gui tool to perform our further analysis.

Here, navigate to the "Source code" > "com" > "litesh.salesperson" > "MainActivity" file.

<figure><img src="../../../.gitbook/assets/image (1256).png" alt=""><figcaption></figcaption></figure>

We can see the `getFilePath` method returns a File object pointing to `orders.txt` inside the public `Documents` directory on external storage.

<figure><img src="../../../.gitbook/assets/image (1258).png" alt=""><figcaption></figcaption></figure>

Next, the `saveOrder` method saves a given entry (as a string) to a file named `orders.txt` in the public `Documents` directory on external storage.

<figure><img src="../../../.gitbook/assets/image (1259).png" alt=""><figcaption></figcaption></figure>

> This code is vulnerable as it stores sensitive data (e.g., order entries) in plain text on external public storage, which is accessible by any app with storage permissions. This exposes the data to unauthorized access, tampering, or leakage, especially if the data contains personal or financial information. Secure storage options like encrypted internal storage or the Android Keystore should be used instead.

First, let's check if we can view the `orders.txt` file. Open the apps menu and look for the "Files" app.

<figure><img src="../../../.gitbook/assets/image (1260).png" alt=""><figcaption></figcaption></figure>

Select "AOSP on IA Emulator" storage from the hamburger menu.

<figure><img src="../../../.gitbook/assets/image (1261).png" alt=""><figcaption></figcaption></figure>

Then go to the "Documents" directory.

<figure><img src="../../../.gitbook/assets/image (1262).png" alt=""><figcaption></figcaption></figure>

Here, we can find the `orders.txt` file.

<figure><img src="../../../.gitbook/assets/image (1263).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1264).png" alt=""><figcaption></figcaption></figure>

We can see the data in plain text (unencrypted).

<figure><img src="../../../.gitbook/assets/image (1265).png" alt=""><figcaption></figcaption></figure>

Next, we will use the "Readorders" APK to read the data stored by the "Salesperson" app.

> **Note:** The "Readorders" APK is acting as a malicious application which is capable of reading the `orders.txt` file created by the "Salesperson" app.

Open the "Readorders" present on the home screen.

<figure><img src="../../../.gitbook/assets/image (1266).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1267).png" alt=""><figcaption></figcaption></figure>

We were successfully able to read the data from the `orders.txt` file created by the "Salesperson" app.

<figure><img src="../../../.gitbook/assets/image (1268).png" alt=""><figcaption></figcaption></figure>

Now, let's pull this malicious "Readorders" APK and decompile it to analyze the source code.

```
## Search File
adb shell pm list packages -f "readorders"
## Get File
adb pull /data/app/com.litesh.readorders-E5v_xQq_ETiMXTdsWhRm3A==/base.apk ~
```

<figure><img src="../../../.gitbook/assets/image (1269).png" alt=""><figcaption></figcaption></figure>

```
## Read and Open
jadx-gui base.apk
```

Navigate to the "MainActivity" file. The code in the method `readOrdersFile` tries to read a file named orders.txt from the public Documents directory on the device's external storage. The same location where the "Salesperson" app is storing the `orders.txt` file.

<figure><img src="../../../.gitbook/assets/image (1270).png" alt=""><figcaption></figcaption></figure>
