# Food Store

#### Objective <a href="#el_1700338310850_408" id="el_1700338310850_408"></a>

* Exploit a SQL Injection Vulnerability: Your mission is to manipulate the signup function in the "Food Store" Android application, allowing you to register as a Pro user, bypassing standard user restrictions.

#### Approach <a href="#el_1700338310834_390" id="el_1700338310834_390"></a>

* **Analyze the Signup Function**: Scrutinize the app's signup process for SQLi vulnerabilities.
* **Craft Malicious SQL Queries**: Develop SQL queries to manipulate the signup process and gain Pro user access.
* **Test and Validate**: Execute your SQLi strategies within the provided lab environment.

#### Hints <a href="#el_1700338310831_386" id="el_1700338310831_386"></a>

* **Focus on Input Validation**: Pay attention to how user inputs are processed and validated in the signup function.
* **Code Review**: Examine the code using the reverse engineering tools to find the injection point.

***

## Dinamic Ejecutation

Fristly, once we start the app, we can see a login and sign up function. I create a new account like eldeim -->

<figure><img src="../../../../../.gitbook/assets/image (1516).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (1517).png" alt=""><figcaption></figcaption></figure>

Just in case, i will create another user wit ' or 1=1--

<figure><img src="../../../../../.gitbook/assets/image (1518).png" alt=""><figcaption></figcaption></figure>

Once, we have all, will be login into the up with its user -->

<figure><img src="../../../../../.gitbook/assets/image (1519).png" alt=""><figcaption></figcaption></figure>
