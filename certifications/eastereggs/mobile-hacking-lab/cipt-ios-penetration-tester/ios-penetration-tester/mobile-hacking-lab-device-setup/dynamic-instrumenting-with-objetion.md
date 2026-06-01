---
description: Guide to Using Objection for Dynamic Instrumentation and Analysis
---

# Dynamic Instrumenting with Objetion

### Prerequisites <a href="#el_1726176653687_732" id="el_1726176653687_732"></a>

* **Jailbroken iOS Device**: Since Objection is built on Frida, it requires a jailbroken iOS device.
* **Python**: Objection is installed via **pip**, so ensure you have Python installed. You can download it from [python.org](https://www.python.org/) or install via **brew**.
* **Frida Installed on the iOS Device**: Ensure you have Frida installed and running on your jailbroken iOS device. Follow the [Frida guide](https://www.mobilehackinglab.com/path-player?courseid=ios-appsec\&unit=66320172d9155074010d39c5) if needed.
* **Target App**: The **DVIA-v2** app should be installed on your device. Download it from the GitHub repository [prateek147/DVIA-v2](https://github.com/prateek147/DVIA-v2).<br>

## Part 1: Setting Up Objection for iOS <a href="#el_1726176509222_396" id="el_1726176509222_396"></a>

### Step 1: Install Objection on Your Computer <a href="#el_1726176520983_401" id="el_1726176520983_401"></a>

Objection can be installed using Python's package manager **pip**. Run the following command in your terminal:

```
pip install objection
```

This installs the **objection** tool on your computer.

### Step 2: Verify Frida is Running on Your iOS Device <a href="#el_1726176522149_425" id="el_1726176522149_425"></a>

1\. **Start Frida on the iOS device** (if not already running):

* Connect to the device via SSH:

```
ssh root@10.11.1.1
```

* Start the Frida server:

```
nohup frida-server &
```

Ensure the Frida server is running in the background on the iOS device.

### Step 3: Ensure App Is Running on the Device <a href="#el_1726177446766_860" id="el_1726177446766_860"></a>

Make sure the target iOS app is running on the device. Objection will inject into a running process to start dynamic instrumentation.

## Part 2: Using Objection for Dynamic Analysis <a href="#el_1726176545785_495" id="el_1726176545785_495"></a>

Objection simplifies many common Frida tasks into straightforward commands. Once you've installed Objection, here are some basic commands to get started.

### Step 1: Attach Objection to the App <a href="#el_1726176545785_497" id="el_1726176545785_497"></a>

Once the app is running, you can start Objection by attaching it to the app process. Run:

```
objection -g DVIA-v2 explore
```

This command launches Objection and opens an interactive shell for live analysis of the app.

### Step 2: Explore the App Environment <a href="#el_1726176545785_503" id="el_1726176545785_503"></a>

Once inside the Objection interactive shell, you can start exploring the app’s environment and runtime behavior.\
\
To get the current environment variables of the app process, use the **env** command:

```
env
```

This will list all the environment variables currently set for the app, which can help in understanding how the app interacts with its environment.\
\
To list the bundles that are loaded by the app, use the following command:

```
ios bundles list_bundles
```

This command will display all bundles within the app, providing insights into the app's structure and configuration.\
To list the frameworks that are used by the app, use:

```
ios bundles list_frameworks
```

This shows all the frameworks loaded by the app, which can be useful for analyzing external dependencies or libraries integrated into the app.

## Part 3: Common Objection Commands for iOS <a href="#el_1726176546369_528" id="el_1726176546369_528"></a>

Here are some common Objection commands to help you quickly perform various tests on an iOS app.

### 1. Bypass SSL pinning <a href="#el_1726176546369_530" id="el_1726176546369_530"></a>

```
ios sslpinning disable
```

This disables SSL pinning in the app, allowing you to intercept network traffic using tools like **Burp Suite** or **mitmproxy**.

### 2. Bypass Jailbreak Detection <a href="#el_1726176546369_534" id="el_1726176546369_534"></a>

Many apps use jailbreak detection to prevent their execution on jailbroken devices. Objection can easily bypass this:

```
ios jailbreak disable
```

This will disable most common jailbreak detection mechanisms in the app.

### 3. Bypass TouchID or FaceID <a href="#el_1726177744395_1044" id="el_1726177744395_1044"></a>

If the app uses TouchID or FaceID for authentication, you can use Objection to bypass these checks:

```
ios ui biometric_bypass
```

This command simulates successful biometric authentication.

### 4. List Loaded Classes <a href="#el_1726176546369_538" id="el_1726176546369_538"></a>

To inspect the Objective-C classes that are loaded by the app, use:

```
ios hooking list classes
```

This will give you a list of all classes currently loaded into the app’s runtime.

### 5. Explore Methods of a Class <a href="#el_1726177628240_974" id="el_1726177628240_974"></a>

To list all methods for the **JailbreakDetection** class, use the following command:

```
ios hooking list class_methods JailbreakDetection
```

### 6. Hook Objective-C Methods <a href="#el_1726177685085_999" id="el_1726177685085_999"></a>

To hook into the **isJailbroken** method of the **JailbreakDetection** class and inspect its parameters or return values, use:

```
ios hooking watch method "+[JailbreakDetection isJailbroken]" --dump-args --dump-return
```

Objection will print out details whenever this method is called during the app’s execution.

## Part 4: Advanced Use Cases <a href="#el_1726176550419_561" id="el_1726176550419_561"></a>

Objection supports more advanced use cases, such as inspecting memory regions, cryptographic functions, and manipulating the app’s runtime environment.

### Example 1: Dumping Keychain Data <a href="#el_1726176550419_563" id="el_1726176550419_563"></a>

To dump all stored Keychain data, use the following command:

```
ios keychain dump
```

This will return a list of all Keychain entries that the app has access to.

### Example 2: Monitoring Cryptographic Functions <a href="#el_1726177872063_1127" id="el_1726177872063_1127"></a>

If you want to monitor cryptographic operations performed by the app, use the following command:

```
ios monitor crypto
```

This command allows you to observe and analyze cryptographic functions such as encryption, decryption, and hashing as they are executed in real time. This can be particularly useful for understanding how the app handles sensitive data like passwords or keys.

### Example 3: Patching a Method at Runtime <a href="#el_1726177885373_1147" id="el_1726177885373_1147"></a>

Objection allows you to modify method implementations in real-time. For example, if you want to modify the return value of a method, you can patch it like this:

```
ios hooking set return_value "+[JailbreakDetection isJailbroken]" false
```

You can modify the method’s implementation to change its behavior or return values dynamically.

### Example 4: Memory Dump and String Search for Hardcoded Secrets <a href="#el_1728062777685_965" id="el_1728062777685_965"></a>

To conduct a thorough search for hardcoded secrets such as API keys, passwords, or sensitive data, you can perform a memory dump and search for strings within the app's memory. This is useful for identifying sensitive information that may be hardcoded or stored insecurely.

1\. **Dumping All Memory Regions**: Use the following command to dump all the memory regions of the app:

```
memory dump all ./process_memory.dmp
```

This will create a dump of all accessible memory regions from the app's process.

2\. **Searching for Specific Strings**: After dumping memory, use the **strings** command in combination with the **grep** command to search for specific keywords, such as "**password**", "**token**", or "**key**":

```
!strings ./process_memory.dmp | grep -i "password"
```

This will search the memory dump for any instances of the keyword "**password**". You can replace "**password**" with any keyword you want to search for, such as "**api\_key**" or "**secret**".

## Part 5: Automating with Objection Scripts <a href="#el_1726176551252_594" id="el_1726176551252_594"></a>

Objection also supports automation by allowing you to create and run custom scripts. You can use these scripts to automate common tasks or analyses across multiple apps.

### Creating an Objection Script <a href="#el_1726178112674_1360" id="el_1726178112674_1360"></a>

Objection scripts are written in plain text and consist of the same commands used in the interactive shell.\
\
Example script (**disable\_security.objection**):

```
ios jailbreak disable
```

```
ios sslpinning disable
```

```
ios ui biometric_bypass
```

### Running an Objection Script <a href="#el_1726178136304_1386" id="el_1726178136304_1386"></a>

To run a script, use the following command:

```
objection -g DVIA-v2 explore --script disable_security.objection
```

This will run the script against the target app.
