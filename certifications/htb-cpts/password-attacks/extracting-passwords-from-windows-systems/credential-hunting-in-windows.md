# Credential Hunting in Windows

### **Key terms to search for**

Whether we end up with access to the GUI or CLI, we know we will have some tools to use for searching but of equal importance is what exactly we are searching for. Here are some helpful key terms we can use that can help us discover some credentials:

* Passwords
* Passphrases
* Keys
* Username
* User account
* Creds
* Users
* Passkeys
* configuration
* dbcredential
* dbpassword
* pwd
* Login
* Credentials

Let's use some of these key terms to search on the IT admin's workstation.

### **Windows Search**

With access to the GUI, it is worth attempting to use `Windows Search` to find files on the target using some of the keywords mentioned above.

<figure><img src="../../../../.gitbook/assets/image (505).png" alt=""><figcaption></figcaption></figure>

## **LaZagne**

We can also take advantage of third-party tools like [LaZagne](https://github.com/AlessandroZ/LaZagne) to quickly discover credentials that web browsers or other installed applications may insecurely store. LaZagne is made up of `modules` which each target different software when looking for passwords. Some of the common modules are described in the table below:

| Module   | Description                                                                                       |
| -------- | ------------------------------------------------------------------------------------------------- |
| browsers | Extracts passwords from various browsers including Chromium, Firefox, Microsoft Edge, and Opera   |
| chats    | Extracts passwords from various chat applications including Skype                                 |
| mails    | Searches through mailboxes for passwords including Outlook and Thunderbird                        |
| memory   | Dumps passwords from memory, targeting KeePass and LSASS                                          |
| sysadmin | Extracts passwords from the configuration files of various sysadmin tools like OpenVPN and WinSCP |
| windows  | Extracts Windows-specific credentials targeting LSA secrets, Credential Manager, and more         |
| wifi     | Dumps WiFi credentials                                                                            |

> Note: Web browsers are some of the most interesting places to search for credentials, due to the fact that many of them offer built-in credential storage. In the most popular browsers, such as `Google Chrome`, `Microsoft Edge`, and `Firefox`, stored credentials are encrypted. However, many tools for decrypting the various credentials databases used can be found online, such as [firefox\_decrypt](https://github.com/unode/firefox_decrypt) and [decrypt-chrome-passwords](https://github.com/ohyicong/decrypt-chrome-passwords). LaZagne supports `35` different browsers on Windows.

It would be beneficial to keep a [standalone copy](https://github.com/AlessandroZ/LaZagne/releases/) of LaZagne on our attack host so we can quickly transfer it over to the target. `LaZagne.exe` will do just fine for us in this scenario. We can use our RDP client to copy the file over to the target from our attack host. If we are using `xfreerdp` all we must do is copy and paste into the RDP session we have established.

Once `LaZagne.exe` is on the target, we can open command prompt or PowerShell, navigate to the directory the file was uploaded to, and execute the following command:

```cmd-session
C:\Users\bob\Desktop> start LaZagne.exe all
```

This will execute LaZagne and run `all` included modules. We can include the option `-vv` to study what it is doing in the background. Once we hit enter, it will open another prompt and display the results.

```cmd-session
|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|


########## User: bob ##########

------------------- Winscp passwords -----------------

[+] Password found !!!
URL: 10.129.202.51
Login: admin
Password: SteveisReallyCool123
Port: 22
```

If we used the `-vv` option, we would see attempts to gather passwords from all LaZagne's supported software. We can also look on the GitHub page under the supported software section to see all the software LaZagne will try to gather credentials from. It may be a bit shocking to see how easy it can be to obtain credentials in clear text. Much of this can be attributed to the insecure way many applications store credentials.

## **findstr**

We can also use [findstr](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/findstr) to search from patterns across many types of files. Keeping in mind common key terms, we can use variations of this command to discover credentials on a Windows target:

&#x20; Credential Hunting in Windows

```cmd-session
C:\> findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```

### Additional considerations

There are thousands of tools and key terms we could use to hunt for credentials on Windows operating systems. Know that which ones we choose to use will be primarily based on the function of the computer. If we land on a Windows Server, we may use a different approach than if we land on a Windows Desktop. Always be mindful of how the system is being used, and this will help us know where to look. Sometimes we may even be able to find credentials by navigating and listing directories on the file system as our tools run.

Here are some other places we should keep in mind when credential hunting:

* Passwords in Group Policy in the SYSVOL share
* Passwords in scripts in the SYSVOL share
* Password in scripts on IT shares
* Passwords in `web.config` files on dev machines and IT shares
* Password in `unattend.xml`
* Passwords in the AD user or computer description fields
* KeePass databases (if we are able to guess or crack the master password)
* Found on user systems and shares
* Files with names like `pass.txt`, `passwords.docx`, `passwords.xlsx` found on user systems, shares, and [Sharepoint](https://www.microsoft.com/en-us/microsoft-365/sharepoint/collaboration)

***

### Lab - Questions

> RDP to 10.129.202.99 (ACADEMY-PWATTACKS-WIN10CHUNTING) with user "Bob" and password "HTB\_@cademy\_stdnt!"

* What password does Bob use to connect to the Switches via SSH? (Format: Case-Sensitive)

Fristly, connect via rdp to the victim machine at the same time as i share a folder -->

```
xfreerdp /u:Bob /p:"HTB_@cademy_stdnt!" /v:10.129.202.99 /drive:share,/home /clipboard
```

We can see that creds -->

<figure><img src="../../../../.gitbook/assets/image (506).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (507).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (508).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (509).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (510).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (511).png" alt=""><figcaption></figcaption></figure>



* What credentials does Bob use with WinSCP to connect to the file server? (Format: username:password, Case-Sensitive)

Thanks to share via RDP, we are share us folder... so...

<figure><img src="../../../../.gitbook/assets/image (512).png" alt=""><figcaption></figcaption></figure>

Now need download the lazagne and upload to this machine -->

```
sudo wget https://github.com/AlessandroZ/LaZagne/releases/latest/download/LaZagne.exe -O LaZagne.exe
```

Into the windows, move the .exe and run

```
start Lazagne.exe all
```

<figure><img src="../../../../.gitbook/assets/image (513).png" alt=""><figcaption></figcaption></figure>
