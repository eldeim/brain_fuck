# SSI

## Introduction to SSI Injection

Server-Side Includes (SSI) is a technology web applications use to create dynamic content on HTML pages.

SSI is supported by many popular web servers such as [Apache](https://httpd.apache.org/docs/current/howto/ssi.html) and [IIS](https://learn.microsoft.com/en-us/iis/configuration/system.webserver/serversideinclude). The use of SSI can often be inferred from the file extension.

Typical file extensions include `.shtml`, `.shtm`, and `.stm`. However, web servers can be configured to support SSI directives in arbitrary file extensions. As such, we cannot conclusively conclude whether SSI is used only from the file extension.

### SSI Directives

SSI utilizes `directives` to add dynamically generated content to a static HTML page. These directives consist of the following components:

* `name`: the directive's name
* `parameter name`: one or more parameters
* `value`: one or more parameter values

An SSI directive has the following syntax:

```ssi
<!--#name param1="value1" param2="value" -->
```

For instance, the following are some common SSI directives.

#### **printenv**

This directive prints environment variables. It does not take any variables.

```ssi
<!--#printenv -->
```

#### **config**

This directive changes the SSI configuration by specifying corresponding parameters. For instance, it can be used to change the error message using the `errmsg` parameter:

```ssi
<!--#config errmsg="Error!" -->
```

#### **echo**

This directive prints the value of any variable given in the `var` parameter. Multiple variables can be printed by specifying multiple `var` parameters. For instance, the following variables are supported:

* `DOCUMENT_NAME`: the current file's name
* `DOCUMENT_URI`: the current file's URI
* `LAST_MODIFIED`: timestamp of the last modification of the current file
* `DATE_LOCAL`: local server time

```ssi
<!--#echo var="DOCUMENT_NAME" var="DATE_LOCAL" -->
```

#### **exec**

This directive executes the command given in the `cmd` parameter:

```ssi
<!--#exec cmd="whoami" -->
```

#### **include**

This directive includes the file specified in the `virtual` parameter. It only allows for the inclusion of files in the web root directory.

```ssi
<!--#include virtual="index.html" -->
```

## Exploiting SSI Injection

### Exploitation

If we enter our name, we are redirected to `/page.shtml`, which displays some general information:

<figure><img src="../../../../.gitbook/assets/image (993).png" alt=""><figcaption></figcaption></figure>

We can guess that the page supports SSI based on the file extension. If our username is inserted into the page without prior sanitization, it might be vulnerable to SSI injection. Let us confirm this by providing a username of `<!--#printenv -->`. This results in the following page:

<figure><img src="../../../../.gitbook/assets/image (994).png" alt=""><figcaption></figcaption></figure>

As we can see, the directive is executed, and the environment variables are printed. Thus, we have successfully confirmed an SSI injection vulnerability. Let us confirm that we can execute arbitrary commands using the `exec` directive by providing the following username: `<!--#exec cmd="id" -->`:

<figure><img src="../../../../.gitbook/assets/image (995).png" alt=""><figcaption></figcaption></figure>

PoCs - Questions

* Exploit the SSI Injection vulnerability to obtain RCE and read the flag.

```
<!--#exec cmd="cat /flag.txt" -->
```
