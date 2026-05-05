---
description: Guide to Using Burp Suite for Traffic Interception and Analysis
---

# Intercepting iOS Network Traffic with Burp Suite

<figure><img src="../../../../../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

### Prerequisites <a href="#el_1726180098966_453" id="el_1726180098966_453"></a>

* **Burp Suite**: You can download Burp Suite from [PortSwigger's website](https://portswigger.net/burp) (Community or Professional Edition) or install via **brew**.
* **Jailbroken or Non-Jailbroken iOS Device**: You can use Burp Suite for testing both jailbroken and non-jailbroken iOS devices.
* **Wi-Fi Network**: Both your iOS device and the computer running Burp Suite must be connected to the same Wi-Fi network.
* **Burp Suite CA Certificate**: To intercept HTTPS traffic, you’ll need to install Burp's Certificate Authority (CA) certificate on your iOS device.

***

## Part 1: Configuring Burp Suite for iOS Traffic Interception <a href="#el_1726180436803_470" id="el_1726180436803_470"></a>

### Step 1: Set Up Burp Suite Proxy <a href="#el_1726180649321_568" id="el_1726180649321_568"></a>

1\. Open Burp Suite and go to the **Proxy** tab.\
2\. Click **Options** and verify that a listener is running on port **8080** (default setting) or any port of your choice. Ensure that **"All interfaces"** is selected in the Bind to address field.

> This allows Burp Suite to listen for traffic coming from any device on the same network.

<figure><img src="../../../../../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

### Step 1.1: Enable VPN <a href="#el_1726180746608_583" id="el_1726180746608_583"></a>

In my case, I will to connect my host windows pc to the VPN "ovpn" and startup the burp in windows, eveything else, i will do into my wsl.

<figure><img src="../../../../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

> Note: We can see the ip (10.11.3.2)

### Step 1.2: Set IP iOS device in BurpSuite <a href="#el_1726180746608_583" id="el_1726180746608_583"></a>

<figure><img src="../../../../../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

### Step 2: Configure iOS Device Proxy Settings <a href="#el_1726180746608_583" id="el_1726180746608_583"></a>

You need to configure your iOS device to route its traffic through Burp Suite.\
1\. On your iOS device, go to **Settings > Wi-Fi**.\
2\. Tap the **i** icon next to your connected Wi-Fi network.

<figure><img src="../../../../../.gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure>

\
3\. Scroll down to **HTTP Proxy** and set it to **Manual**.\
4\. Enter the following details:

* **Server**: The IP address of your computer running Burp Suite (you can find it by running **ifconfig** or **ipconfig** on your computer).
* **Port**: The port Burp Suite is listening on (default is **8080**).

<figure><img src="../../../../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

## Part 2: Installing Burp Suite CA Certificate <a href="#el_1726180466117_481" id="el_1726180466117_481"></a>

To intercept HTTPS traffic, Burp Suite needs to act as a "man-in-the-middle" (MITM) proxy, and for that, your iOS device needs to trust Burp's CA certificate.\
\
Follow the manual from Portswigger: https://portswigger.net/burp/documentation/desktop/getting-started/intercepting-http-traffic for this or execute below steps:

#### Step 1: Download the Burp CA Certificate on the iOS Device <a href="#el_1726181008990_636" id="el_1726181008990_636"></a>

1\. On your iOS device, open **Safari** and navigate to:

```
http://burp
```

2\. This will automatically download the Burp CA certificate (named **cacert.der**).

<figure><img src="../../../../../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

> Note: All it into us navegator

#### Step 2: Install the CA Certificate <a href="#el_1726181015701_656" id="el_1726181015701_656"></a>

1\. After downloading, navigate to **Settings > General > VPN & Device Management** (or **Profiles & Device Management** depending on the iOS version).

2\. You should see the **Burp Suite Professional CA** profile listed. Tap on it and install the certificate.

3\. Go to **Settings > General > About > Certificate Trust Settings**.

<figure><img src="../../../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

Now, or upload the certf previusly download or, in the safary browser search and download again the certf -->

<figure><img src="../../../../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

4\. Enable full trust for **Burp Suite Professional CA** by toggling the switch.

<figure><img src="../../../../../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

For end, we need activated ir in about -> "Cetificate Trust Setings"

<figure><img src="../../../../../.gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (59).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

## Part 3: Intercepting HTTP/HTTPS Traffic <a href="#el_1726180484502_491" id="el_1726180484502_491"></a>

Once the proxy is set up and the CA certificate is installed, Burp Suite will begin intercepting all HTTP and HTTPS traffic from the iOS device.

#### Step 1: Enable Interception  <a href="#el_1726181105164_691" id="el_1726181105164_691"></a>

1\. In Burp Suite, go to the **Proxy** tab.2. Ensure the **Intercept** button is turned on.3. Open the app you want to test on your iOS device.\
Now, you should see requests from the app appearing in Burp Suite's intercept tab, allowing you to view and manipulate HTTP/HTTPS requests and responses.

## Part 4: Common Burp Suite Tasks <a href="#el_1726180518245_523" id="el_1726180518245_523"></a>

#### 1. Testing API Calls and Parameters <a href="#el_1726181179805_752" id="el_1726181179805_752"></a>

You can view and manipulate the API requests made by the app to the backend server.\
1\. Open the **HTTP history** tab under **Proxy** in Burp Suite.2. Find the relevant API requests and inspect the parameters, headers, and data being sent.3. Modify the request (e.g., change parameters or tokens) and forward it to observe how the server responds.

#### 2. Replay or Modify Requests  <a href="#el_1726181405446_803" id="el_1726181405446_803"></a>

Burp Suite allows you to replay captured requests or modify them to test for vulnerabilities.1. Right-click on any request in the HTTP history and select **Send to Repeater**.\
2\. In the **Repeater** tab, you can manually modify and re-send the request, testing for various vulnerabilities like IDOR (Insecure Direct Object Reference), authentication bypass, or broken access control.

#### 3. Intercepting Mobile App Login Requests <a href="#el_1726181496101_832" id="el_1726181496101_832"></a>

Intercept login requests from the mobile app to capture or manipulate authentication tokens or credentials. This is especially useful for testing how the app handles login, session management, and authentication mechanisms.1. Log in to the app while Burp is intercepting the traffic.2. In Burp Suite’s HTTP history, search for the login request. Review the data being sent, including credentials, tokens, and session information.

## Part 5: Bypassing SSL Pinning <a href="#el_1726180502633_507" id="el_1726180502633_507"></a>

Many iOS apps implement SSL pinning, which prevents interception of HTTPS traffic, even with the CA certificate installed. Here’s how you can bypass SSL pinning:

#### Method 1: Using Frida and Objection (for Jailbroken Devices)  <a href="#el_1726181144243_709" id="el_1726181144243_709"></a>

You can use **Frida** or **Objection** to dynamically bypass SSL pinning in iOS apps. See the [Frida Guide](https://www.mobilehackinglab.com/path-player?courseid=ios-appsec\&unit=66320172d9155074010d39c5) or the [Objection Guide](https://www.mobilehackinglab.com/path-player?courseid=ios-appsec\&unit=66dee992414c21bf3d053855) for more details on how to disable SSL pinning using these tools.\
Once SSL pinning is bypassed, Burp Suite will be able to intercept HTTPS traffic from the app.

#### Method 2: Using SSL Kill Switch 2 (for Jailbroken Devices)  <a href="#el_1726181159657_719" id="el_1726181159657_719"></a>

For jailbroken iOS devices, **SSL Kill Switch 2** is a popular tweak that disables SSL pinning across apps:1. Install **SSL Kill Switch 2** via **Cydia** or **Sileo** on your jailbroken iOS device.2. Once installed, it automatically disables SSL pinning in most apps.\
\
After enabling SSL Kill Switch, restart the app and Burp Suite will intercept the traffic.

#### Method 3: Patch the App (Non-Jailbroken Devices)  <a href="#el_1726181165760_729" id="el_1726181165760_729"></a>

For non-jailbroken devices, you can use reverse engineering techniques to modify the app’s binary and disable SSL pinning directly. This requires tools like **Hopper** or **Ghidra** to analyze and modify the app.

## Part 6: Using Burp Extensions for Mobile App Testing <a href="#el_1726180592791_538" id="el_1726180592791_538"></a>

Burp Suite has a marketplace called **BApp Store**, which contains various extensions to enhance your testing experience. Some useful extensions for mobile app testing include:

#### 1. MobileAssistant <a href="#el_1726181577338_850" id="el_1726181577338_850"></a>

This extension helps in testing mobile applications by providing shortcuts for common tasks such as inspecting **plist files**, checking **iOS keychain data**, and **parsing mobile API requests**.\
To install MobileAssistant:1. Go to the **Extensions** tab in Burp Suite.2. Click **BApp Store**.3. Search for **MobileAssistant** and install it.

#### 2. Logger++ <a href="#el_1726181633461_860" id="el_1726181633461_860"></a>

Logger++ is a powerful extension for logging and analyzing HTTP/S traffic. It can help you visualize how the mobile app interacts with the backend.\
To install Logger++:1. Go to the **Extensions** tab in Burp Suite.2. Click **BApp Store**.3. Search for **Logger++** and install it.

#### 3. JSON Beautifier <a href="#el_1726181663988_876" id="el_1726181663988_876"></a>

This extension automatically formats JSON requests and responses to make them more readable.\
To install JSON Beautifier:1. Go to the Extensions tab in Burp Suite.2. Click **BApp Store**.3. Search for **JSON Beautifier** and install it.

## Part 7: Automating Security Tests with Burp Suite <a href="#el_1726180602505_549" id="el_1726180602505_549"></a>

Burp Suite offers features for automating certain security tests to help discover vulnerabilities faster.

#### 1. Burp Scanner (Professional Edition)  <a href="#el_1726181724564_894" id="el_1726181724564_894"></a>

Burp Suite’s scanner automatically scans captured traffic for security vulnerabilities, such as SQL Injection, XSS, and insecure communications. To use it:\
1\. Send the captured request to the **Scanner** by right-clicking and selecting **Send to Scanner**.\
2\. Burp Suite will run an automated scan and report any vulnerabilities it discovers.

#### 2. Intruder <a href="#el_1726181754159_904" id="el_1726181754159_904"></a>

The **Intruder** tool allows you to automate attacks like fuzzing, brute force, or parameter tampering.\
\
To use the Intruder:1. Right-click on a request and select **Send to Intruder**.2. Configure the payloads (e.g., usernames or passwords) and run the attack.3. Burp will try all the payloads and report the results.
