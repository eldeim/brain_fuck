# LAB - Insecure Auth 1

In this lab environment, you will have GUI access to a Debian machine. An application named **Frenzy** is available on the Android Emulator.

**Objective:** Complete the following tasks:

* **Task 1:** Decompile the Frenzy APK file, analyze the code, and understand the regular expression responsible for weak password validation during user registration.
* **Task 2:** Based on your observations from Task 1, construct a regex that accepts strings like **Bazinga@12321#**. The regex should check for the following: a minimum length of 14 characters, at least one uppercase and one lowercase letter, at least one digit, at least one special character and no spaces.

**Note:** You can start the emulator using the script located on the Desktop. Additionally, check the **/root/Tools** directory for available tools.

***

Frist we execute the startemulator android and click into Frenzy App. We can see a Login panel an Register panel -->

<figure><img src="../../../.gitbook/assets/image (1272).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1273).png" alt=""><figcaption></figcaption></figure>

We can see that to create a user account, we need create a password must be least 14 characters min, and others characters.

Try to set a password example: aB12#!

<figure><img src="../../../.gitbook/assets/image (1274).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1275).png" alt=""><figcaption></figcaption></figure>

WTF, we can create a ccount, try login -->

<figure><img src="../../../.gitbook/assets/image (1276).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1277).png" alt=""><figcaption></figcaption></figure>

OKAY! Now, go to get the package name, download APK and decompile it. Open with jadx-gui -->

```
## Search package
adb shell pm list packages -f | grep "frenzy"
## Download APK
adb pull /data/app/~~oZ0lNhDdkIp2NaWMhGczgw==/com.example.frenzy-ttByxQb49HI7GiOb62XhPQ==/base.apk .
## Open jadx
jadx-gui base.apk
```

<figure><img src="../../../.gitbook/assets/image (1278).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1279).png" alt=""><figcaption></figcaption></figure>

As seen above, the password validation is handled by the `isValidPassword()` method. This method uses a regular expression (regex) to check whether the entered password meets certain criteria.

The regex used is: `^(?=.*[0-9])(?=.*[!@#$%^&*])(?=\\S+$).{4,}$`

**Let's break down the regular expression:**

* `(?=.*[0-9])`
  * This is a positive lookahead. A positive lookahead is a special part of a regular expression that checks if a certain condition is true ahead in the string, without actually consuming any characters.
  * It means: the string must contain at least one digit (0 through 9).
  * `.*` allows any characters before the digit.
  * This regex basically says - "Look ahead from the current position, and make sure there's at least one digit somewhere ahead in the string."
* `(?=.*[!@#$%^&*])`
  * Another positive lookahead.
  * Ensures that there is at least one special character from the set `!@#$%^&*` somewhere in the string.
* `(?=\\S+$)`
  * Yet another positive lookahead.
  * `\S` matches any non-whitespace character. The extra `\` is used to escape backslash in the code.
  * `\S+$` means the entire string must be made of non-whitespace characters (no spaces, tabs, etc.).
  * Ensures that the entire string has no whitespace.
* `.{4,}` → Minimum length of 4 characters.
  * This means: the string must be at least 4 characters long.
  * The `.` matches any character (except newline), and `{4,}` requires at least 4 characters.
* `$`
  * End of the string. Ensures that the match ends at the end of the password.

**So, this regex ensures that a password:**

* Has at least one digit.
* Has at least one special character (!@#$%^&\*).
* Contains no spaces.
* Is at least 4 characters long.

Although the UI might say that passwords must be at least 14 characters long, the actual code only enforces a minimum of 4 characters, which is a vulnerability.

Now, if we were to construct a strong regex that also checks the password for:

* At least one uppercase letter
* At least one lowercase letter
* Ensuring the minimum password length is 14 characters

The resulting regex would be: `^(?=.*[0-9])(?=.*[!@#$%^&*])(?=.*[a-z])(?=.*[A-Z])(?=\\S+$).{14,}$`

In addition to what we saw earlier:

* `(?=.*[a-z])`: Ensures that there is at least one lowercase letter.
* `(?=.*[A-Z])`: Ensures that there is at least one uppercase letter.
* `.{14,}$`: Ensures that the length of the string is at least 14 characters.
