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
