# Skills Assessment

* Try to escalate your privileges and exploit different vulnerabilities to read the flag at '/flag.php'.

To login in the panel, i can see a uid indentifier -->

<figure><img src="../../../.gitbook/assets/image (209).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (210).png" alt=""><figcaption></figcaption></figure>

I modify it for example `uid=1` -->

<figure><img src="../../../.gitbook/assets/image (211).png" alt=""><figcaption></figcaption></figure>

Then login i can see another peticion with uid nad user url uid, change it by 1 for example -->

<figure><img src="../../../.gitbook/assets/image (212).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (213).png" alt=""><figcaption></figcaption></figure>

OKAY, i am another user, i will go to reload the profile web and intercept another this peticion for see mor info about others users -->

<figure><img src="../../../.gitbook/assets/image (214).png" alt=""><figcaption></figcaption></figure>

OKAY, i can enumerate user with this uid, go to intruder -->

<figure><img src="../../../.gitbook/assets/image (215).png" alt=""><figcaption></figcaption></figure>

okayy!!! user with uid==52 is Administrator,s e that -->

<figure><img src="../../../.gitbook/assets/image (217).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (218).png" alt=""><figcaption></figcaption></figure>

Okay, i can only see this, true. WHO I CAN BE ADMIN USER??

In my user, i have a section of change my password, go to intercept it -->

<figure><img src="../../../.gitbook/assets/image (219).png" alt=""><figcaption></figcaption></figure>

Allright, first it call a /api.php/tohen, and it send my uid too -->

<figure><img src="../../../.gitbook/assets/image (220).png" alt=""><figcaption></figcaption></figure>

OKAY, the sen mi token user and uid with the new password and send by POST to /reset.php. Now, modify it again -->

In the first peticon to /api.php/token, modify the uid to admin==52 -->

<figure><img src="../../../.gitbook/assets/image (221).png" alt=""><figcaption></figcaption></figure>

He give me his token user, nice: `{"token":"e51a85fa-17ac-11ec-8e51-e78234eb7b0c"}` COPY IT

<figure><img src="../../../.gitbook/assets/image (222).png" alt=""><figcaption></figcaption></figure>

After alterate all camps, give me an error "Acces Denied" .. F\&CK U! So.. i will ty to `Change request method` -->

<figure><img src="../../../.gitbook/assets/image (223).png" alt=""><figcaption></figcaption></figure>

OKAY! F\&cking http verb tampening ... Now log in to Administrador

<figure><img src="../../../.gitbook/assets/image (224).png" alt=""><figcaption></figcaption></figure>

Intercep the peticon and chang the uid by 52 -->

<figure><img src="../../../.gitbook/assets/image (225).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (226).png" alt=""><figcaption></figcaption></figure>

I can see a category with name, add event, so...

<figure><img src="../../../.gitbook/assets/image (227).png" alt=""><figcaption></figcaption></figure>

Now intercept it to see the body -->

<figure><img src="../../../.gitbook/assets/image (228).png" alt=""><figcaption></figcaption></figure>

I can see a XML struccture, and lohh0 reflected, so now i will try to read an internal file -->

<figure><img src="../../../.gitbook/assets/image (229).png" alt=""><figcaption></figcaption></figure>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE name [
  <!ENTITY company "Inlane Freight">
]>
            <root>
            <name>&company;</name>
            <details>test2</details>
            <date>2002-02-12</date>
            </root>
```

<figure><img src="../../../.gitbook/assets/image (230).png" alt=""><figcaption></figcaption></figure>

So... with it i can read the flag -->

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=/flag.php">
]>
            <root>
            <name>&company;</name>
            <details>test2</details>
            <date>2002-02-12</date>
            </root>
```

<figure><img src="../../../.gitbook/assets/image (231).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (232).png" alt=""><figcaption></figcaption></figure>
