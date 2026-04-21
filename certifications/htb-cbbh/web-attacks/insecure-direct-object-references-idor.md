# Insecure Direct Object References (IDOR)

`Insecure Direct Object References (IDOR)` vulnerabilities are among the most common web vulnerabilities and can significantly impact the vulnerable web application. IDOR vulnerabilities occur when a web application exposes a direct reference to an object, like a file or a database resource, which the end-user can directly control to obtain access to other similar objects. If any user can access any resource due to the lack of a solid access control system, the system is considered to be vulnerable.

For example, if users request access to a file they recently uploaded, they may get a link to it such as (`download.php?file_id=123`). So, as the link directly references the file with (`file_id=123`), what would happen if we tried to access another file (which may not belong to us) with (`download.php?file_id=124`)? If the web application does not have a proper access control system on the back-end, we may be able to access any file by sending a request with its `file_id`. In many cases, we may find that the `id` is easily guessable, making it possible to retrieve many files or resources that we should not have access to based on our permissions.

### URL Parameters & APIs

The very first step of exploiting IDOR vulnerabilities is identifying Direct Object References. Whenever we receive a specific file or resource, we should study the HTTP requests to look for URL parameters or APIs with an object reference (e.g. `?uid=1` or `?filename=file_1.pdf`). These are mostly found in URL parameters or APIs but may also be found in other HTTP headers, like cookies.

In the most basic cases, we can try incrementing the values of the object references to retrieve other data, like (`?uid=2`) or (`?filename=file_2.pdf`). We can also use a fuzzing application to try thousands of variations and see if they return any data. Any successful hits to files that are not our own would indicate an IDOR vulnerability.

## Mass IDOR Enumeration

### Insecure Parameters

Let's start with a basic example that showcases a typical IDOR vulnerability. The exercise below is an `Employee Manager` web application that hosts employee records:

<figure><img src="../../../.gitbook/assets/image (1040).png" alt=""><figcaption></figcaption></figure>

Our web application assumes that we are logged in as an employee with user id `uid=1` to simplify things. This would require us to log in with credentials in a real web application, but the rest of the attack would be the same. Once we click on `Documents`, we are redirected to

`/documents.php`:

<figure><img src="../../../.gitbook/assets/image (1041).png" alt=""><figcaption></figcaption></figure>

When we get to the `Documents` page, we see several documents that belong to our user. These can be files uploaded by our user or files set for us by another department (e.g., HR Department). Checking the file links, we see that they have individual names:

```html
/documents/Invoice_1_09_2021.pdf
/documents/Report_1_10_2021.pdf
```

When we try changing the `uid` to `?uid=2`, we don't notice any difference in the page output, as we are still getting the same list of documents, and may assume that it still returns our own documents:

<figure><img src="../../../.gitbook/assets/image (1041).png" alt=""><figcaption></figcaption></figure>

However, `we must be attentive to the page details during any web pentest` and always keep an eye on the source code and page size. If we look at the linked files, or if we click on them to view them, we will notice that these are indeed different files, which appear to be the documents belonging to the employee with `uid=2`:

```html
/documents/Invoice_2_08_2020.pdf
/documents/Report_2_12_2020.pdf
```

This is a common mistake found in web applications suffering from IDOR vulnerabilities, as they place the parameter that controls which user documents to show under our control while having no access control system on the back-end. Another example is using a filter parameter to only display a specific user's documents (e.g. `uid_filter=1`), which can also be manipulated to show other users' documents or even completely removed to show all documents at once.

### Mass Enumeration

We can try manually accessing other employee documents with `uid=3`, `uid=4`, and so on. However, manually accessing files is not efficient in a real work environment with hundreds or thousands of employees. So, we can either use a tool like `Burp Intruder` or `ZAP Fuzzer` to retrieve all files or write a small bash script to download all files, which is what we will do.

We can click on \[`CTRL+SHIFT+C`] in Firefox to enable the `element inspector`, and then click on any of the links to view their HTML source code, and we will get the following:

```html
<li class='pure-tree_link'><a href='/documents/Invoice_3_06_2020.pdf' target='_blank'>Invoice</a></li>
<li class='pure-tree_link'><a href='/documents/Report_3_01_2020.pdf' target='_blank'>Report</a></li>
```

We can pick any unique word to be able to `grep` the link of the file. In our case, we see that each link starts with `<li class='pure-tree_link'>`, so we may `curl` the page and `grep` for this line, as follows:

```shell-session
eldeim@htb[/htb]$ curl -s "http://SERVER_IP:PORT/documents.php?uid=3" | grep "<li class='pure-tree_link'>"

<li class='pure-tree_link'><a href='/documents/Invoice_3_06_2020.pdf' target='_blank'>Invoice</a></li>
<li class='pure-tree_link'><a href='/documents/Report_3_01_2020.pdf' target='_blank'>Report</a></li>
```

As we can see, we were able to capture the document links successfully. We may now use specific bash commands to trim the extra parts and only get the document links in the output. However, it is a better practice to use a `Regex` pattern that matches strings between `/document` and `.pdf`, which we can use with `grep` to only get the document links, as follows:

```shell-session
eldeim@htb[/htb]$ curl -s "http://SERVER_IP:PORT/documents.php?uid=3" | grep -oP "\/documents.*?.pdf"

/documents/Invoice_3_06_2020.pdf
/documents/Report_3_01_2020.pdf
```

Now, we can use a simple `for` loop to loop over the `uid` parameter and return the document of all employees, and then use `wget` to download each document link:

```bash
#!/bin/bash

url="http://SERVER_IP:PORT"

for i in {1..10}; do
        for link in $(curl -s "$url/documents.php?uid=$i" | grep -oP "\/documents.*?.pdf"); do
                wget -q $url/$link
        done
done
```

When we run the script, it will download all documents from all employees with `uids` between 1-10, thus successfully exploiting the IDOR

### PoCs - Questions

* Repeat what you learned in this section to get a list of documents of the first 20 user uid's in /documents.php, one of which should have a '.txt' file with the flag.

If I intercept the main web peticion /documents.php, we can see a uid -->

<figure><img src="../../../.gitbook/assets/image (1042).png" alt=""><figcaption></figcaption></figure>

Go to intruder and ffuf by this uid -->

<figure><img src="../../../.gitbook/assets/image (1043).png" alt=""><figcaption></figcaption></figure>

After the attack, i can see the biggest peticion length and see the doc flag, gg

## Bypassing Encoded References

In the previous section, we saw an example of an IDOR that uses employee uids in clear text, making it easy to enumerate. In some cases, web applications make hashes or encode their object references, making enumeration more difficult, but it may still be possible.

Let's go back to the `Employee Manager` web application to test the `Contracts` functionality:

<figure><img src="../../../.gitbook/assets/image (1044).png" alt=""><figcaption></figcaption></figure>

If we click on the `Employment_contract.pdf` file, it starts downloading the file. The intercepted request in Burp looks as follows

<figure><img src="../../../.gitbook/assets/image (1045).png" alt=""><figcaption></figcaption></figure>

We see that it is sending a `POST` request to `download.php` with the following data:

Code: php

```php
contract=cdd96d3cc73d1dbdaffa03cc6cd7339b
```

Using a `download.php` script to download files is a common practice to avoid directly linking to files, as that may be exploitable with multiple web attacks. In this case, the web application is not sending the direct reference in cleartext but appears to be hashing it in an `md5` format. Hashes are one-way functions, so we cannot decode them to see their original values.

We can attempt to hash various values, like `uid`, `username`, `filename`, and many others, and see if any of their `md5` hashes match the above value. If we find a match, then we can replicate it for other users and collect their files. For example, let's try to compare the `md5` hash of our `uid`, and see if it matches the above hash:

```shell-session
eldeim@htb[/htb]$ echo -n 1 | md5sum

c4ca4238a0b923820dcc509a6f75849b -
```

Unfortunately, the hashes do not match. We can attempt this with various other fields, but none of them matches our hash. In advanced cases, we may also utilize `Burp Comparer` and fuzz various values and then compare each to our hash to see if we find any matches. In this case, the `md5` hash could be for a unique value or a combination of values, which would be very difficult to predict, making this direct reference a `Secure Direct Object Reference`

### Function Disclosure

As most modern web applications are developed using JavaScript frameworks, like `Angular`, `React`, or `Vue.js`, many web developers may make the mistake of performing sensitive functions on the front-end, which would expose them to attackers.This function appears to be sending a `POST` request with the `contract` parameter, which is what we saw above. The value it is sending is an `md5` hash using the `CryptoJS` library, which also matches the request we saw earlier. So, the only thing left to see is what value is being hashed.

In this case, the value being hashed is `btoa(uid)`, which is the `base64` encoded string of the `uid` variable, which is an input argument for the function. Going back to the earlier link where the function was called, we see it calling `downloadContract('1')`. So, the final value being used in the `POST` request is the `base64` encoded string of `1`, which was then `md5` hashed.

We can test this by `base64` encoding our `uid=1`, and then hashing it with `md5`, as follows:

```shell-session
eldeim@htb[/htb]$ echo -n 1 | base64 -w 0 | md5sum

cdd96d3cc73d1dbdaffa03cc6cd7339b -
```

> Tip: We are using the `-n` flag with `echo`, and the `-w 0` flag with `base64`, to avoid adding newlines, in order to be able to calculate the `md5` hash of the same value, without hashing newlines, as that would change the final `md5` hash.

### Mass Enumeration

We can start by calculating the hash for each of the first ten employees using the same previous command while using `tr -d` to remove the trailing `-` characters, as follows:

```shell-session
eldeim@htb[/htb]$ for i in {1..10}; do echo -n $i | base64 -w 0 | md5sum | tr -d ' -'; done

cdd96d3cc73d1dbdaffa03cc6cd7339b
0b7e7dee87b1c3b98e72131173dfbbbf
0b24df25fe628797b3a50ae0724d2730
f7947d50da7a043693a592b4db43b0a1
8b9af1f7f76daf0f02bd9c48c4a2e3d0
006d1236aee3f92b8322299796ba1989
b523ff8d1ced96cef9c86492e790c2fb
d477819d240e7d3dd9499ed8d23e7158
3e57e65a34ffcb2e93cb545d024f5bde
5d4aace023dc088767b4e08c79415dcd
```

Next, we can make a `POST` request on `download.php` with each of the above hashes as the `contract` value, which should give us our final script:

Code: bash

```bash
#!/bin/bash

for i in {1..10}; do
    for hash in $(echo -n $i | base64 -w 0 | md5sum | tr -d ' -'); do
        curl -sOJ -X POST -d "contract=$hash" http://SERVER_IP:PORT/download.php
    done
done
```

With that, we can run the script, and it should download all contracts for employees 1-10:

Bypassing Encoded References

```shell-session
eldeim@htb[/htb]$ bash ./exploit.sh
eldeim@htb[/htb]$ ls -1

contract_006d1236aee3f92b8322299796ba1989.pdf
contract_0b24df25fe628797b3a50ae0724d2730.pdf
contract_0b7e7dee87b1c3b98e72131173dfbbbf.pdf
contract_3e57e65a34ffcb2e93cb545d024f5bde.pdf
contract_5d4aace023dc088767b4e08c79415dcd.pdf
contract_8b9af1f7f76daf0f02bd9c48c4a2e3d0.pdf
contract_b523ff8d1ced96cef9c86492e790c2fb.pdf
contract_cdd96d3cc73d1dbdaffa03cc6cd7339b.pdf
contract_d477819d240e7d3dd9499ed8d23e7158.pdf
contract_f7947d50da7a043693a592b4db43b0a1.pdf
```

PoCs - Questions

* Try to download the contracts of the first 20 employee, one of which should contain the flag, which you can read with 'cat'. You can either calculate the 'contract' parameter value, or calculate the '.pdf' file name directly.

To intercept the contract pdf peticon, we can see a uid encode with, url -> base64 -->

<figure><img src="../../../.gitbook/assets/image (1046).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1047).png" alt=""><figcaption></figcaption></figure>

Go to intruder and configurate the numbers of 0-20 == 21, run list and then base64 endoce + url and send the peticion -->

<figure><img src="../../../.gitbook/assets/image (1048).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1049).png" alt=""><figcaption></figcaption></figure>

## IDOR in Insecure APIs

While `IDOR Information Disclosure Vulnerabilities` allow us to read various types of resources, `IDOR Insecure Function Calls` enable us to call APIs or execute functions as another user.

### Identifying Insecure APIs

Going back to our `Employee Manager` web application, we can start testing the `Edit Profile` page for IDOR vulnerabilities:

<figure><img src="../../../.gitbook/assets/image (1040).png" alt=""><figcaption></figcaption></figure>

When we click on the `Edit Profile` button, we are taken to a page to edit information of our user profile, namely `Full Name`, `Email`, and `About Me`, which is a common feature in many web applications:

<figure><img src="../../../.gitbook/assets/image (1050).png" alt=""><figcaption></figcaption></figure>

We can change any of the details in our profile and click `Update profile`, and we'll see that they get updated and persist through refreshes, which means they get updated in a database somewhere. Let's intercept the `Update` request in Burp and look at it:

<figure><img src="../../../.gitbook/assets/image (1051).png" alt=""><figcaption></figcaption></figure>

We see that the page is sending a `PUT` request to the `/profile/api.php/profile/1` API endpoint. `PUT` requests are usually used in APIs to update item details, while `POST` is used to create new items, `DELETE` to delete items, and `GET` to retrieve item details. So, a `PUT` request for the `Update profile` function is expected. The interesting bit is the JSON parameters it is sending:

```json
{
    "uid": 1,
    "uuid": "40f5888b67c748df7efba008e7c2f9d2",
    "role": "employee",
    "full_name": "Amy Lindon",
    "email": "a_lindon@employees.htb",
    "about": "A Release is like a boat. 80% of the holes plugged is not good enough."
}
```

We see that the `PUT` request includes a few hidden parameters, like `uid`, `uuid`, and most interestingly `role`, which is set to `employee`. The web application also appears to be setting the user access privileges (e.g. `role`) on the client-side, in the form of our `Cookie: role=employee` cookie, which appears to reflect the `role` specified for our user. This is a common security issue. The access control privileges are sent as part of the client's HTTP request, either as a cookie or as part of the JSON request, leaving it under the client's control, which could be manipulated to gain more privileges.

So, unless the web application has a solid access control system on the back-end, `we should be able to set an arbitrary role for our user, which may grant us more privileges`. However, how would we know what other roles exist?

### Exploiting Insecure APIs

We know that we can change the `full_name`, `email`, and `about` parameters, as these are the ones under our control in the HTML form in the `/profile` web page. So, let's try to manipulate the other parameters.

There are a few things we could try in this case:

1. Change our `uid` to another user's `uid`, such that we can take over their accounts
2. Change another user's details, which may allow us to perform several web attacks
3. Create new users with arbitrary details, or delete existing users
4. Change our role to a more privileged role (e.g. `admin`) to be able to perform more actions

Let's start by changing our `uid` to another user's `uid` (e.g. `"uid": 2`). However, any number we set other than our own `uid` gets us a response of `uid mismatch`:

<figure><img src="../../../.gitbook/assets/image (1052).png" alt=""><figcaption></figcaption></figure>

The web application appears to be comparing the request's `uid` to the API endpoint (`/1`). This means that a form of access control on the back-end prevents us from arbitrarily changing some JSON parameters, which might be necessary to prevent the web application from crashing or returning errors.

Perhaps we can try changing another user's details. We'll change the API endpoint to `/profile/api.php/profile/2`, and change `"uid": 2` to avoid the previous `uid mismatch`:

<figure><img src="../../../.gitbook/assets/image (1053).png" alt=""><figcaption></figcaption></figure>

Next, let's see if we can create a new user with a `POST` request to the API endpoint. We can change the request method to `POST`, change the `uid` to a new `uid`, and send the request to the API endpoint of the new `uid`:

We get an error message saying `Creating new employees is for admins only`. The same thing happens when we send a `Delete` request, as we get `Deleting employees is for admins only`. The web application might be checking our authorization through the `role=employee` cookie because this appears to be the only form of authorization in the HTTP request.

Finally, let's try to change our `role` to `admin`/`administrator` to gain higher privileges. Unfortunately, without knowing a valid `role` name, we get `Invalid role` in the HTTP response, and our `role` does not update:

<figure><img src="../../../.gitbook/assets/image (1054).png" alt=""><figcaption></figcaption></figure>

So, `all of our attempts appear to have failed`. We cannot create or delete users as we cannot change our `role`. We cannot change our own `uid`, as there are preventive measures on the back-end that we cannot control, nor can we change another user's details for the same reason. `So, is the web application secure against IDOR attacks?`.

So far, we have only been testing the `IDOR Insecure Function Calls`. However, we have not tested the API's `GET` request for `IDOR Information Disclosure Vulnerabilities`. If there was no robust access control system in place, we might be able to read other users' details, which may help us with the previous attacks we attempted.

`Try to test the API against IDOR Information Disclosure vulnerabilities by attempting to get other users' details with GET requests`. If the API is vulnerable, we may be able to leak other users' details and then use this information to complete our IDOR attacks on the function calls.

### PoCs - Questions

* Try to read the details of the user with 'uid=5'. What is their 'uuid' value?

I interrcept the enter perticion for us info profile and we can see your uid, we can modify this parameter:

<figure><img src="../../../.gitbook/assets/image (1055).png" alt=""><figcaption></figcaption></figure>

## Chaining IDOR Vulnerabilities

Usually, a `GET` request to the API endpoint should return the details of the requested user, so we may try calling it to see if we can retrieve our user's details. We also notice that after the page loads, it fetches the user details with a `GET` request to the same API endpoint:

<figure><img src="../../../.gitbook/assets/image (1056).png" alt=""><figcaption></figcaption></figure>

As mentioned in the previous section, the only form of authorization in our HTTP requests is the `role=employee` cookie, as the HTTP request does not contain any other form of user-specific authorization, like a JWT token, for example.

### Information Disclosure

Let's send a `GET` request with another `uid`:

<figure><img src="../../../.gitbook/assets/image (1057).png" alt=""><figcaption></figcaption></figure>

As we can see, this returned the details of another user, with their own `uuid` and `role`, confirming an `IDOR Information Disclosure vulnerability`:

Code: json

```json
{
    "uid": "2",
    "uuid": "4a9bd19b3b8676199592a346051f950c",
    "role": "employee",
    "full_name": "Iona Franklyn",
    "email": "i_franklyn@employees.htb",
    "about": "It takes 20 years to build a reputation and few minutes of cyber-incident to ruin it."
}
```

This provides us with new details, most notably the `uuid`, which we could not calculate before, and thus could not change other users' details.

### Modifying Other Users' Details

Now, with the user's `uuid` at hand, we can change this user's details by sending a `PUT` request to `/profile/api.php/profile/2` with the above details along with any modifications we made, as follows:

<figure><img src="../../../.gitbook/assets/image (1058).png" alt=""><figcaption></figcaption></figure>

We don't get any access control error messages this time, and when we try to `GET` the user details again, we see that we did indeed update their details:

<figure><img src="../../../.gitbook/assets/image (1059).png" alt=""><figcaption></figcaption></figure>

### Chaining Two IDOR Vulnerabilities

Since we have identified an IDOR Information Disclosure vulnerability, we may also enumerate all users and look for other `roles`, ideally an admin role. `Try to write a script to enumerate all users, similarly to what we did previously`.

Once we enumerate all users, we will find an admin user with the following details:

```json
{
    "uid": "X",
    "uuid": "a36fa9e66e85f2dd6f5e13cad45248ae",
    "role": "web_admin",
    "full_name": "administrator",
    "email": "webadmin@employees.htb",
    "about": "HTB{FLAG}"
}
```

We may modify the admin's details and then perform one of the above attacks to take over their account. However, as we now know the admin role name (`web_admin`), we can set it to our user so we can create new users or delete current users. To do so, we will intercept the request when we click on the `Update profile` button and change our role to `web_admin`:

<figure><img src="../../../.gitbook/assets/image (1060).png" alt=""><figcaption></figcaption></figure>

This time, we do not get the `Invalid role` error message, nor do we get any access control error messages, meaning that there are no back-end access control measures to what roles we can set for our user. If we `GET` our user details, we see that our `role` has indeed been set to `web_admin`:

Code: json

```json
{
    "uid": "1",
    "uuid": "40f5888b67c748df7efba008e7c2f9d2",
    "role": "web_admin",
    "full_name": "Amy Lindon",
    "email": "a_lindon@employees.htb",
    "about": "A Release is like a boat. 80% of the holes plugged is not good enough."
}
```

Now, we can refresh the page to update our cookie, or manually set it as `Cookie: role=web_admin`, and then intercept the `Update` request to create a new user and see if we'd be allowed to do so:

<figure><img src="../../../.gitbook/assets/image (1061).png" alt=""><figcaption></figcaption></figure>

We did not get an error message this time. If we send a `GET` request for the new user, we see that it has been successfully created:

<figure><img src="../../../.gitbook/assets/image (1062).png" alt=""><figcaption></figcaption></figure>

PoCs - Questions

* Try to change the admin's email to 'flag@idor.htb', and you should get the flag on the 'edit profile' page.

After intercept the profile peticion, i modify the uid profile to 10 -->

<figure><img src="../../../.gitbook/assets/image (1069).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1063).png" alt=""><figcaption></figcaption></figure>

I can see the information of admin in the front too, with it, i can modify the email and intercept the peticion -->

<figure><img src="../../../.gitbook/assets/image (1064).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1065).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1068).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1070).png" alt=""><figcaption></figcaption></figure>
