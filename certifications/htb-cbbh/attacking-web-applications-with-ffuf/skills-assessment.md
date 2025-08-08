# Skills Assessment

* Run a sub-domain/vhost fuzzing scan on '\*.academy.htb' for the IP shown above. What are all the sub-domains you can identify? (Only write the sub-domain name)

```
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://94.237.55.43:59580/ -H "Host: FUZZ.academy.htb" -fs 985
```

* Before you run your page fuzzing scan, you should first run an extension fuzzing scan. What are the different extensions accepted by the domains?

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://faculty.academy.htb:46674/indexFUZZ -t 40
```

* One of the pages you will identify should say 'You don't have access!'. What is the full page URL?

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://faculty.academy.htb:57821/FUZZ -recursion -recursion-depth 1 -e .php,.phps,.php7 -v -t 80  -fs 287
```

* In the page from the previous question, you should be able to find multiple parameters that are accepted by the page. What are they?

> First one with GET

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://faculty.academy.htb:57821/courses/linux-security.php7?FUZZ=key -fs 774 -t 40
```

> Then with POST

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://faculty.academy.htb:57821/courses/linux-security.php7 -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs 774 -t 40
```

* Try fuzzing the parameters you identified for working values. One of them should return a flag. What is the content of the flag?

```
ffuf -w /opt/useful/seclists/Usernames/xato-net-10-million-usernames.txt:FUZZ -u http://faculty.academy.htb:57821/courses/linux-security.php7 -X POST -d 'username=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -t 40 -fs 781
## Then get the flag
curl -X POST http://faculty.academy.htb:57821/courses/linux-security.php7 -d 'username=harry'
```
