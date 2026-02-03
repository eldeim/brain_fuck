# Skills Assessment

* Try to exploit the upload form to read the flag found at the root directory "/".

<figure><img src="../../../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

I can see into the main web a section with name /contact/, in it i can upload a image(screenshot), but i can see into his source code a whitlist -->

<figure><img src="../../../.gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

> Intercept this peticion with burpsuite -->

## XEE

<figure><img src="../../../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

Try to change the extension name with .svg -IT WORK!- Then i need to test the Content-Type of xml - svg

```
cat /usr/share/seclists/Discovery/Web-Content/web-all-content-types.txt | grep svg

image/svg+xml
application/vnd.oipf.dae.svg+xml
```

<figure><img src="../../../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

Into the code, inset xml malicious code -->

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg>&xxe;</svg>
```

<figure><img src="../../../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

With it, can use another malicious codes for view for example the index.php of this web -->

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
<svg>&xxe;</svg>
```

<figure><img src="../../../.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

We can see the content of index.php base64 endoce, with it, we can enumerate anothers endpoint into the web, for example upload.php

<figure><img src="../../../.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

> Decode base64 it&#x20;

<figure><img src="../../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

Allright!! With it we can see, the route of save the files/imgs is `/user_feedback_submissions/`, maybe:  `contact/user_feedback_submissions/` with it, too can see the rename of the images save;

```
$fileName = date('ymd') . '_' . basename($_FILES["uploadFile"]["name"]);code
```

> date(ymd) \_ name of upload file

With it, i can test with a simple upload if i can see the picture -->

<figure><img src="../../../.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

## WebShell Upload

First again, delete de front restriccions -->

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Then, with the burp active, intercept the peticon of upload a image and send to Repeater:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can upload a simple webshell code, but... we need test all casuistics of the content-type, magics numbers and extensions.

First, we modify the extension to see which can send -->

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> ws.phar.jpg - doble extension nice! We can use the intruder

Second, we need imput malius code, for examen a web shell and try to upload. The safest, we can see a normal webshell and need alterate the content-type and mime type -->

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> With it, we can see that the content type correct is jpg o jpeg and the Magic Numbers is the yoya of the jpg iamges

To finaly, we can see the content upload and base64 encoded. And with the another vulnerability XXE, we can view the addres to safe the images: `contact/user_feedback_submissions/date(ymd) _ name of upload file` -->

<figure><img src="../../../.gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>
