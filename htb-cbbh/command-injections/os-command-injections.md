# OS Command Injections

## **PHP Example**

For example, a web application written in PHP may use the exec, system, shell\_exec, passthru, or popen functions to execute commands directly on the back-end server, each having a slightly different use case.

```php
<?php
if (isset($_GET['filename'])) {
    system("touch /tmp/" . $_GET['filename'] . ".pdf"); }
?>
```

> Perhaps a particular web application has a functionality that allows users to create a new `.pdf` document that gets created in the `/tmp` directory

## **NodeJS Example**

This is not unique to `PHP` only, but can occur in any web development framework or language. For example, if a web application is developed in `NodeJS`, a developer may use `child_process.exec` or `child_process.spawn` for the same purpose.

```javascript
app.get("/createfile", function(req, res){
    child_process.exec(`touch /tmp/${req.query.filename}.txt`);
})
```

The above code is also vulnerable to a command injection vulnerability, as it uses the `filename` parameter from the `GET` request as part of the command without sanitizing it first. Both `PHP` and `NodeJS` web applications can be exploited using the same command injection methods.
