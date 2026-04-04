# Obtaining Session Identifiers without User Interaction

## **Part 1: Simulate the attacker**

Navigate to `http://xss.htb.net` and, using Web Developer Tools (Shift+Ctrl+I in the case of Firefox), notice that the application uses a cookie named `auth-session` most probably as a session identifier.

<figure><img src="../../../.gitbook/assets/image (265).png" alt=""><figcaption></figcaption></figure>

Now fire up Wireshark to start sniffing traffic on the local network as follows.

```shell-session
eldeim@htb[/htb]$ sudo -E wireshark
```

You will come across the below.

<figure><img src="../../../.gitbook/assets/image (266).png" alt=""><figcaption></figcaption></figure>

Right-click "tun0" and then click "Start capture"

## **Part 2: Simulate the victim**

Navigate to `http://xss.htb.net` through a `New Private Window` and login to the application using the credentials below:

* Email: heavycat106
* Password: rocknrol

This is an account that we created to look into the application!

## **Part 3: Obtain the victim's cookie through packet analysis**

Inside Wireshark, first, apply a filter to see only HTTP traffic. This can be done as follows (don't forget to press Enter after specifying the filter).

<figure><img src="../../../.gitbook/assets/image (267).png" alt=""><figcaption></figcaption></figure>

Navigate to `Edit` -> `Find Packet`

<figure><img src="../../../.gitbook/assets/image (268).png" alt=""><figcaption></figcaption></figure>

Left-click on `Packet list` and then click `Packet bytes`

<figure><img src="../../../.gitbook/assets/image (269).png" alt=""><figcaption></figcaption></figure>

Select `String` on the third drop-down menu and specify `auth-session` on the field next to it. Finally, click `Find`. Wireshark will present you with the packets that include an `auth-session` string.

<figure><img src="../../../.gitbook/assets/image (270).png" alt=""><figcaption></figcaption></figure>

The cookie can be copied by right-clicking on a row that contains it, then clicking on `Copy` and finally clicking `Value`.

<figure><img src="../../../.gitbook/assets/image (271).png" alt=""><figcaption></figcaption></figure>

## **Part 4: Hijack the victim's session**

Back to the browser window using which you first browsed the application (not the Private Window), open Web Developer Tools, navigate to _storage_, and change your current cookie's value to the one you obtained through Wireshark (remember to remove the `auth-session=` part).

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
