# Skills Assessment

* Run a sub-domain/vhost fuzzing scan on '\*.academy.htb' for the IP shown above. What are all the sub-domains you can identify? (Only write the sub-domain name)

```
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://94.237.55.43:59580/ -H "Host: FUZZ.academy.htb" -fs 985
```

* Before you run your page fuzzing scan, you should first run an extension fuzzing scan. What are the different extensions accepted by the domains?

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://faculty.academy.htb:46674/indexFUZZ -t 40
```

* One of the pages you will identify should say 'You don't have access!'. What is the full page URL?

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -u http://academy.htb:46674/FUZZ -e .phps,.php7,.php,.phps,.phtml,.phar,.html,.htm,.hta -recursion -recursion-depth 2 -t 40 -fs 279
```





* In the page from the previous question, you should be able to find multiple parameters that are accepted by the page. What are they?







* Try fuzzing the parameters you identified for working values. One of them should return a flag. What is the content of the flag?\
