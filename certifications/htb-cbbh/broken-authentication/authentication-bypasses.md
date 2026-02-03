# Authentication Bypasses

## Authentication Bypass via Direct Access

### Direct Access

This code redirects the user to `/index.php` if the session is not active, i.e., if the user is not authenticated. However, the PHP script does not stop execution, resulting in protected information within the page being sent in the response body:

<figure><img src="../../../.gitbook/assets/image (139).png" alt=""><figcaption></figcaption></figure>

We can easily trick the browser into displaying the admin page by intercepting the response and changing the status code from `302` to `200`. To do this, enable `Intercept` in Burp. Afterward, browse to the `/admin.php` endpoint in the web browser. Next, right-click on the request and select `Do intercept > Response to this request` to intercept the response:

<figure><img src="../../../.gitbook/assets/image (140).png" alt=""><figcaption></figcaption></figure>

Afterward, forward the request by clicking on `Forward`. Since we intercepted the response, we can now edit it. To force the browser to display the content, we need to change the status code from `302 Found` to `200 OK`:

<figure><img src="../../../.gitbook/assets/image (141).png" alt=""><figcaption></figcaption></figure>

To prevent the protected information from being returned in the body of the redirect response, the PHP script needs to exit after issuing the redirect:

```php
if(!$_SESSION['active']) {
	header("Location: index.php");
	exit;
}
```

### PoCs - Questions

* Apply what you learned in this section to bypass authentication to obtain the flag.

By intercepting the peticion request, modify the 302 to 200 OK

***

## Authentication Bypass via Parameter Modification

This type of vulnerability is closely related to authorization issues such as `Insecure Direct Object Reference (IDOR)` vulnerabilities, which are covered in more detail in the [Web Attacks](https://academy.hackthebox.com/module/details/134) module.

### Parameter Modification

Let us take a look at our target web application. This time, we are provided with credentials for the user `htb-stdnt`. After logging in, we are redirected to `/admin.php?user_id=183`:

<figure><img src="../../../.gitbook/assets/image (142).png" alt=""><figcaption></figcaption></figure>

In our web browser, we can see that we seem to be lacking privileges, as we can only see a part of the available data:

<figure><img src="../../../.gitbook/assets/image (143).png" alt=""><figcaption></figcaption></figure>

To investigate the purpose of the `user_id` parameter, let us remove it from our request to `/admin.php`. When doing so, we are redirected back to the login screen at `/index.php`, even though our session provided in the `PHPSESSID` cookie is still valid:

<figure><img src="../../../.gitbook/assets/image (144).png" alt=""><figcaption></figcaption></figure>

Thus, we can assume that the parameter `user_id` is related to authentication. We can bypass authentication entirely by accessing the URL `/admin.php?user_id=183` directly:

<figure><img src="../../../.gitbook/assets/image (145).png" alt=""><figcaption></figcaption></figure>

Based on the parameter name `user_id`, we can infer that the parameter specifies the ID of the user accessing the page. If we can guess or brute-force the user ID of an administrator, we might be able to access the page with administrative privileges, thus revealing the admin information. We can use the techniques discussed in the `Brute-Force Attacks` sections to obtain an administrator ID. Afterward, we can obtain administrative privileges by specifying the admin's user ID in the `user_id` parameter.

### PoCs - Questions

* Apply what you learned in this section to bypass authentication to obtain the flag

To login with us credential and intercept te request, we can see it into dashboard -->

<figure><img src="../../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Fuzzing it number with Intruder. The number of admin is `372`

