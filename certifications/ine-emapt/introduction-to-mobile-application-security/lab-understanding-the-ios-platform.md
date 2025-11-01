# LAB - Understanding the iOS Platform

## **Inadequate Privacy Controls**

> In this lab environment, you will get access to a Debian machine, which has all the required tools installed on it for this lab, along with an Android emulator. To start the Android emulator, run the "startemulator.sh" script present at "/root/Desktop."

**Objective:** To analyze and differentiate between necessary and excessive permissions in a menu app to assess potential privacy risks.

The following Android application can be useful:

* foodies.apk: A demo menu (restaurant) app (Pre-installed on the emulator).

### Tools

* **adb:** Android Debug Bridge is a versatile command-line tool that allows developers and users to interact with Android devices and emulators. It's part of the Android SDK Platform-Tools package and provides a range of functionalities for debugging, testing, and managing Android devices.
* **Jadx:** Jadx is a popular open-source tool used for decompiling and analyzing Android application packages (APKs). It allows developers and security researchers to reverse-engineer APK files to understand their inner workings, extract resources such as source code, images, and other assets, and analyze the app's behavior.

***

### Basic Analize

First we need start the Android Emulator

```
cd /root/Desktop
./startemulator.sh
```

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now we need search into the mobile phone the app to audit with name: `Foodies`

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now, we open this app and analize the functionality, observe this app solicite us the cam, gps, audio, calendar, phone calls and SMS pemissions

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

After we've granted permissions, we can see that it's a demo menu app in which the user can select products to order. Select few items and click on "ORDER NOW" button.-->

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Extract adb Packects - APK

1. Search the install package of the app "Foddies" into the emulate

```
adb shell pm list packages -f "foodies"
```

> `adb shell` : open a shell into the android emulator
>
> `pm list packages -f` : list all packages install
>
> `"foddies"` : filter by name

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now, extract the APK file into emulator into us PC -->

```
adb pull /data/app/~~8OwxUFHEPiFvMY755MLmeg==/com.example.foodies-zzYZRENkYLpy0ZuZMnPfPA==/base.apk ./
```

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Examine APK with Jadx-gui

Now we can use jadx-gui to examine te content about this APK

```
jadx-gui base.apk
```

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now navigate to the "Resources" > "AndroidManifest.xml" file

<figure><img src="../../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Here we can see the list of permissions that the app is requesting. Let's examine these permissions and classify them as "essential", "optional" and "irrelevant" permissions.

> The table below outlines the various permissions requested by the menu app, along with their justifications and relevance. By analyzing each permission, we can determine whether it is genuinely required for the app’s functionality or if it represents an unnecessary or excessive access request. This classification helps in identifying which permissions are essential and optional and which are irrelevant or privacy-invasive in the context of a food-related application.

#### Summary Table:

| Permission               | Justification | Notes                                                                 |
| ------------------------ | ------------- | --------------------------------------------------------------------- |
| `ACCESS_COARSE_LOCATION` | Yes           | Location-based services.                                              |
| `ACCESS_FINE_LOCATION`   | Yes           | More accurate location-based services.                                |
| `READ_CONTACTS`          | Maybe         | If inviting/sharing with contacts. Should be optional.                |
| `RECORD_AUDIO`           | Maybe         | For voice interaction. Should be optional.                            |
| `CALL_PHONE`             | No            | Risky unless explicitly calling restaurants.                          |
| `CAMERA`                 | Maybe         | If used for QR scanning/photos. Should be justified in app UX.        |
| `SEND_SMS` / `READ_SMS`  | No            | High-risk. OTPs better handled by SMS Retriever API or Firebase Auth. |
| `BLUETOOTH`              | No            | Irrelevant unless there's a unique feature involving beacons/devices. |
| `READ_CALENDAR`          | No            | Highly suspicious; no food app needs calendar access.                 |

Now, let's continue our analysis by navigating to "Source code" > "com" > "example.foodies" > "MainActivity"

Here, we observe that the app defines an array named `requiredPermissions`, which includes several irrelevant Android permissions

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

These permissions at runtime using the `ActivityCompat.requestPermissions()` method within the `checkPermissions()` function.

<figure><img src="../../../.gitbook/assets/image (14) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Conclusion

In this case, a food menu app, and distinguish between those that are essential for core functionality and those that pose unnecessary privacy risks. This exercise reinforced the importance of applying the principle of least privilege in mobile app development, where apps should only request permissions that are absolutely necessary and always provide users with transparency and control over their data.
