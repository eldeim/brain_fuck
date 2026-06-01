# IPC (Inter-Process Communication)

* Export Activities
* Export Broadcast receivers
* Export Content Providers
* Exported Services

***

## Export Activities

<figure><img src="../../../../../.gitbook/assets/image (1482).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (1483).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

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


<figure><img src="../../../../../.gitbook/assets/image (1488).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (1487).png" alt=""><figcaption></figcaption></figure>

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

***

Deep Links


### Exploitation <a href="#el_1715346764817_363" id="el_1715346764817_363"></a>

* Extract the source code from the AndroGoat APK
* Open “AndroidManifest.xml”
* $DIR/resources/AndroidManifest.xml
* Search for Deep Links starting with "scheme:" in the AndroidManifest.xml
* Figure out how to use the defined links, and start / exploit them via adb commands, like the below one:

```
adb shell am start -W "[schema]://[host]/[path]?[queryparm]=[value]"
```

***

## WebViews <a href="#el_1715353406954_354" id="el_1715353406954_354"></a>

### Preparation <a href="#el_1715353406955_357" id="el_1715353406955_357"></a>

* Start an Android device
* Connect to the device viia ADB
* Install the AndroGoat APK
* [https://github.com/satishpatnayak/MyTest/blob/master/AndroGoat.apk](https://github.com/satishpatnayak/MyTest/blob/master/AndroGoat.apk)
* Prepare the shared preferences
* Open the AndroGoat app
* Go to “Insecure Data Storage” > “Shared Preferences – Part 1”
* Enter a username and password and click “Save”

### Execution <a href="#el_1715353406957_363" id="el_1715353406957_363"></a>

* Open the AndroGoat app
* Go to “Input Validations” > “Input Validations - WebViews”
* Enter a valid URL (https://owasp.org), click “Load” and check the result
* Extract the source code from the AndroGoat APK
* Understand the vulnerability in the code of “Input Validations – WebView”
* $DIR/sources/owasp/sat/agoat/InputValidationsWebViewURLActivity.java
* Create a file URI for the shared preferences in the AndroGoat data folder
* /data/data/owasp.sat.agoat/shared\_prefs/users.xml
* Enter the file URI, click “Load” and check the result
* Extract the sensitive information

***

## Shared Preferences <a href="#el_1715354819494_351" id="el_1715354819494_351"></a>

### Preparation <a href="#el_1715354819495_354" id="el_1715354819495_354"></a>

* Create an Android device
* Connect device to ADB
* Install the AndroGoat APK
* [https://github.com/satishpatnayak/MyTest/blob/master/AndroGoat.apk](https://github.com/satishpatnayak/MyTest/blob/master/AndroGoat.apk)
* Prepare the shared preferences
* Open the AndroGoat app
* Go to “Insecure Data Storage” > “Shared Preferences – Part 1”
* Enter a username and password and click “Save”

### Execution <a href="#el_1715354819497_360" id="el_1715354819497_360"></a>

* Open a root shell with ADB
* Browse to the AndroGoat data folder
* /data/data/owasp.sat.agoat/
* Find the shared preferences called “users.xml”
* Read the shared preferences with “cat”
* Extract the sensitive information

***

## Local Databases <a href="#el_1715355016345_354" id="el_1715355016345_354"></a>

<figure><img src="../../../../../.gitbook/assets/image (1503).png" alt=""><figcaption></figcaption></figure>

### Preparation <a href="#el_1715355016346_357" id="el_1715355016346_357"></a>

* Start an Android device
* Connect to the device via ADB
* Install the AndroGoat APK
* [https://github.com/satishpatnayak/MyTest/blob/master/AndroGoat.apk](https://github.com/satishpatnayak/MyTest/blob/master/AndroGoat.apk)
* Prepare the local database
* Open the AndroGoat app
* Go to “Insecure Data Storage” > “SQLite”
* Enter a username and password and click “Save”

### Exploitation <a href="#el_1715355016349_363" id="el_1715355016349_363"></a>

* Open a root shell with ADB
* Browse to the AndroGoat data folder
* /data/data/owasp.sat.agoat/
* Find the local database called “aGoat”
* Open the local database with “sqlite3” or 'sqlitebrowser'\
  List all tables and find the table where user information is stored
* Create and execute a query to all data from the table
* Extract the sensitive information

***

## Temp Data Storage <a href="#el_1715360272418_354" id="el_1715360272418_354"></a>

### Preparation <a href="#el_1715360272420_357" id="el_1715360272420_357"></a>

* Start an Android device
* Connect to the device via ADB
* Install the AndroGoat APK
* [https://github.com/satishpatnayak/MyTest/blob/master/AndroGoat.apk](https://github.com/satishpatnayak/MyTest/blob/master/AndroGoat.apk)
* Prepare the local database
* Open the AndroGoat app
* Go to “Insecure Data Storage” > “Temp File”
* Enter a username and password and click “Save”

### Exploitation <a href="#el_1715360272425_363" id="el_1715360272425_363"></a>

* Extract the source code from the AndroGoat APK
* Understand the vulnerability in the code of “Temp File”
* $DIR/sources/owasp/sat/agoat/InsecureStorageTempActivity.java
* Open a root shell with ADB
* Browse to the AndroGoat data folder
* /data/data/owasp.sat.agoat/
* Find the temp file
* Read the temp file with “cat”
* Extract the sensitive information

***

## SQL Injections

<figure><img src="../../../../../.gitbook/assets/image (1501).png" alt=""><figcaption></figcaption></figure>

### Preparation <a href="#el_1715360414753_357" id="el_1715360414753_357"></a>

* Start an Android device
* Connect to the device via ADB
* Install the AndroGoat APK
* [https://github.com/satishpatnayak/MyTest/blob/master/AndroGoat.apk](https://github.com/satishpatnayak/MyTest/blob/master/AndroGoat.apk)
* Prepare the local database
* Open the AndroGoat app
* Go to “Insecure Data Storage” > “SQLite”
* Enter a username and password and click “Save”
* Do this at least one more time with a different username

### Exploitation <a href="#el_1715360414755_363" id="el_1715360414755_363"></a>

<figure><img src="../../../../../.gitbook/assets/image (1502).png" alt=""><figcaption></figcaption></figure>

* Open the AndroGoat app
* Go to “Input Validations” > “Input Validations - SQLI”
* Enter a non-existing username, click “Verify” and check the result
* Enter an existing username, click “Verify” and check the result
* Enter a single quote (‘), click “Verify” and check the result
* Extract the source code from the AndroGoat APK
* Understand the vulnerability in the code of “Input Validations - SQLI”
* $DIR/sources/owasp/sat/agoat/SQLinjectionActivity.java
* Test SQLI payloads until you find all users
* Extract the sensitive information
