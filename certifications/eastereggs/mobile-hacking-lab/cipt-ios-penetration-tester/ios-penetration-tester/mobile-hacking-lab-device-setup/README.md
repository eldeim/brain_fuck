# Mobile Hacking Lab Device Setup

There are several iOS hacking labs for you to practice which are also related to the attack surface section of this course. You can find them [here](https://www.mobilehackinglab.com/free-mobile-hacking-labs).

* To connect to the virtual device in any iOS lab, first open the lab page starting with iOS device'

<figure><img src="https://lwfiles.mycourse.app/63942c32c9a203516ce07c09-public/6806bfb80d79017f4c56b2c14845047b.png" alt=""><figcaption></figcaption></figure>

* Next, click on Connect on the left to open the Connect page

<figure><img src="https://lwfiles.mycourse.app/63942c32c9a203516ce07c09-public/93d1904dfe646bb820686410003d0b8c.png" alt=""><figcaption></figcaption></figure>

* Download the OVPN File provided and scroll down to see download options for USBFlux and download the appropriate version of USBFlux.

<figure><img src="https://lwfiles.mycourse.app/63942c32c9a203516ce07c09-public/b3f4f05f03a3ef700161b8f0d5583fca.png" alt=""><figcaption></figcaption></figure>

* Next install **USBFLux**\
  \- If you are using MacOS, the installation is pretty straightforward with drag n drop.\
  \- For Linux, you can download the binary for your architecture from [https://github.com/corellium/usbfluxd/releases/tag/v1.0](https://github.com/corellium/usbfluxd/releases/tag/v1.0%C2%A0) and add it to your path as described here [https://github.com/corellium/usbfluxd?tab=readme-ov-file#installation](https://%20https//github.com/corellium/usbfluxd?tab=readme-ov-file#installation)&#x20;
* The next step is to connect to the VPN. For this you can utilize OpenVPN or Tunnelblick in MacOS and load the OVPN file.
* On Linux you can start openVPN via:

```
sudo openvpn device.ovpn
```

* If the VPN is working you can start USBFluxd on the foreground on Linux via:

```
sudo systemctl start usbmuxd
```

```
sudo avahi-daemon
```

```
sudo usbfluxd -f -n
```

* or using the GUI application on Mac OS

### **Bonus: On MacOS the device is also visible if you open Xcode**

Once connected, you will see the device in the Manage Run Destinations window of Xcode. The pin is 000000, wait for a while and it should start uploading build cache to the device. If it does not do anything, restart the XCode.\
\
And if you already have frida installed ([https://frida.re/docs/installation/](https://frida.re/docs/installation/)) you can run:

```
frida-ps -U
```

```
frida-ps -Ua
```

Which will give you a list of (active) applications on your iPhone and confirms the USB connection works.
