# Devices needed



<figure><img src="../../../../../.gitbook/assets/image (1464).png" alt=""><figcaption></figcaption></figure>

* 2 Devices with Google APIS
  * 1 rooted
  * 1 normal
* 2 Devices with Google Play Store
  * 1 rooted
  * 1 normal

<figure><img src="../../../../../.gitbook/assets/image (1465).png" alt=""><figcaption></figcaption></figure>

***

## How to ROOT Google Play Store Divices

1. Use the rootAVD procedure from [https://github.com/newbit1/rootAVD](https://github.com/newbit1/rootAVD) to get root to the device.

```
git clone https://github.com/newbit1/rootAVD.git
```

```
cd rootAVD
```

#### Linux / Mac <a href="#el_1710522352698_419" id="el_1710522352698_419"></a>

```
./rootAVD.sh ListAllAVDs
```

```
./rootAVD.sh system-images/android-33/google_apis_playstore/arm64-v8a/ramdisk.img
```

#### Windows <a href="#el_1710522401215_438" id="el_1710522401215_438"></a>

```
rootAVD.bat ListAllAVDs
```

```
rootAVD.bat %LOCALAPPDATA%\Android\Sdk\system-images\android-33\google_apis_playstore\x86\ramdisk.img
```

Cold boot now

<img src="https://lwfiles.mycourse.app/63942c32c9a203516ce07c09-public/dbdf9eeb67a677592b1d2ff75c3dac6e.png" alt="" height="106" width="418">

***

## How to ROOT Google APIs Devices

{% embed url="https://blog.deephacking.tech/es/posts/como-instalar-magisk-en-dispositivos-emulados-con-android-studio/" %}
