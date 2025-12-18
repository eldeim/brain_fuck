# Footprinting Lab - Easy

We were commissioned by the company `Inlanefreight Ltd` to test three different servers in their internal network. The company uses many different services, and the IT security department felt that a penetration test was necessary to gain insight into their overall security posture.

The first server is an internal DNS server that needs to be investigated. In particular, our client wants to know what information we can get out of these services and how this information could be used against its infrastructure. Our goal is to gather as much information as possible about the server and find ways to use that information against the company. However, our client has made it clear that it is forbidden to attack the services aggressively using exploits, as these services are in production.

Additionally, our teammates have found the following credentials "ceil:qwer1234", and they pointed out that some of the company's employees were talking about SSH keys on a forum.

The administrators have stored a `flag.txt` file on this server to track our progress and measure success. Fully enumerate the target and submit the contents of this file as proof.

* Enumerate the server carefully and find the flag.txt file. Submit the contents of this file as the answer.

Frist, i perform a quick scanner via TCP to identify all opens ports -->

```
nmap -p- --open -sS 10.129.42.195 -Pn -n
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
53/tcp   open  domain
2121/tcp open  ccproxy-ftp
```

Now, see his versions and other things...

```
nmap -p 21,22,53,2121 -sCV 10.129.42.195 -Pn -n
PORT     STATE SERVICE VERSION
21/tcp   open  ftp
| fingerprint-strings: 
|   GenericLines: 
|     220 ProFTPD Server (ftp.int.inlanefreight.htb) [10.129.42.195]
|     Invalid command: try being more creative
|_    Invalid command: try being more creative
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3f:4c:8f:10:f1:ae:be:cd:31:24:7c:a1:4e:ab:84:6d (RSA)
|   256 7b:30:37:67:50:b9:ad:91:c0:8f:f7:02:78:3b:7c:02 (ECDSA)
|_  256 88:9e:0e:07:fe:ca:d0:5c:60:ab:cf:10:99:cd:6c:a7 (ED25519)
53/tcp   open  domain  ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
2121/tcp open  ftp
| fingerprint-strings: 
|   GenericLines: 
|     220 ProFTPD Server (Ceil's FTP) [10.129.42.195]
|     Invalid command: try being more creative
|_    Invalid command: try being more creative

```

Another thing we can perform, there is a UDP scan -->

```
nmap --top-ports 5000 --open -sU 10.129.42.195 -Pn -n --min-rate 1000
PORT    STATE SERVICE
53/udp  open  domain
623/udp open  asf-rmcp
```

> Note: I already looked the anonymous ftp

So... We have credentials, now try to log in via ftp access -->

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Into the port 21, we havent anything, so... exist another port, there is 2121, try to login -->

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can see a .ssh folder, perharps we can optain the id\_rsa -->

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Nice, after download it, set the necessary permissions with chmod and connect with user ceil -->

```
chmod 600 id_rsa
ssh ceil@10.129.42.195 -I id_rsa
```

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
