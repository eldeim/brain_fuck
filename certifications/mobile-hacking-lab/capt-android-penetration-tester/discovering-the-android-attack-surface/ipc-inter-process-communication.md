# IPC (Inter-Process Communication)

* Export Activities
* Export Broadcast receivers
* Export Content Providers
* Exported Services

***

## Export Activities

<figure><img src="../../../../.gitbook/assets/image (1482).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (1483).png" alt=""><figcaption></figcaption></figure>

***

### Explotation

* Extract the source code from the AndroGoat APK, with apktool, jadx or any other tool which can decode the AndroidManifest.
* Open “AndroidManifest.xml”
* $DIR/resources/AndroidManifest.xml
* Find the exported receiver that has an “intent-filter” or attribute “android:exported” set to true
* Kill the AndroGoat app (just to be sure)
* Open a user shell with ADB
* Start the relevant Activity to open the protected screen using “am” and the following syntax:

> Search packages
>
> ```
>  pm list packages | grep goat
> ```

```
adb shell am start com.example.package/.className
```

```
adb shell am start "[data]" com.example.package/.className
```

```
adb shell am start –a [action] –c [category] com.example.package/.className
```

* Verify you successfully bypassed the 'pin screen' and can directly download the sensitive information from the 'invoice screen'

***

## Exported Services <a href="#el_1715342932965_354" id="el_1715342932965_354"></a>

<figure><img src="../../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### Exploitation <a href="#el_1715342932968_363" id="el_1715342932968_363"></a>

* Extract the source code from the AndroGoat APK
* Open “AndroidManifest.xml”
* $DIR/resources/AndroidManifest.xml
* Find the exported service that has an “intent-filter” or attribute “android:exported” set to “true”
* Open the AndroGoat app
* Open a user shell with ADB
* Start the service using “am"

```
adb shell am startservice com.example.package/.className
```

* Open the “Downloads” folder via the Files system app
* Open the downloaded file
* Extract the sensitive information

***

Export Broadcast receivers


<figure><img src="../../../../.gitbook/assets/image (1488).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (1487).png" alt=""><figcaption></figcaption></figure>

### Exploitation <a href="#el_1715343305479_363" id="el_1715343305479_363"></a>

* Extract the source code from the AndroGoat APK
* Open “AndroidManifest.xml”
* $DIR/resources/AndroidManifest.xml
* Find the exported activity that has an “intent-filter” or attribute “android:exported” set to “true”
* Open the AndroGoat app
* Open a user shell with ADB
* Broadcast to the receiver using “am”
* am broadcast -n "…"
* Quickly check your device’s screen
* Extract the sensitive information from the Toast message
