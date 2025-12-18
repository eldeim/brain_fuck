# API Fuzzing

Android Emulator.

The **HeyDoc** app's backend exposes certain endpoints that were intended to be called only by internal systems, but there are no access controls or IP restrictions in place. The developers assumed that no one would discover these obscure endpoints without documentation.

**Objective:** Your task is to perform parameter type fuzzing to uncover the HeyDoc app's hidden API functions.

The valid credentials for **HeyDoc** app are as follows:

* **Username:** alice
* **Password:** Bazinga@12345#

The following wordlist will be useful:

* /home/student/Desktop/Wordlists/parameter-names.txt

***

First LogIn nto the APP with the credentials given -->

<figure><img src="../../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now, configure proxy cmd and Burp proxy-->

```
## View us IP 
hostname -I
## Set local proxy
adb shell settings put global http_proxy <host-ip>:8080
```

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

In the basic functionatility of APK, we can get a appointment with the doctor selecting the hours -->

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So, i can see into burpsuite, all peticions navegate into `/api/v1/appointments`,what happend if i do fuff into this directory -->

<figure><img src="../../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can notice that two requests returned a **405** Method Not Allowed status code. They are:

* **/api/v1/appointments/override**
* **/api/v1/appointments/free\_all**

We have found valid parameters: `override` and `free_all`. The 405 error indicates that the expected HTTP method is not GET, but something else.

<figure><img src="../../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We now get a different error, which means the backend is expecting some JSON data with this request. Since we don’t know what data it requires, let’s pass some dummy JSON data:

```
{
    "test": "test"
}
```

Additionally, ensure that the following header is set to specify the content type:

```
Content-Type: application/json
```

<figure><img src="../../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now, send the request again.

<figure><img src="../../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We are making progress. The response reveals that the expected JSON data should contain an appointment ID. From previous exploration, we know that appointments are represented by `id` values ranging from 1 to 4. Let's modify the JSON with this information and observe the response.

<figure><img src="../../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Success! It appears this hidden endpoint is used to override booked appointments. We have managed to book an appointment for user `alice` at a time slot that was previously unavailable because it was booked by another user. To confirm this, let’s hit the `/api/v1/appointments/mine` endpoint again.

<figure><img src="../../../../.gitbook/assets/image (12) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (13) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now, let's explore the next hidden endpoint: `/api/v1/appointments/free_all`. Send this request to **Repeater**.

Change the request method to **POST** and observe the response.

<figure><img src="../../../../.gitbook/assets/image (14) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The response indicates that this endpoint is used to free up all appointments.

To confirm this, hit the `/api/v1/appointments` endpoint.

<figure><img src="../../../../.gitbook/assets/image (15) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The `is_booked` value is now `false` for all appointment time slots. The same can be observed in the app. Make sure to first exit the app and then reopen it.

<figure><img src="../../../../.gitbook/assets/image (16) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
