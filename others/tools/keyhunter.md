---
description: Password Scanner for Wordlists
---

# KeyHunter

{% embed url="https://github.com/gitblanc/KeyHunter" %}

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Install

Clone the repository:

```
git clone https://github.com/your-username/keyhunter.git
cd keyhunter
```

Install the dependencies:

```
pip install -r requirements.txt
```

Run the script:

```
python3 keyhunter.py [options]
```

## Use

* Search for passwords across multiple and different files (_**FASTER**_):

```
python3 keyhunter.py /path/to/wordlist_parts/ password [-v] [--output /path/to/scan.txt]
```

* Search for passwords in a file (_**SLOWER**_):

```
python3 keyhunter.py /path/to/wordlist.txt password [-v] [--output /path/to/scan.txt]
```

* Split a file in multiple smaller ones (to split massive wordlists):

```
python3 keyhunter.py /path/to/wordlist.txt --split [--max_size 50]
```

* Export results to pdf:

```
python3 keyhunter.py /path/to/wordlist.txt search_term --pdf
```
