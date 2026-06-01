---
description: Guide to Using Frida for Dynamic Instrumentation and Analysis
---

# Intrumentation on iOS - Frida setup + codeshare

<figure><img src="../../../../../../.gitbook/assets/image (44).png" alt="" width="437"><figcaption></figcaption></figure>

### Prerequisites <a href="#el_1724273565812_367" id="el_1724273565812_367"></a>

* **Jailbroken iOS Device**: Frida requires a jailbroken iOS device for proper functionality.
* **Python**: Ensure Python is installed on your computer (for using Frida tools and writing scripts). You can download it from [python.org](https://www.python.org/) or install via **brew**.
* **SSH Access to iOS Device**: You should be able to connect to your iOS device via SSH.
* **Target App**: The **DVIA-v2** app should be installed on your device. Download it from the GitHub repository [prateek147/DVIA-v2](https://github.com/prateek147/DVIA-v2).

## Part 1: Setting Up Frida for iOS <a href="#el_1726095381471_379" id="el_1726095381471_379"></a>

### Step 1: Install Frida cliet on Your Computer <a href="#el_1726095701289_444" id="el_1726095701289_444"></a>

<figure><img src="../../../../../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

First, install Frida tools on your computer using Python's package manager **pip**:

```
pip install frida-tools
// or
pipx install frida-tools
```

This will install Frida along with helpful tools like **frida**, **frida-ps**, **frida-trace**, and **frida-ls-devices**.

#### Step 2: Install Frida on an iOS Device (not needed on lab device) <a href="#el_1726095622965_429" id="el_1726095622965_429"></a>

> **If you are using a Mobile Hacking Lab Device, frida-server is already installed and you can skip this step.**
>
> ![](<../../../../../../.gitbook/assets/image (1445).png>)

* If you want to install Frida on a non -rooted device or jailbroken device you can follow below steps, or follow [this manual on HackTricks](https://book.hacktricks.xyz/mobile-pentesting/ios-pentesting/frida-configuration-in-ios#installing-frida).
* For upgrading frida server on the built-in Corellium device [follow this manual from Corellium](https://support.corellium.com/features/frida/change-the-frida-server-version#replacing-the-built-in-frida-server-on-ios)

#### Use Frida since Corellium

1. What if frida-server is running:

```
ps aux | grep frida
```

If not be, weak up the server:

```
frida-server &
```

2. Conexion, list the process

```
frida-ps -R
```

And list all apps installed:

```
frida-ps -Ra
```

3. Interact with apps, Spawnear app

```
frida -R -f com.highaltitudehacks.DVIAswiftv2
```

Spawnear + script:

```
frida -R -f com.highaltitudehacks.DVIAswiftv2 -l script.js
```

Spawnear whitout pause:

```
frida -R -f com.highaltitudehacks.DVIAswiftv2 --no-pause
```

Once your iOS device is jailbroken, you can install the Frida server. Follow these steps:\
\
1\. **Connect to the iOS device via SSH**:

```
ssh root@10.11.1.1
```

2\. **Add the Frida repository**:

```
echo "deb https://build.frida.re/ ./" >> /etc/apt/sources.list.d/cydia.list
```

3\. **Install the Frida server**:

```
apt update
```

```
apt install re.frida.server
```

4\. **Start the Frida server**:

```
nohup frida-server &
```

5\. **Ensure Frida server is running**: The Frida server should now be running in the background. Make sure your device is reachable and Frida sever are reachable.

### Part 2: Common Frida Commands for iOS <a href="#el_1726131474573_760" id="el_1726131474573_760"></a>

#### 1. List Connected Devices <a href="#el_1726131484422_764" id="el_1726131484422_764"></a>

Use the **frida-ls-devices** command to check if your iOS device is connected and detected:

```
frida-ls-devices
```

It will display a list of available devices. If your jailbroken iOS device is connected, you’ll see it listed.

#### 2. List Running Processes on iOS <a href="#el_1726131485303_784" id="el_1726131485303_784"></a>

To list all running processes on your iOS device, use the **frida-ps** command:

```
frida-ps -U
```

The **-U** flag tells Frida to connect to the iOS device over USB. This will list all processes running on the device, including system processes and apps. Use this to find the process you want to target (e.g., a specific app).

#### 3. Attach Frida to an App <a href="#el_1726131485891_804" id="el_1726131485891_804"></a>

To hook into the **DVIA-v2** app process, use the following command:

```
frida -U -n DVIA-v2
```

This attaches Frida to the running app process so that you can inject scripts.

### Part 3: Using Frida Tools for iOS <a href="#el_1726131899899_1170" id="el_1726131899899_1170"></a>

<figure><img src="../../../../../../.gitbook/assets/image (1446).png" alt=""><figcaption></figcaption></figure>

Frida offers several command-line tools for more advanced dynamic instrumentation without writing full scripts.

<figure><img src="../../../../../../.gitbook/assets/image (1447).png" alt=""><figcaption></figcaption></figure>

#### 1. frida-discover: Discover Methods Dynamically <a href="#el_1727961611341_390" id="el_1727961611341_390"></a>

**frida-discover** is a useful Frida tool that can dynamically discover methods, including Objective-C and Swift methods, in an application. It helps automate the discovery process, allowing you to explore the app's internal functions without prior knowledge of class names or method signatures.\
For example, to discover all called classed and methods (and how many times):

```
frida-discover -U -n DVIA-v2
```

This will list all called classes and methods of the **DVIA-v2** app, making it easier to find what you need to hook or trace.

#### 2. frida-trace: Automatically Trace Function Calls <a href="#el_1726131914402_1186" id="el_1726131914402_1186"></a>

**frida-trace** is a Frida tool that automatically traces function calls, allowing you to see how an app interacts with different system libraries or internal methods.\
\
To trace all classes and/or methods containing a specific string (case-insensitive) with one of the following frida-trace commands:

```
frida-trace -U -n DVIA-v2 -i "*jailbreak*/i"
```

```
frida-trace -U DVIA-v2 -m "*[Jailbreak* *]"
```

This will trace all classes and/or methods containing the term "jailbreak" (case-insensitive) and print the trace information in real-time.

### Part 4: Writing Frida Scripts for iOS <a href="#el_1726131660808_984" id="el_1726131660808_984"></a>

Frida uses JavaScript for injecting code and hooking into app methods. Here's how to write and run some basic scripts for iOS.

#### Example 1: Hooking an Objective-C Method <a href="#el_1726131678949_993" id="el_1726131678949_993"></a>

Suppose you want to hook into an Objective-C method of an iOS app. Here’s a basic example hooking the jailbreak detection method of the **DVIA-v2** app:

```
if (ObjC.classes.JailbreakDetection) {
    var myClass = ObjC.classes.JailbreakDetection;
    var myMethod = myClass["+ isJailbroken"];

    Interceptor.attach(myMethod.implementation, {
        onEnter: function(args) {
            console.log("Hooked ObjC method: JailbreakDetection +isJailbroken");
          	// You can inspect or modify args here
        },
        onLeave: function(retval) {
            console.log("Returned ObjC value:", retval);
          	// You can inspect or modify retval here
        }
    });
} else {
  console.log("Hooking ObjC method failed!");
}
```

This script hooks into the method **isJailbroken** in the class **JailbreakDetection**, logs when the method is called (**onEnter**), and logs the return value (**onLeave**).

#### Step 1: Save the Script <a href="#el_1726131679677_1013" id="el_1726131679677_1013"></a>

Save the above script as **ios\_hook\_objc.js**.

#### Step 2: Run the Script <a href="#el_1726131679988_1023" id="el_1726131679988_1023"></a>

To run the script on the **DVIA-v2** app:

```
frida -U -n DVIA-v2 -l ios_hook_objc.js
```

This attaches the script to the app process and hooks into the specified method.

#### Step 3: Trigger the Hook <a href="#el_1727965095833_764" id="el_1727965095833_764"></a>

Click the **Jailbreak Test 2** within the **DVIA-v2** app to trigger the hook.

#### Example 2: Hooking a Swift Method <a href="#el_1726131680612_1043" id="el_1726131680612_1043"></a>

To hook a Swift method, you first need to know the mangled method name and class signature. Once you've found it (using tools like **frida-discover** or **frida-trace**), you can use Frida to hook into the method.\
\
Here’s an example Frida script to hook into a jailbreak detection method of the **DVIA-v2** app:

```
var myMethod = Module.findExportByName(null, "$s7DVIA_v232JailbreakDetectionViewControllerC12isJailbrokenSbyF");

if (myMethod) {
    Interceptor.attach(myMethod, {
        onEnter: function (args) {
            console.log("Hooked Swift method: JailbreakDetectionViewController +isJailbroken");
          	// You can inspect or modify args here
        },
        onLeave: function (retval) {
            console.log("Returned Swift value:", retval);
          	// You can inspect or modify retval here
        }
    });
} else {
  console.log("Hooking Swift method failed!");
}
```

This script hooks into **NSURLSession** to bypass SSL pinning. It can be useful for intercepting network traffic and analyzing secure API calls.

#### Step 1: Save the Script <a href="#el_1726131776012_1114" id="el_1726131776012_1114"></a>

Save the above script as **ios\_hook\_swift.js**.

#### Step 2: Run the Script <a href="#el_1726131776320_1124" id="el_1726131776320_1124"></a>

To run the script on the **DVIA-v2** app:

```
frida -U -n DVIA-v2 -l ios_hook_swift.js
```

This attaches the script to the app process and hooks into the specified method.<br>

#### Step 3: Trigger the Hook <a href="#el_1727965053158_733" id="el_1727965053158_733"></a>

Click the **Jailbreak Test 1** within the **DVIA-v2** app to trigger the hook.

### Part 5: Advanced iOS Use Cases <a href="#el_1726132009400_1281" id="el_1726132009400_1281"></a>

#### Hooking Swift Methods Dynamically <a href="#el_1726132013350_1285" id="el_1726132013350_1285"></a>

If you want to dynamically explore and hook mangled Swift methods, you can write Frida scripts to search for specific methods of a specific class, then decide which one to hook.\
\
Example of searching mangled Swift methods :

```
const className = "JailbreakDetection".toLowerCase();
const methodName = "isJailbroken".toLowerCase();

function searchSwiftExports(className, methodName) {
    var modules = Process.enumerateModulesSync();
    var found = false;

    modules.forEach(function(module) {
        var moduleExports = Module.enumerateExportsSync(module.name);

        moduleExports.forEach(moduleExport => {
            if (-1 < moduleExport.name.toLowerCase().indexOf(className) < moduleExport.name.toLowerCase().indexOf(methodName)) {
                console.log("Found matching", moduleExport.type, "in module", module.name, ":"+ moduleExport.name, "at", moduleExport.address)
                found = true;
            }
        });
    });

    if (!found) {
        console.log("No matching export found!");
    }
}

searchSwiftExports(className, methodName);
```

Run this script to search for mangled methods that (partially) match class **JailbreakDetection** and method **isJailbroken**, and then you can target specific methods to hook.

> _NOTE: You can also use the Frida's_ [_Swift API_](https://frida.re/docs/swift-api/) _to explore Swift classes and methods, but this API is poorly documented and very buggy._

#### Example: Manipulating Return Values <a href="#el_1726132029737_1311" id="el_1726132029737_1311"></a>

Here’s an example of modifying the return value of a Swift method dynamically:

```
var myMethod = Module.findExportByName(null, "$s7DVIA_v232JailbreakDetectionViewControllerC12isJailbrokenSbyF");

if (myMethod) {
    Interceptor.attach(myMethod, {
        onLeave: function (retval) {
            console.log("Original Swift return value:", retval.toInt32());
          	
            // Modify the return value to 'false' (which is 0)
            retval.replace(0);
            
            console.log("Modified Swift return value to false (0)");
        }
    });
} else {
  console.log("Hooking Swift method failed!");
}
```

This script changes the return value of **isJailbroken** in **JailbreakDetectionViewController** to always return **false**.
