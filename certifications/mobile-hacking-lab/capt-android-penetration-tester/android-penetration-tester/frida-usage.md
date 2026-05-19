# Frida Usage

<figure><img src="../../../../.gitbook/assets/image (1461).png" alt=""><figcaption></figcaption></figure>

| Commands                              | Executable                                                       |
| ------------------------------------- | ---------------------------------------------------------------- |
| <p>frida-ps -Ua<br>frida-ps -Uai</p> | Get a list of active and installed applications via **frida-ps** |
| frida-trace -i "open" -U              | Use frida-trace to trace the app                                 |

***

## Frida CodeShare

### Install .apk

Install the app on your connected Android device:

```
abd install –r [filename].apk
```

> Be configurated burp previusly

### Weak up Frida Server

```
adb root
adb shell "/data/local/tmp/frida-server &"
```

> If u us MAGISK, it isnt necesary

### Bypass SSL pinning via the CodeShare script:

{% embed url="https://codeshare.frida.re/@akabe1/frida-multiple-unpinning/" %}

```
frida-ps -Ua
frida --codeshare akabe1/frida-multiple-unpinning -U -p [pid]
```

## Others Frida Scripts

{% embed url="https://codeshare.frida.re/" %}

<figure><img src="../../../../.gitbook/assets/image (1462).png" alt=""><figcaption></figcaption></figure>

