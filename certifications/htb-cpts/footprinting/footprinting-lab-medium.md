# Footprinting Lab - Medium

This second server is a server that everyone on the internal network has access to. In our discussion with our client, we pointed out that these servers are often one of the main targets for attackers and that this server should be added to the scope.

Our customer agreed to this and added this server to our scope. Here, too, the goal remains the same. We need to find out as much information as possible about this server and find ways to use it against the server itself. For the proof and protection of customer data, a user named `HTB` has been created. Accordingly, we need to obtain the credentials of this user as proof.

* Enumerate the server carefully and find the username "HTB" and its password. Then, submit this user's password as the answer.

Frist scan TCP ports and then UDP -->

```
nmap -p- --open -sS 10.129.202.41 -Pn -n
PORT      STATE SERVICE
111/tcp   open  rpcbind
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
2049/tcp  open  nfs
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49668/tcp open  unknown
49679/tcp open  unknown
49680/tcp open  unknown
49681/tcp open  unknown

```

```
nmap --top-ports 1000 -sU 10.129.202.41 -Pn -n --min-rate 2000 -D 5
PORT      STATE  SERVICE
111/udp   open   rpcbind
626/udp   closed serialnumberd
2049/udp  open   nfs
16829/udp closed unknown
18004/udp closed unknown
19650/udp closed unknown
```

Now, search versions about the ports founds via TCP -->

```
nmap -p 111,135,445,2049,3389,5985,47001 -sCV 10.129.202.41 -Pn -n --min-rate 1000 -D 5
PORT      STATE SERVICE       VERSION
111/tcp   open  rpcbind       2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|_  100005  1,2,3       2049/udp6  mountd
135/tcp   open  msrpc         Microsoft Windows RPC
445/tcp   open  microsoft-ds?
2049/tcp  open  mountd        1-3 (RPC #100005)
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: WINMEDIUM
|   NetBIOS_Domain_Name: WINMEDIUM
|   NetBIOS_Computer_Name: WINMEDIUM
|   DNS_Domain_Name: WINMEDIUM
|   DNS_Computer_Name: WINMEDIUM
|   Product_Version: 10.0.17763
|_  System_Time: 2025-10-18T18:21:01+00:00
|_ssl-date: 2025-10-18T18:21:08+00:00; -1s from scanner time.
| ssl-cert: Subject: commonName=WINMEDIUM
| Not valid before: 2025-10-17T18:13:49
|_Not valid after:  2026-04-18T18:13:49
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-10-18T18:21:02
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

```

We can see a NFS system, try to get all volumes -->

<figure><img src="../../../.gitbook/assets/image (482).png" alt=""><figcaption></figcaption></figure>

With it, we can observe that exist the folde /TechSupport available for everyone, so... now download it:

```
mkdir ./TechSupport
sudo mount -t nfs 10.129.202.41:/TechSupport ./TechSupport/ -o nolock
```

> Note: We need to be root

<figure><img src="../../../.gitbook/assets/image (483).png" alt=""><figcaption></figcaption></figure>

Exist a lot of files, but... only one contain something -->

<figure><img src="../../../.gitbook/assets/image (484).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (485).png" alt=""><figcaption></figcaption></figure>

&#x20;`alex:lol123!mD`

So... we have credentials and... the port 389 RDP open... soo... try to login -->

```
xfreerdp /u:Alex /p:'lol123!mD' /v:10.129.202.41 /f /clipboard
```

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Onece connected, we can see the user alex and two atyppical folders in the root directory.

I research in devshare and found a credentials -->

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

`sa:87N1ns@slls83`

We have a user sa (i have not idea what it is) and his password, so... after research more, i can found into Desktop, a redirect of SQL Server Login -->

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

An exception error... try to execute it SQL Server with Administrator and the same pass -->

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Nice! We are local admins, so... try to login again. Once seach the pass -->

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
