# Attacking Common Services

## Interacting with Common Services

### &#x20;**Windows CMD - Findstr**

```cmd-session
c:\htb>findstr /s /i cred n:\*.*

n:\Contracts\private\secret.txt:file with all credentials
n:\Contracts\private\credentials.txt:admin:SecureCredentials!
```

We can find more `findstr` examples [here](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/findstr#examples).

### **Linux - Mount**

```shell-session
eldeim@htb[/htb]$ sudo mkdir /mnt/Finance
eldeim@htb[/htb]$ sudo mount -t cifs -o username=plaintext,password=Password123,domain=. //192.168.220.129/Finance /mnt/Finance
```

As an alternative, we can use a credential file.

> Note: We need to install `cifs-utils` to connect to an SMB share folder. To install it we can execute from the command line `sudo apt install cifs-utils`.

```shell-session
eldeim@htb[/htb]$ mount -t cifs //192.168.220.129/Finance /mnt/Finance -o credentials=/path/credentialfile
```

The file `credentialfile` has to be structured like this:

```txt
username=plaintext
password=Password123
domain=.
```

### **Linux - Find**

```shell-session
[!bash!]$ find /mnt/Finance/ -name *cred*

/mnt/Finance/Contracts/private/credentials.txt
```

Next, let's find files that contain the string `cred`:

```shell-session
[!bash!]$ grep -rn /mnt/Finance/ -ie cred

/mnt/Finance/Contracts/private/credentials.txt:1:admin:SecureCredentials!
/mnt/Finance/Contracts/private/secret.txt:1:file with all credentials
```

## Command Line Utilities - **MySQL**

### **Linux - SQSH**

```shell-session
eldeim@htb[/htb]$ sqsh -S 10.129.20.13 -U username -P Password123
```

The `sqlcmd` utility lets you enter Transact-SQL statements, system procedures, and script files through a variety of available modes:

* At the command prompt.
* In Query Editor in SQLCMD mode.
* In a Windows script file.
* In an operating system (Cmd.exe) job step of a SQL Server Agent job.

### **Windows - SQLCMD**

```cmd-session
C:\htb> sqlcmd -S 10.129.20.13 -U username -P Password123
```

To learn more about `sqlcmd` usage, you can see [Microsoft documentation](https://docs.microsoft.com/en-us/sql/ssms/scripting/sqlcmd-use-the-utility).

### **Linux - MySQL**

```shell-session
eldeim@htb[/htb]$ mysql -u username -pPassword123 -h 10.129.20.13
```

We can easily start an interactive SQL Session using Windows:

### **Windows - MySQL**

```cmd-session
C:\htb> mysql.exe -u username -pPassword123 -h 10.129.20.13
```

***

## **Tools to Interact with Common Services**

| **SMB**                                                                                  | **FTP**                                     | **Email**                                          | **Databases**                                                                                                                |
| ---------------------------------------------------------------------------------------- | ------------------------------------------- | -------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| [smbclient](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html)          | [ftp](https://linux.die.net/man/1/ftp)      | [Thunderbird](https://www.thunderbird.net/en-US/)  | [mssql-cli](https://github.com/dbcli/mssql-cli)                                                                              |
| [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec)                              | [lftp](https://lftp.yar.ru/)                | [Claws](https://www.claws-mail.org/)               | [mycli](https://github.com/dbcli/mycli)                                                                                      |
| [SMBMap](https://github.com/ShawnDEvans/smbmap)                                          | [ncftp](https://www.ncftp.com/)             | [Geary](https://wiki.gnome.org/Apps/Geary)         | [mssqlclient.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/mssqlclient.py)                             |
| [Impacket](https://github.com/SecureAuthCorp/impacket)                                   | [filezilla](https://filezilla-project.org/) | [MailSpring](https://getmailspring.com/)           | [dbeaver](https://github.com/dbeaver/dbeaver)                                                                                |
| [psexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/psexec.py)   | [crossftp](http://www.crossftp.com/)        | [mutt](http://www.mutt.org/)                       | [MySQL Workbench](https://dev.mysql.com/downloads/workbench/)                                                                |
| [smbexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbexec.py) |                                             | [mailutils](https://mailutils.org/)                | [SQL Server Management Studio or SSMS](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms) |
|                                                                                          |                                             | [sendEmail](https://github.com/mogaal/sendemail)   |                                                                                                                              |
|                                                                                          |                                             | [swaks](http://www.jetmore.org/john/code/swaks/)   |                                                                                                                              |
|                                                                                          |                                             | [sendmail](https://en.wikipedia.org/wiki/Sendmail) |                                                                                                                              |

### Service Misconfigurations

```shell-session
admin:admin
admin:password
admin:<blank>
root:12345678
administrator:Password
```
