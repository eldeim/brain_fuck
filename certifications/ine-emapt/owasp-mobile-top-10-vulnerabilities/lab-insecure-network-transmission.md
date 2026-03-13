# LAB - Insecure Network Transmission

In this lab environment, you will have GUI access to a Debian machine. The **InsecureBankv2** application is available on the Android Emulator.

**Objective:** Intercept the app traffic with Burp Suite to capture sensitive credentials transmitted in plaintext.

The valid credentials for InsecureBankv2 are as follows:

* **Username:** jack
* **Password:** Jack@123$

> **Note:** You can start the emulator using the script located on the Desktop. Additionally, check the **/root/Tools** directory for available tools.

***

### Tools

The best tools for this lab are:

* Burp Suite

***

Frist excute the emulator with `./emulator`, then the app and it request us credentials -->

<figure><img src="../../../.gitbook/assets/image (354).png" alt=""><figcaption></figcaption></figure>

> This application relies on a back-end server to function properly

To start the back-end server, run the following command:

```bash
## Wake up a server with python tool
cd /root/Tools/AndroLabServer
python2.7 app.py
```

<figure><img src="../../../.gitbook/assets/image (355).png" alt=""><figcaption></figcaption></figure>

Then, into the app go to the "Preferences" settings -->

<figure><img src="../../../.gitbook/assets/image (356).png" alt=""><figcaption></figcaption></figure>

Enter the IP address of the host machine where the back-end server is running, then click **Submit**

> **Note:** (use the `ifconfig` command to find the IP address). Use the device’s on-screen keyboard to enter your input.

<figure><img src="../../../.gitbook/assets/image (357).png" alt=""><figcaption></figcaption></figure>

Now, try to log in and if all configurations are good, we can see the user and password in text plain, try (test:test)

<figure><img src="../../../.gitbook/assets/image (358).png" alt=""><figcaption></figcaption></figure>

### BurpSuite Configuration

So... now, we can try to intercept all trafict with burp -->

<figure><img src="../../../.gitbook/assets/image (359).png" alt=""><figcaption></figcaption></figure>

Here, we need remplace the localhost ip (127.0.0.1) to us IP -->

<figure><img src="../../../.gitbook/assets/image (361).png" alt=""><figcaption></figcaption></figure>

Now, we need to add a proxy on the Android device. Open **Settings** and click on **Network & internet**.

<figure><img src="../../../.gitbook/assets/image (18) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now, turn on the **Intercept** in Burp and navegate/login -->

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
