# Web Attacks

{% file src="../../.gitbook/assets/Web_Attacks_Module_Cheat_Sheet.pdf" %}

## **HTTP Verb Tampering**

The first web attack discussed in this module is [HTTP Verb Tampering](https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/07-Input_Validation_Testing/03-Testing_for_HTTP_Verb_Tampering). An HTTP Verb Tampering attack exploits web servers that accept many HTTP verbs and methods. This can be exploited by sending malicious requests using unexpected methods, which may lead to bypassing the web application's authorization mechanism or even bypassing its security controls against other web attacks. HTTP Verb Tampering attacks are one of many other HTTP attacks that can be used to exploit web server configurations by sending malicious HTTP requests.

## **Insecure Direct Object References (IDOR)**

The second attack discussed in this module is [Insecure Direct Object References (IDOR)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References). IDOR is among the most common web vulnerabilities and can lead to accessing data that should not be accessible by attackers. What makes this attack very common is essentially the lack of a solid access control system on the back-end. As web applications store users' files and information, they may use sequential numbers or user IDs to identify each item. Suppose the web application lacks a robust access control mechanism and exposes direct references to files and resources. In that case, we may access other users' files and information by simply guessing or calculating their file IDs.

## **XML External Entity (XXE) Injection**

The third and final web attack we will discuss is [XML External Entity (XXE) Injection](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_\(XXE\)_Processing). Many web applications process XML data as part of their functionality. Suppose a web application utilizes outdated XML libraries to parse and process XML input data from the front-end user. In that case, it may be possible to send malicious XML data to disclose local files stored on the back-end server. These files may be configuration files that may contain sensitive information like passwords or even the source code of the web application, which would enable us to perform a Whitebox Penetration Test on the web application to identify more vulnerabilities. XXE attacks can even be leveraged to steal the hosting server's credentials, which would compromise the entire server and allow for remote code execution.
