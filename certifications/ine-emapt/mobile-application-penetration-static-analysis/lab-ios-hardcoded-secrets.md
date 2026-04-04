# LAB - iOS: Hardcoded Secrets

## Lab Environment

In this lab environment, you will get access to a Debian machine, which has all the required tools installed on it for this lab, along with an IPA file.

**Objective:** Perform static analysis on the IPA file to find the hardcoded API key.

The following file can be useful:

* reqnest.ipa: Present on the "Desktop".

### Tools

* unzip
* strings
* grep

## Flag

* What is a hardcoded API key value?

We can see into the Desktop folder a file with name "`reqnest.ipa`", unzip it -->

```
cd Desktop
ls
unzip reqnest.ipa
```

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Into this zip/ipa we can see a reqnest filne, so use `strings` to read leggible words -->

```
strings reqnest | grep api
```

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

