# Host software setup

## Android studio  <a href="#el_1710184298117_386" id="el_1710184298117_386"></a>

Android Studio is used by Developers to build Android Apps, and also useful if you want to setup local Virtual Devices (Android Virtual Device, called AVD) instead of a Physical Device or Cloud solution.\
\
Download Android Studio from: [https://developer.android.com/studio](https://developer.android.com/studio)\
\
If you want to use Android Virtual Devices on your local machine, or \
Make sure to install Android SDK build-tools, Android-SDK Platform tools, And Android Emulator.

*   After you open Android Studio go to more actions -> SDK Manager<br>

    <figure><img src="https://lwfiles.mycourse.app/63942c32c9a203516ce07c09-public/ebook/ac74f32f4a3cc347283462dad88a3b71/image1.png" alt=""><figcaption></figcaption></figure>
*   Search for Android SDK:<br>

    <figure><img src="https://lwfiles.mycourse.app/63942c32c9a203516ce07c09-public/ebook/ac74f32f4a3cc347283462dad88a3b71/image2.png" alt=""><figcaption></figcaption></figure>

<br>

*   Select the components to install (Android SDK build-tools, Android-SDK Platform tools, And Android Emulator):

    <figure><img src="https://lwfiles.mycourse.app/63942c32c9a203516ce07c09-public/ebook/ac74f32f4a3cc347283462dad88a3b71/image3.png" alt=""><figcaption></figcaption></figure>
*   For the emulator pick a recent Android version to install in the SDK platforms tab

    <figure><img src="https://lwfiles.mycourse.app/63942c32c9a203516ce07c09-public/ebook/ac74f32f4a3cc347283462dad88a3b71/image4.png" alt=""><figcaption></figcaption></figure>

## ADB isolated installation <a href="#el_1710184298143_409" id="el_1710184298143_409"></a>

To use devices provided by Mobile Hacking Lab only ADB (isolated) is needed

### Linux <a href="#el_1710364455818_736" id="el_1710364455818_736"></a>

Execute the commands:

```
sudo apt-get update
```

```
sudo apt-get install android-tools-adb
```

### OSX <a href="#el_1710184298156_421" id="el_1710184298156_421"></a>

Execute the commands:

```
brew cask install android-platform-tools
```

### Windows / all <a href="#el_1710184298161_427" id="el_1710184298161_427"></a>

Download the files from [https://developer.android.com/studio/releases/platform-tools](https://developer.android.com/studio/releases/platform-tools)Add the folder to your path environment variable.

***

## Local Android Virtual Device setup <a href="#el_1698173623285_595" id="el_1698173623285_595"></a>

### Create Android Virtual Device (AVD) <a href="#el_1698173623285_611" id="el_1698173623285_611"></a>

1. Download and setup [Android Studio](https://developer.android.com/studio)&#x20;
2. Open Android Studio and go to Virtual Device Manager

<img src="https://lwfiles.mycourse.app/63942c32c9a203516ce07c09-public/10fb0092e3b19100dae05d7d2c8deebf.png" alt="" height="142" width="461">

3\. Create a new Device via: Create Device

* For example: Pixel 5, with Android 13 (API 33) and Google API's
* > In my case, Pixel 7a with Android 16 (API 36) & Google API's
  >
  > ![](<../../../../../../.gitbook/assets/image (1460).png>)
* Use recommended architecture, depending on your hardware. On Mac m1 and higher you can use ARM on Windows this is not recommended.
* > In my case, I add 6gb of ram

<img src="https://lwfiles.mycourse.app/63942c32c9a203516ce07c09-public/e537c3d78d2bff8c4b081b768b64e0a8.png" alt="" height="233" width="579">

<img src="https://lwfiles.mycourse.app/63942c32c9a203516ce07c09-public/a1a606b2d629ddc89b13bc48eb5ae03a.png" alt="" height="665" width="984">

<img src="https://lwfiles.mycourse.app/63942c32c9a203516ce07c09-public/67fe96cab43f458b9ffc51c1f1cc308c.png" alt="" height="673" width="991">



