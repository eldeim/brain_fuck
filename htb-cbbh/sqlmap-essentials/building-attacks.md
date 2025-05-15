# Building Attacks

## Curl Commands

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

```shell-session
eldeim@htb[/htb]$ sqlmap 'http://www.example.com/?id=1' -H 'User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0' -H 'Accept: image/webp,*/*' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Connection: keep-alive' -H 'DNT: 1'
```

## GET/POST Requests

```shell-session
eldeim@htb[/htb]$ sqlmap 'http://www.example.com/' --data 'uid=1&name=test'
## or
eldeim@htb[/htb]$ sqlmap 'http://www.example.com/' --data 'uid=1*&name=test'
```

## Custom SQLMap Requests

```shell-session
elde
```

The same effect can be done with the usage of option `-H/--header`:

```shell-session
eldeim@htb[/htb]$ sqlmap ... --cookie='PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'
```

For example, if there is a requirement to specify the (session) cookie value to `PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c` option `--cookie` would be used as follows:

\
\
