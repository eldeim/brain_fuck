# Web Fuzzer

## Burp Intruder

We can then go to `Intruder` by clicking on its tab or with the shortcut \[`CTRL+SHIFT+I`], which takes us right to `Burp Intruder`:

![intruder\_target](https://academy.hackthebox.com/storage/modules/110/burp_intruder_target.jpg)

On the first tab, '`Target`', we see the details of the target we will be fuzzing, which is fed from the request we sent to `Intruder`.

### Positions

The second tab, '`Positions`', is where we place the payload position pointer, which is the point where words from our wordlist will be placed and iterated over. We will be demonstrating how to fuzz web directories, which is similar to what's done by tools like `ffuf` or `gobuster`.

To check whether a web directory exists, our fuzzing should be in '`GET /DIRECTORY/`', such that existing pages would return `200 OK`, otherwise we'd get `404 NOT FOUND`. So, we will need to select `DIRECTORY` as the payload position, by either wrapping it with `§` or by selecting the word `DIRECTORY` and clicking on the `Add §` button:

![intruder\_position](https://academy.hackthebox.com/storage/modules/110/burp_intruder_position.jpg)

> Tip: the `DIRECTORY` in this case is the pointer's name, which can be anything, and can be used to refer to each pointer, in case we are using more than position with different wordlists for each.

### **Payload Sets**

The first thing we must configure is the `Payload Set`. The payload set identifies the Payload number, depending on the attack type and number of Payloads we used in the Payload Position Pointers:

![Payload Sets](https://academy.hackthebox.com/storage/modules/110/burp_intruder_payload_set.jpg)

In this case, we only have one Payload Set, as we chose the '`Sniper`' Attack type with only one payload position. If we have chosen the '`Cluster Bomb`' attack type, for example, and added several payload positions, we would get more payload sets to choose from and choose different options for each. In our case, we'll select `1` for the payload set.

Next, we need to select the `Payload Type`, which is the type of payloads/wordlists we will be using. Burp provides a variety of Payload Types, each of which acts in a certain way. For example:

* `Simple List`: The basic and most fundamental type. We provide a wordlist, and Intruder iterates over each line in it.
* `Runtime file`: Similar to `Simple List`, but loads line-by-line as the scan runs to avoid excessive memory usage by Burp.
* `Character Substitution`: Lets us specify a list of characters and their replacements, and Burp Intruder tries all potential permutations.

There are many other Payload Types, each with its own options, and many of which can build custom wordlists for each attack. Try clicking on the `?` next to `Payload Sets`, and then click on `Payload Type`, to learn more about each Payload Type. In our case, we'll be going with a basic `Simple List`.

### Payload Options

We will select `/opt/useful/seclists/Discovery/Web-Content/common.txt` as our wordlist. We can see that Burp Intruder loads all lines of our wordlist into the Payload Options table:

![Payload Options](https://academy.hackthebox.com/storage/modules/110/burp_intruder_payload_wordlist.jpg)

We can add another wordlist or manually add a few items, and they would be appended to the same list of items. We can use this to combine multiple wordlists or create customized wordlists. In Burp Pro, we also can select from a list of existing wordlists contained within Burp by choosing from the `Add from list` menu option.

> Tip: In case you wanted to use a very large wordlist, it's best to use `Runtime file` as the Payload Type instead of `Simple List`, so that Burp Intruder won't have to load the entire wordlist in advance, which may throttle memory usage.

### **Payload Processing**

Let's try adding a rule that skips any lines that start with a `.` (as shown in the wordlist screenshot earlier). We can do that by clicking on the `Add` button and then selecting `Skip if matches regex`, which allows us to provide a regex pattern for items we want to skip. Then, we can provide a regex pattern that matches lines starting with `.`, which is: `^\..*$`:

![payload processing](https://academy.hackthebox.com/storage/modules/110/burp_intruder_payload_processing_1.jpg)

We can see that our rule gets added and enabled:&#x20;

<figure><img src="https://academy.hackthebox.com/storage/modules/110/burp_intruder_payload_processing_2.jpg" alt=""><figcaption></figcaption></figure>

### **Payload Encoding**

The fourth and final option we can apply is `Payload Encoding`, enabling us to enable or disable Payload URL-encoding.

![payload encoding](https://academy.hackthebox.com/storage/modules/110/burp_intruder_payload_encoding.jpg)

We'll leave it enabled.

### Options

Another useful option is the `Grep - Match`, which enables us to flag specific requests depending on their responses. As we are fuzzing web directories, we are only interested in responses with HTTP code `200 OK`. So, we'll first enable it and then click `Clear` to clear the current list. After that, we can type `200 OK` to match any requests with this string and click `Add` to add the new rule. Finally, we'll also disable `Exclude HTTP Headers`, as what we are looking for is in the HTTP header:

![options match](https://academy.hackthebox.com/storage/modules/110/burp_intruder_options_match.jpg)

We may also utilize the `Grep - Extract` option, which is useful if the HTTP responses are lengthy, and we're only interested in a certain part of the response. So, this helps us in only showing a specific part of the response. We are only looking for responses with HTTP Code `200 OK`, regardless of their content, so we will not opt for this option.

***

## ZAP Fuzzer

### Fuzz

To start our fuzzing, we will visit the URL from the exercise at the end of this section to capture a sample request. As we will be fuzzing for directories, let's visit `<http://SERVER_IP:PORT/test/>` to place our fuzzing location on `test` later on. Once we locate our request in the proxy history, we will right-click on it and select (`Attack>Fuzz`), which will open the `Fuzzer` window:

![payload processing](https://academy.hackthebox.com/storage/modules/110/zap_fuzzer.jpg)

### Locations

The `Fuzz Location` is very similar to `Intruder Payload Position`, where our payloads will be placed. To place our location on a certain word, we can select it and click on the `Add` button on the right pane. So, let's select `test` and click on `Add`:

![payload processing](https://academy.hackthebox.com/storage/modules/110/zap_fuzzer_add.jpg)

As we can see, this placed a `green` marker on our selected location and opened the `Payloads` window for us to configure our attack payloads.

### Payloads

The attack payloads in ZAP's Fuzzer are similar in concept to Intruder's Payloads, though they are not as advanced as Intruder's. We can click on the `Add` button to add our payloads and select from 8 different payload types. The following are some of them:

* `File`: This allows us to select a payload wordlist from a file.
* `File Fuzzers`: This allows us to select wordlists from built-in databases of wordlists.
* `Numberzz`: Generates sequences of numbers with custom increments

&#x20;So, we can select `File Fuzzers` as the `Type`, and then we will select the first wordlist from `dirbuster`:

![payload processing](https://academy.hackthebox.com/storage/modules/110/zap_fuzzer_add_payload.jpg)

Once we click the `Add` button, our payload wordlist will get added, and we can examine it with the `Modify` button.

### Processors

We may also want to perform some processing on each word in our payload wordlist. The following are some of the payload processors we can use:

* Base64 Decode/Encode
* MD5 Hash
* Postfix String
* Prefix String
* SHA-1/256/512 Hash
* URL Decode/Encode
* Script

We can click on the `Generate Preview` button to preview how our final payload will look in the request:

![payload processing](https://academy.hackthebox.com/storage/modules/110/zap_fuzzer_add_processor.jpg)

Once that's done, we can click on `Add` to add the processor and click on `Ok` in the processors and payloads windows to close them.

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Start

With all of our options configured, we can finally click on the `Start Fuzzer` button to start our attack. Once our attack is started, we can sort the results by the `Response` code, as we are only interested in responses with code `200`:

![payload processing](https://academy.hackthebox.com/storage/modules/110/zap_fuzzer_attack.jpg)

As we can see, we got one hit with code `200` with the `skills` payload, meaning that the `/skills/` directory exists on the server and is accessible. We can click on the request in the results window to view its details:&#x20;

<figure><img src="https://academy.hackthebox.com/storage/modules/110/zap_fuzzer_dir.jpg" alt=""><figcaption></figcaption></figure>
