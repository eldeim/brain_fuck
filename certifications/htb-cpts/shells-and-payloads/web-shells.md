# Web Shells

## Laudanum, One Webshell to Rule Them Al

### Laudanum Demonstration

Now that we understand what Laudanum is and how it works, let's look at a web application we have found in our lab environment and see if we can run a web shell. If you wish to follow along with this demonstration, you will need to add an entry into your `/etc/hosts` file on your attack VM or within Pwnbox for the host we are attacking. That entry should read: `<target ip> status.inlanefreight.local`. Once this is done, you can play and explore this demonstration as long as you are on the VPN or using Pwnbox.

### **Move a Copy for Modification**

```shell-session
eldeim@htb[/htb]$ cp /usr/share/laudanum/aspx/shell.aspx /home/tester/demo.aspx
```

Add your IP address to the `allowedIps` variable on line `59`. Make any other changes you wish. It can be prudent to remove the ASCII art and comments from the file. These items in a payload are often signatured on and can alert the defenders/AV to what you are doing.

### **Modify the Shell for Use**

![The image shows a code snippet in a text editor. It highlights an array of allowed IP addresses, including "10.10.14.12". A yellow arrow points to this line, indicating its significance.](https://academy.hackthebox.com/storage/modules/115/modify-shell.png)

We are taking advantage of the upload function at the bottom of the status page(`Green Arrow`) for this to work. Select your shell file and hit upload. If successful, it should print out the path to where the file was saved (Yellow Arrow). Use the upload function. Success prints out where the file went, navigate to it.

### **Take Advantage of the Upload Function**

![The image shows a server status page with BIOS, disk, and services information. Several services are marked as "Stopped" in red. A section for importing configuration files is highlighted with a yellow arrow pointing to the file path and a green arrow pointing to the "Upload File" button.](https://academy.hackthebox.com/storage/modules/115/laud-upload.png)

Once the upload is successful, you will need to navigate to your web shell to utilize its functions. The image below shows us how to do it. As seen from the last image, our shell was uploaded to the `\\files\` directory, and the name was kept the same. This won't always be the case. You may run into some implementations that randomize filenames on upload that do not have a public files directory or any number of other potential safeguards. For now, we are lucky that's not the case. With this particular web application, our file went to `status.inlanefreight.local\\files\demo.aspx` and will require us to browse for the upload by using that \ in the path instead of the / like normal. Once you do this, your browser will clean it up in your URL window to appear as `status.inlanefreight.local//files/demo.aspx`.

### **Navigate to Our Shell**

![The image shows a Laundanum ASPX Shell interface with a command input field labeled "cmd /c" and a "Submit Query" button. A green arrow points to the URL "status.inlanefreight.local/files/demo.aspx" in the browser's address bar.](https://academy.hackthebox.com/storage/modules/115/laud-nav.png)

We can now utilize the Laudanum shell we uploaded to issue commands to the host. We can see in the example that the `systeminfo` command was run.

### Lab - Questions

* Establish a web shell session with the target using the concepts covered in this section. Submit the full path of the directory you land in. (Format: c:\path\you\land\in)

First copy the webshell .aspx and upload into the website, before that search dir the /&#x20;

```
eldeim@htb[/htb]$ cp /usr/share/laudanum/aspx/shell.aspx /home/tester/demo.aspx
```

<figure><img src="../../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Where is the Laudanum aspx web shell located on Pwnbox? Submit the full path. (Format: /path/to/laudanum/aspx)

```
/usr/share/laudanum/aspx/shell.aspx
```

***

## Antak Webshell

One great resource to use in learning is `IPPSEC's` blog site [ippsec.rocks](https://ippsec.rocks/?). The site is a powerful learning tool.&#x20;

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Working with Antak

The Antak files can be found in the `/usr/share/nishang/Antak-WebShell` directory.

```shell-session
eldeim@htb[/htb]$ ls /usr/share/nishang/Antak-WebShell

antak.aspx  Readme.md
```

Antak web shell functions like a Powershell Console. However, it will execute each command as a new process. It can also execute scripts in memory and encode commands you send. As a web shell, Antak is a pretty powerful tool.

### **Move a Copy for Modification**

```shell-session
eldeim@htb[/htb]$ cp /usr/share/nishang/Antak-WebShell/antak.aspx /home/administrator/Upload.aspx
```

Make sure you set credentials for access to the web shell. Modify `line 14`, adding a user (green arrow) and password (orange arrow). This comes into play when you browse to your web shell, much like Laudanum. This can help make your operations more secure by ensuring random people can't just stumble into using the shell. It can be prudent to remove the ASCII art and comments from the file. These items in a payload are often signatured on and can alert the defenders/AV to what you are doing.

### **Modify the Shell for Use**

![The image shows a code snippet with a conditional statement checking if the username is "Disclaimer" and the password is "ForLegitUseOnly". If true, it sets execution visibility and enables it. Green and orange arrows highlight the username and password conditions.](https://academy.hackthebox.com/storage/modules/115/antak-changes.png)

For the sake of demonstrating the tool, we are uploading it to the same status portal we used for Laudanum. That host was a Windows host, so our shell should work just fine with PowerShell. Upload the file and then navigate to the page for use. It will give you a user and password prompt. Remember, with this web application, the files are stored in the `\\files\` directory. When you navigate to the `upload.aspx` file, you should see a prompt as we have below.

### **Shell Success**

![The image shows a login form for the Antak Webshell with fields for username and password, both filled with "htb-student". A "Login" button is present below the fields. The URL in the browser's address bar is "status.inlanefreight.local/files/upload.aspx".](https://academy.hackthebox.com/storage/modules/115/antak-creds-prompt.png)

As seen in the following image, we will be granted access if our credentials are entered properly.

![The image shows the Antak Webshell interface with a blue command prompt area displaying the message: "Welcome to Antak - A Webshell which utilizes PowerShell. Use help for more details. Use clear to clear the screen." Below are buttons labeled "Submit," "Browse," "Upload the File," "Encode and Execute," "Download," "Parse web.config," "Execute SQL Query," and a field for entering a connection string.](https://academy.hackthebox.com/storage/modules/115/antak-success.png)

Now that we have access, we can utilize PowerShell commands to navigate and take actions against the host

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Lab - Questions

* Where is the Antak webshell located on Pwnbox? Submit the full path. (Format:/path/to/antakwebshell)

```
/usr/share/nishang/Antak-WebShell/antak.aspx
```

* Establish a web shell with the target using the concepts covered in this section. Submit the name of the user on the target that the commands are being issued as. In order to get the correct answer you must navigate to the web shell you upload using the vHost name. (Format: \*\***\***\*, 1 space

Upload de the webshell, and after target this we can see a login endpoint, to access of that, we need the credentials that are in the file -->

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

***

## PHP Web Shells

### Hands-on With a PHP-Based Web Shell.

<figure><img src="https://academy.hackthebox.com/storage/modules/115/vendors_tab.png" alt=""><figcaption></figcaption></figure>

We will be using [WhiteWinterWolf's PHP Web Shell](https://github.com/WhiteWinterWolf/wwwolf-php-webshell). We can download this or copy and paste the source code into a `.php` file. Keep in mind that the file type is significant, as we will soon witness. Our goal is to upload the PHP web shell via the Vendor Logo `browse` button. Attempting to do this initially will fail since rConfig is checking for the file type. It will only allow uploading image file types (.png,.jpg,.gif, etc.). However, we can bypass this utilizing `Burp Suite`.

Start Burp Suite, navigate to the browser's network settings menu and fill out the proxy settings. `127.0.0.1` will go in the IP address field, and `8080` will go in the port field to ensure all requests pass through Burp (recall that Burp acts as the web proxy).

### **Proxy Settings**

![Proxy settings dialog with options for no proxy, auto-detect, system settings, and manual configuration with HTTP proxy set to 127.0.0.1:8080.](https://academy.hackthebox.com/storage/modules/115/proxy_settings.png)

Our goal is to change the `content-type` to bypass the file type restriction in uploading files to be "presented" as the vendor logo so we can navigate to that file and have our web shell.

> Note: `Firefox` removed `FTP` support starting with [version 90](https://www.mozilla.org/en-US/firefox/90.0/releasenotes/).

### Bypassing the File Type Restriction

With Burp open and our web browser proxy settings properly configured, we can now upload the PHP web shell. Click the browse button, navigate to wherever our .php file is stored on our attack box, and select open and `Save` (we may need to accept the PortSwigger Certificate). It will seem as if the web page is hanging, but that's just because we need to tell Burp to forward the HTTP requests. Forward requests until you see the POST request containing our file upload. It will look like this:

#### **Post Request**

![Burp Suite showing intercepted HTTP request with headers and PHP code snippet.](https://academy.hackthebox.com/storage/modules/115/burp.png)

As mentioned in an earlier section, you will notice that some payloads have comments from the author that explain usage, provide kudos and links to personal blogs. This can give us away, so it's not always best to leave the comments in place. We will change Content-type from `application/x-php` to `image/gif`. This will essentially "trick" the server and allow us to upload the .php file, bypassing the file type restriction. Once we do this, we can select `Forward` twice, and the file will be submitted. We can turn the Burp interceptor off now and go back to the browser to see the results.

#### **Vendor Added**

![Vendor management page showing added vendor 'NetVen' with options to add, edit, or remove vendors, and a table listing 'Cisco' and 'NetVen'.](https://academy.hackthebox.com/storage/modules/115/added_vendor.png)

The message: `Added new vendor NetVen to Database` lets us know our file upload was successful. We can also see the NetVen vendor entry with the logo showcasing a ripped piece of paper. This means rConfig did not recognize the file type as an image, so it defaulted to that image. We can now attempt to use our web shell. Using the browser, navigate to this directory on the rConfig server:

`/images/vendor/connect.php`

This executes the payload and provides us with a non-interactive shell session entirely in the browser, allowing us to execute commands on the underlying OS.

#### **Webshell Success**

![Web interface for fetching files with fields for host, port, path, command execution, and sudo output showing allowed commands.](https://academy.hackthebox.com/storage/modules/115/web_shell_now.png)

### Lab - Questions

* In the example shown, what must the Content-Type be changed to in order to successfully upload the web shell? (Format: .../... )

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

* Use what you learned from the module to gain a web shell. What is the file name of the gif in the /images/vendor directory on the target? (Format: xxxx.gif)

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
