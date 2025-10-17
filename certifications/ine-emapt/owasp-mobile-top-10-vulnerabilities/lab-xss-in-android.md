# LAB -  XSS in Android

In this lab environment, you will get access to a Debian machine, which has all the required tools installed on it for this lab, along with an Android emulator. To start the Android emulator, run the "startemulator.sh" script present at "/root/Desktop."

**Objective:** Find and exploit the XSS vulnerability in the vulnerable APK.

The following Android application can be useful:

* allsafe.apk: Intentionally vulnerable Android application. (Pre-installed on the emulator).

### Tools

* **adb:** Android Debug Bridge is a versatile command-line tool that allows developers and users to interact with Android devices and emulators. It's part of the Android SDK Platform-Tools package and provides a range of functionalities for debugging, testing, and managing Android devices.
* **Jadx:** Jadx is a popular open-source tool used for decompiling and analyzing Android application packages (APKs). It allows developers and security researchers to reverse-engineer APK files to understand their inner workings, extract resources such as source code, images, and other assets, and analyze the app's behavior.

***

Frist ejecute the android emulator with `./startemulator.sh`

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Get APK

Now, while the emulator run, we extract the "Allsafe" app from the emulator to perform our analysis

```bash
## Extract APK files
adb shell pm list packages -f
```

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> We have a list of all the packages

let's narrow down this list to find the package for the "Allsafe" app -->

```bash
## List concret "allsafe" apk app
adb shell pm list packages -f "allsafe"
```

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now, let's pull this package, to obtain APK file -->

```
adb pull /data/app/~~oZ0lNhDdkIp2NaWMhGczgw==/infosecadventures.allsafe-ttByxQb49HI7GiOb62XhPQ==/base.apk /root/Desktop/
```

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Jadx Tool Inspect

Wich the APK file obtain, we can use jadx-gui to decompile and read&#x20;

```
jadx-gui base.apk
```

### XXS Identificate

Now examinate code use click on the search icon and search for the text "`setJavaScriptEnabled`", and select the node and click on "Open".

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Here we can notice the code seems to be vulnerable to XSS, as:

* JavaScript is enabled via: `settings.setJavaScriptEnabled(true);`
* User input is loaded directly into WebView with: `webView.loadData(payload.getText().toString(), "text/html", "UTF-8");`
* There is no input sanitization or validation for malicious scripts in payload.

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### XSS Ejecute

Knowing this, we can ejecute the app and try to make a basic HTML Injection & Basic XSS ->

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

