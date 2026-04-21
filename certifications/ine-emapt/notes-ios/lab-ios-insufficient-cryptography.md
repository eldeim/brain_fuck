# LAB - iOS: Insufficient Cryptography

In this lab environment, you will get access to a Debian machine, which has all the required tools installed on it for this lab, along with an IPA file.

**Objective:** Complete the following task and retrieve the flag.

* **Task 1:** You are provided with the first four characters of a Base64-encoded string: **"bm92"**. Your task is to find the complete original string from which this encoded fragment was derived.
* **Task 2:** Leverage your discoveries from Task 1 and follow a chain of clues to uncover and retrieve the secret flag.

**The following file can be useful:**

* **MySchool.ipa**: Present on the "Desktop/IPA-Files".

***

<figure><img src="../../../.gitbook/assets/image (440).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (441).png" alt=""><figcaption></figcaption></figure>

```
unzip MySchool.zip
```

<figure><img src="../../../.gitbook/assets/image (442).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (443).png" alt=""><figcaption></figcaption></figure>

```
strings MySchool > output
ls
cat output | grep "bm92"
```

> We found the entire base64-encoded string: **bm92YXRlY2hfdXNlcjpzdXBlcnNlY3JldHBhc3N3b3JkQG5vdmF0ZWNo**

```
echo "bm92YXRlY2hfdXNlcjpzdXBlcnNlY3JldHBhc3N3b3JkQG5vdmF0ZWNo" | base64 -d
```

<figure><img src="../../../.gitbook/assets/image (444).png" alt=""><figcaption></figcaption></figure>

After decoding the Base64-encoded string, we discover what appears to be a set of credentials: `novatech_user:supersecretpassword@novatech` suggesting access to a service, API or system associated with `novatech`.

Let's try finding the URLs and retrieving the flag. NOW GREP by HTTP

<figure><img src="../../../.gitbook/assets/image (445).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (446).png" alt=""><figcaption></figcaption></figure>
