# JWTs

## Attacking Signature Verification

### Missing Signature Verification

Before jumping into the attack, let us look at our target web application. Starting our target and accessing the provided URL, we are greeted with a simple login page:

<figure><img src="../../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

We can use the provided credentials to log in to the web application, which displays an almost empty page:

<figure><img src="../../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

Due to the message `You are not an admin!`, we can infer that there are users with different privilege levels. Let us investigate if we can find a way to escalate our privileges to an administrator to see if this will display more information to us.

As we can see in the response to a successful login request, the web application uses a JWT as our session cookie to identify our user:

<figure><img src="../../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

The JWT contains our username, an `isAdmin` claim, and an expiry timestamp. Since our goal is to escalate our privileges to an administrator, the `isAdmin` claim seems to be an obvious way to achieve that goal. We can simply manipulate that parameter in the payload, and `jwt.io` will automatically re-encode the JWT on the left side. However, as discussed previously, this will invalidate the JWT's signature.

This is where our first attack comes into play. Before accepting a JWT, the web application must verify the JWT's signature to ensure it has not been tampered with. If the web application is misconfigured to accept JWTs without verifying their signature, we can manipulate our JWT to escalate privileges.

Due to recent update of `jwt.io`, certain actions have been limited, such as editing the contents (payload) of the JWT token directly. As an alternative, [jwt.lannysport.net](https://jwt.lannysport.net/) can be used.

To achieve this, let us change the `isAdmin` parameter's value to `true` in `jwt.io`:

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

We can then pass the manipulated JWT in the `session` cookie in the request to `/home`:

```http
GET /home HTTP/1.1
Host: 172.17.0.2
Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6dHJ1ZSwiZXhwIjoxNzExMTg2MDQ0fQ.S85PjpnL6BNhBCWk6OYDHc_XjfWogMJV8wq5pKJ6Tv4
```

Since the web application does not verify the JWT's signature, it will grant us admin access:

<figure><img src="../../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

Just like before, we can then pass the manipulated JWT in the `session` cookie in the request to `/home`:

```http
GET /home HTTP/1.1
Host: 172.17.0.2
Cookie: session=eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6dHJ1ZSwiZXhwIjoxNzExMTg2MDQ0LCJpYXQiOjE3MTExODY0NTJ9.

Since the web application accepts the JWT with the none algorithm, it will grant us admin access:
```

Since the web application accepts the JWT with the `none` algorithm, it will grant us admin access:

<figure><img src="../../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

> Note: Even though the JWT does not contain a signature, the final period (`.`) still needs to be present.

***

### Labs - Questions

* Escalate your privileges to obtain the flag

<figure><img src="../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

First, login into the web and see we arent admin user, and read us JWT token -->

<figure><img src="../../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6ZmFsc2UsImV4cCI6MTc1OTE2ODMwMn0.XIYDwO686JUF7AuOL5j2izk8WggqlSxYifIRkCpJ4Tw
```

Now, read it into [https://jwt.lannysport.net/](https://jwt.lannysport.net/)

<figure><img src="../../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

Change isAdmin same True, copy this JWT and paste into the new in the website and reload

<figure><img src="../../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

We can see a errror, so... with this changes, create a new into cybercheft with null sing -->

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

***

## Attacking the Signing Secret

\
