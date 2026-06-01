# LAB - Testing Debuggability

In this lab environment, you will get access to a Debian machine, which has all the required tools installed on it for this lab, along with an Android emulator. To start the Android emulator, run the "startemulator.sh" script present at "/root/Desktop."

**Objective:** Perform static and dynamic analysis to determine if an application is debuggable.

The following Android application can be useful:

* allsafe.apk: Intentionally vulnerable Android application. (Pre-installed on the emulator).

### Tools

* **adb:** Android Debug Bridge is a versatile command-line tool that allows developers and users to interact with Android devices and emulators. It's part of the Android SDK Platform-Tools package and provides a range of functionalities for debugging, testing, and managing Android devices.
* **Jadx:** Jadx is a popular open-source tool used for decompiling and analyzing Android application packages (APKs). It allows developers and security researchers to reverse-engineer APK files to understand their inner workings, extract resources such as source code, images, and other assets, and analyze the app's behavior.
* **jdb:** JDB (Java Debugger) is a command-line debugging tool and a part of the Java Development Kit (JDK). It is used to debug Java applications by allowing developers to inspect and manipulate the execution of Java programs during runtime.

***

After run the andorid emulator, we can see an app "Allsafe", excute it

<figure><img src="../../../../.gitbook/assets/image (1244).png" alt=""><figcaption></figcaption></figure>

Now in another terminal, use adb to get the apk file -->

```
## Search package
adb shell pm list packages -f "allsafe"
## Get APK
adb pull /data/app/~~oZ0lNhDdkIp2NaWMhGczgw==/infosecadventures.allsafe-ttByxQb49HI7GiOb62XhPQ==/base.apk /root/Desktop/
```

Open it APK with `jadx-gui`

```
jadx-gui base.apk
```

Here, navigate to the "Resources" folder and look for the "AndroidManifest.xml" file.

<figure><img src="../../../../.gitbook/assets/image (1245).png" alt=""><figcaption></figcaption></figure>

### Search debuggable

Inside the "AndroidManifest.xml" file. search for the "`android:debuggable`" attribute

<figure><img src="../../../../.gitbook/assets/image (1246).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (1247).png" alt=""><figcaption></figcaption></figure>

We can see that the attribute is present, having "true" as its value; hence, we can conclude that the app is debuggable.

## Dynamic Analysis

You can also check if an application is debuggable by connecting `jdb` to its running process; if the connection is successful, it indicates that debugging is enabled.

Let's start a new process by running the "Allsafe" app.

<figure><img src="../../../../.gitbook/assets/image (1248).png" alt=""><figcaption></figcaption></figure>

Now, using adb and jdwp, we can identify the PID of the active application that we want to debug -->

```
adb jdwp
```

<figure><img src="../../../../.gitbook/assets/image (1249).png" alt=""><figcaption></figcaption></figure>

The last launched PID corresponds to our application.

Now we will create a communication channel by using adb between the application process (with the PID) and our host machine by using a specific local port.

```
adb forward tcp:55555 jdwp:<PID>
```

<figure><img src="../../../../.gitbook/assets/image (1250).png" alt=""><figcaption></figcaption></figure>

Now, using jdb, attach the debugger to the local communication channel port and start a debug session.

```
jdb -connect com.sun.jdi.SocketAttach:hostname=localhost,port=55555
```

<figure><img src="../../../../.gitbook/assets/image (1251).png" alt=""><figcaption></figcaption></figure>

Let's try the command given below.<br>

<figure><img src="../../../../.gitbook/assets/image (1252).png" alt=""><figcaption></figcaption></figure>

We successfully attached jdb to the running process. Hence, debugging is activated.

#### Conclusion

In this lab, we learnt how to identify whether an Android application is debuggable by analyzing its manifest file and using runtime tools like `adb`. We explored both static and dynamic techniques to determine the app's debug status, and gained hands-on experience with tools commonly used for analyzing debuggability. This foundational knowledge helps ensure that apps are properly configured for secure deployment.
