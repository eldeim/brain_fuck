# iOS Device Setup

A physical device is helpful since a jailbroken device is required for some parts in this course. If you do not have a physical device, you can use our Lab Environment for this course as explained in [Corellium Device Setup](https://www.mobilehackinglab.com/path-player?courseid=ios-appsec\&unit=66dee0a8e0d02c8199018fe9) section. This guide walks you through setting up your device and installing necessary tools.

### Preparing the Device  <a href="#el_1726505073348_499" id="el_1726505073348_499"></a>

* **Factory Reset:** Start with a clean device by performing a factory reset. Go to **Settings > General > Reset > Erase All Content and Settings**.
* **Enable Developer Mode:** To use developer tools, enable developer mode by going to **Settings > Privacy & Security > Developer Mode**. This requires approval and a device restart.
* **Disable Lock Screen and Passcode:** During testing, you may want to disable any lock screen or passcode to make frequent access easier.

### Setup Device for Development <a href="#el_1726505103379_511" id="el_1726505103379_511"></a>

1\. **Install Xcode**: For **Xcode** setup, read [Software Setup](https://www.mobilehackinglab.com/path-player?courseid=ios-appsec\&unit=66c5b9fee8fdad44270acb8e).\
2\. **Connect Device to Xcode**: Connect the device to your Mac and open **Xcode**. In **Xcode**, go to **Window > Devices and Simulators**, select the connected device, and enable it for development.\
3\. **Trust the Device**: On the iOS device, go to **Settings > General > Device Management** and trust the connected Mac.\
4\. **Provisioning Profile**: Generate or use a provisioning profile to allow app installation and testing on the device.

### Jailbreak the iOS Device <a href="#el_1726505213151_572" id="el_1726505213151_572"></a>

Jailbreaking is not always required for testing app security, but it can provide deeper access to the device and app data.

#### Step 1: Choose a Jailbreak Method <a href="#el_1727084411654_1286" id="el_1727084411654_1286"></a>

The method / tool to use depends on the iOS version and supported iPhones based on chipset.\
[https://theapplewiki.com/wiki/Jailbreak](https://theapplewiki.com/wiki/Jailbreak) is a good resource to check which tool is suitable. \
\
Popular jailbreak tools include:<br>

* **Dopamine** (iOS 15-16)
* **palera1n** (iOS 15-17)
* **Checkra1n** (iOS 12 - 14)
* **Unc0ver** iOS 11 - 14.3)
* **Taurine** or **Chimera** (for iOS 14.x)

#### Step 2: Install Cydia or Sileo <a href="#el_1727084416956_1301" id="el_1727084416956_1301"></a>

Package managers allow you to install jailbreak tweaks and tools.

#### Step 3: Installing Security Testing Tools for Jailbroken Devices <a href="#el_1727084423819_1306" id="el_1727084423819_1306"></a>

* Install **Frida**, **SSL Kill Switch**, **Cycript**, or **Radare2** using a package manager or through SSH.
* Use **Filza** or **iFile** for file system exploration.

> _Note: Jailbreaking a device removes many built-in security features, so avoid using it as your main device. It also voids Apple’s warranty._

### Installing Apps <a href="#el_1726505192438_560" id="el_1726505192438_560"></a>

You can install apps for testing in a few ways:

* **Install via Xcode**: Compile and run the app on the connected device through **Xcode**.
* **Install from IPA**: If you have an IPA file (iOS app archive), you can use tools like **ideviceinstaller** (from **libimobiledevice**) or **Cydia Impactor** to sideload it onto the device.
* **Install from App Store**: Download the app directly from the **App Store**, if possible. However, you may not have access to certain debug features without a developer build.

### System Logs and Monitoring <a href="#el_1726505239176_586" id="el_1726505239176_586"></a>

* **View System Logs**: Use **Xcode**’s console or **idevicesyslog** (from **libimobiledevice**) to monitor system logs in real-time.
* **Install Syslog Tools**: For jailbroken devices, you can install **Syslog** from **Cydia** to capture detailed app logs.

### Restoring the Device <a href="#el_1726505265206_598" id="el_1726505265206_598"></a>

If anything goes wrong or you want your old content for any reason, you may want to restore the device to its original state:

* **Factory Reset**: Go to **Settings > General > Reset > Erase All Content and Settings** to restore the device.
* **Unjailbreak (if applicable)**: Use tools like **Cydia Eraser** to remove the jailbreak and restore the stock iOS system.
