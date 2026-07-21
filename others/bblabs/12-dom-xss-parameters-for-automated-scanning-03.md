---
description: >-
  Red social con 12 parametros diferentes inyectables en el DOM via distintos
  sinks (innerHTML, document.write, eval, outerHTML, setTimeout,
  location.replace). Ideal para practicar con herramientas auto
---

# 12 DOM XSS Parameters for Automated Scanning  - 03

***

### Tools

* dalfox
* DevTools of browser
* Burp

***

#### Credentials

alice:password123 | rol=user

***

### Vulnerability Table

| #  | Parameter    | Type      | Injection Context              | Feature                         |
| -- | ------------ | --------- | ------------------------------ | ------------------------------- |
| 1  | `?q=`        | Reflected | `HTML body`                    | Search results heading          |
| 2  | `?ref=`      | Reflected | `HTML body`                    | Referral banner                 |
| 3  | `?msg=`      | Reflected | `HTML body`                    | Notification banner             |
| 4  | `?tab=`      | Reflected | `HTML body`                    | Tab indicator                   |
| 5  | `?next=`     | Reflected | `href attribute`               | Continue link (javascript: XSS) |
| 6  | `?user=`     | Reflected | `HTML body (inside <strong>)`  | Welcome back greeting           |
| 7  | `#fragment`  | DOM XSS   | `innerHTML (client-side)`      | Section navigation              |
| 8  | `?debug=`    | Reflected | `Script string → eval()`       | Hidden debug mode               |
| 9  | `?sort=`     | Reflected | `HTML body`                    | Sort order indicator            |
| 10 | `?lang=`     | Reflected | `Script string → setTimeout()` | Language selector               |
| 11 | `?redirect=` | Reflected | `href attribute + JS redirect` | Post-action redirect            |
| 12 | `?title=`    | Reflected | `<title> tag`                  | Dynamic page title              |

### Automated Scanning with dalfox

[dalfox](https://github.com/hahwul/dalfox) is an automated XSS scanner. Since the server reflects parameter values into the HTML response, dalfox can detect the injection points.

* Scan all parameters at once:

```
dalfox url "http://localhost:1000/?q=test&ref=x&msg=x&tab=x&sort=x&user=x&lang=x&next=x&debug=x&redirect=x&title=x"
```

* Scan individual parameters:`dalfox url "http://localhost:1000/?q=test" -p q`&#x20;
* Save results to file:`dalfox url "http://localhost:1000/?q=test&ref=x&msg=x" --output results.json --format json`

***

## Usage Dalfox - Real commands

```
dalfox url --url "https://g6ae68f4qq.lab.bblabs.es//?q=test&ref=x&msg=x&tab=x&sort=x&user=x&lang=x&next=x&debug=x&redirect=x&title=x" --cookies "token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwidXNlcm5hbWUiOiJhbGljZSIsImlhdCI6MTc4....."
```

<figure><img src="../../.gitbook/assets/image (1529).png" alt=""><figcaption></figcaption></figure>
