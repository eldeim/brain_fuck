# DNS & Subdomains

## WHOIS

```shell-session
eldeim@htb[/htb]$ whois inlanefreight.com

[...]
Domain Name: inlanefreight.com
Registry Domain ID: 2420436757_DOMAIN_COM-VRSN
Registrar WHOIS Server: whois.registrar.amazon
Registrar URL: https://registrar.amazon.com
Updated Date: 2023-07-03T01:11:15Z
Creation Date: 2019-08-05T22:43:09Z
[...]
```

Each WHOIS record typically contains the following information:

* `Domain Name`: The domain name itself (e.g., example.com)
* `Registrar`: The company where the domain was registered (e.g., GoDaddy, Namecheap)
* `Registrant Contact`: The person or organization that registered the domain.
* `Administrative Contact`: The person responsible for managing the domain.
* `Technical Contact`: The person handling technical issues related to the domain.
* `Creation and Expiration Dates`: When the domain was registered and when it's set to expire.
* `Name Servers`: Servers that translate the domain name into an IP address.

## DNS

<table><thead><tr><th width="78">Record Type</th><th>Full Name</th><th>Description</th><th>Zone File Example</th></tr></thead><tbody><tr><td><code>A</code></td><td>Address Record</td><td>Maps a hostname to its IPv4 address.</td><td><code>www.example.com.</code> IN A <code>192.0.2.1</code></td></tr><tr><td><code>AAAA</code></td><td>IPv6 Address Record</td><td>Maps a hostname to its IPv6 address.</td><td><code>www.example.com.</code> IN AAAA <code>2001:db8:85a3::8a2e:370:7334</code></td></tr><tr><td><code>CNAME</code></td><td>Canonical Name Record</td><td>Creates an alias for a hostname, pointing it to another hostname.</td><td><code>blog.example.com.</code> IN CNAME <code>webserver.example.net.</code></td></tr><tr><td><code>MX</code></td><td>Mail Exchange Record</td><td>Specifies the mail server(s) responsible for handling email for the domain.</td><td><code>example.com.</code> IN MX 10 <code>mail.example.com.</code></td></tr><tr><td><code>NS</code></td><td>Name Server Record</td><td>Delegates a DNS zone to a specific authoritative name server.</td><td><code>example.com.</code> IN NS <code>ns1.example.com.</code></td></tr><tr><td><code>TXT</code></td><td>Text Record</td><td>Stores arbitrary text information, often used for domain verification or security policies.</td><td><code>example.com.</code> IN TXT <code>"v=spf1 mx -all"</code> (SPF record)</td></tr><tr><td><code>SOA</code></td><td>Start of Authority Record</td><td>Specifies administrative information about a DNS zone, including the primary name server, responsible person's email, and other parameters.</td><td><code>example.com.</code> IN SOA <code>ns1.example.com. admin.example.com. 2024060301 10800 3600 604800 86400</code></td></tr><tr><td><code>SRV</code></td><td>Service Record</td><td>Defines the hostname and port number for specific services.</td><td><code>_sip._udp.example.com.</code> IN SRV 10 5 5060 <code>sipserver.example.com.</code></td></tr><tr><td><code>PTR</code></td><td>Pointer Record</td><td>Used for reverse DNS lookups, mapping an IP address to a hostname.</td><td><code>1.2.0.192.in-addr.arpa.</code> IN PTR <code>www.example.com.</code></td></tr></tbody></table>

### Digging DNS

<table><thead><tr><th width="218">Command</th><th>Description</th></tr></thead><tbody><tr><td><code>dig domain.com</code></td><td>Performs a default A record lookup for the domain.</td></tr><tr><td><code>dig domain.com A</code></td><td>Retrieves the IPv4 address (A record) associated with the domain.</td></tr><tr><td><code>dig domain.com AAAA</code></td><td>Retrieves the IPv6 address (AAAA record) associated with the domain.</td></tr><tr><td><code>dig domain.com MX</code></td><td>Finds the mail servers (MX records) responsible for the domain.</td></tr><tr><td><code>dig domain.com NS</code></td><td>Identifies the authoritative name servers for the domain.</td></tr><tr><td><code>dig domain.com TXT</code></td><td>Retrieves any TXT records associated with the domain.</td></tr><tr><td><code>dig domain.com CNAME</code></td><td>Retrieves the canonical name (CNAME) record for the domain.</td></tr><tr><td><code>dig domain.com SOA</code></td><td>Retrieves the start of authority (SOA) record for the domain.</td></tr><tr><td><code>dig @1.1.1.1 domain.com</code></td><td>Specifies a specific name server to query; in this case 1.1.1.1</td></tr><tr><td><code>dig +trace domain.com</code></td><td>Shows the full path of DNS resolution.</td></tr><tr><td><code>dig -x 192.168.1.1</code></td><td>Performs a reverse lookup on the IP address 192.168.1.1 to find the associated host name. You may need to specify a name server.</td></tr><tr><td><code>dig +short domain.com</code></td><td>Provides a short, concise answer to the query.</td></tr><tr><td><code>dig +noall +answer domain.com</code></td><td>Displays only the answer sec</td></tr></tbody></table>

## Subdomain Bruteforcing

### DNSEnum

Let's see `dnsenum` in action by demonstrating how to enumerate subdomains for our target, `inlanefreight.com`. In this demonstration, we'll use the `subdomains-top1million-5000.txt` wordlist from [SecLists](https://github.com/danielmiessler/SecLists), which contains the top 5000 most common subdomains.

```bash
dnsenum --enum inlanefreight.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -r
```

> `-r`: This option enables recursive subdomain brute-forcing, meaning that if `dnsenum` finds a subdomain, it will then try to enumerate subdomains of that subdomain

```shell-session
eldeim@htb[/htb]$ dnsenum --enum inlanefreight.com -f  /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt 

dnsenum VERSION:1.2.6

-----   inlanefreight.com   -----

Host's addresses:
__________________

inlanefreight.com.                       300      IN    A        134.209.24.248

[...]

Brute forcing with /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt:
_______________________________________________________________________________________

www.inlanefreight.com.                   300      IN    A        134.209.24.248
support.inlanefreight.com.               300      IN    A        134.209.24.248
[...]

done.
```

## **Exploiting Zone Transfers**

You can use the `dig` command to request a zone transfer:

```shell-session
eldeim@htb[/htb]$ dig axfr @nsztm1.digi.ninja zonetransfer.me
```

This command instructs `dig` to request a full zone transfer (`axfr`) from the DNS server responsible for `zonetransfer.me`. If the server is misconfigured and allows the transfer, you'll receive a complete list of DNS records for the domain, including all subdomains.

```shell-session
eldeim@htb[/htb]$ dig axfr @nsztm1.digi.ninja zonetransfer.me

; <<>> DiG 9.18.12-1~bpo11+1-Debian <<>> axfr @nsztm1.digi.ninja zonetransfer.me
; (1 server found)
;; global options: +cmd
zonetransfer.me.	7200	IN	SOA	nsztm1.digi.ninja. robin.digi.ninja. 2019100801 172800 900 1209600 3600
zonetransfer.me.	300	IN	HINFO	"Casio fx-700G" "Windows XP"
zonetransfer.me.	301	IN	TXT	"google-site-verification=tyP28J7JAUHA9fw2sHXMgcCC0I6XBmmoVi04VlMewxA"
zonetransfer.me.	7200	IN	MX	0 ASPMX.L.GOOGLE.COM.
...
zonetransfer.me.	7200	IN	A	5.196.105.14
zonetransfer.me.	7200	IN	NS	nsztm1.digi.ninja.
zonetransfer.me.	7200	IN	NS	nsztm2.digi.ninja.
_acme-challenge.zonetransfer.me. 301 IN	TXT	"6Oa05hbUJ9xSsvYy7pApQvwCUSSGgxvrbdizjePEsZI"
_sip._tcp.zonetransfer.me. 14000 IN	SRV	0 0 5060 www.zonetransfer.me.
14.105.196.5.IN-ADDR.ARPA.zonetransfer.me. 7200	IN PTR www.zonetransfer.me.
asfdbauthdns.zonetransfer.me. 7900 IN	AFSDB	1 asfdbbox.zonetransfer.me.
asfdbbox.zonetransfer.me. 7200	IN	A	127.0.0.1
asfdbvolume.zonetransfer.me. 7800 IN	AFSDB	1 asfdbbox.zonetransfer.me.
canberra-office.zonetransfer.me. 7200 IN A	202.14.81.230
...
;; Query time: 10 msec
;; SERVER: 81.4.108.41#53(nsztm1.digi.ninja) (TCP)
;; WHEN: Mon May 27 18:31:35 BST 2024
;; XFR size: 50 records (messages 1, bytes 2085)
```

`zonetransfer.me` is a service specifically setup to demonstrate the risks of zone transfers so that the `dig` command will return the full zone record.

## Virtual Hosting

```bash
sudo nano /etc/hosts
## Add
94.237.52.18 inlanefreight.htb
## Run
gobuster vhost -u http://inlanefreight.htb:39472 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
```

> The `--append-domain` flag appends the base domain to each word in the wordlist.

### Crt.sh lookup

While `crt.sh` offers a convenient web interface, you can also leverage its API for automated searches directly from your terminal. Let's see how to find all 'dev' subdomains on `facebook.com` using `curl` and `jq`:

```shell-session
eldeim@htb[/htb]$ curl -s "https://crt.sh/?q=facebook.com&output=json" | jq -r '.[]
 | select(.name_value | contains("dev")) | .name_value' | sort -u
 
*.dev.facebook.com
*.newdev.facebook.com
*.secure.dev.facebook.com
dev.facebook.com
devvm1958.ftw3.facebook.com
facebook-amex-dev.facebook.com
facebook-amex-sign-enc-dev.facebook.com
newdev.facebook.com
secure.dev.facebook.com
```

## ReconSpider

<pre class="language-bash"><code class="lang-bash"><strong>## Download
</strong>eldeim@htb[/htb]$ pip3 install scrapy
<strong>eldeim@htb[/htb]$ wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
</strong>eldeim@htb[/htb]$ unzip ReconSpider.zip 
## Use
eldeim@htb[/htb]$ python3 ReconSpider.py http://inlanefreight.com
</code></pre>

## Google Dorking

Here are some common examples of Google Dorks, for more examples, refer to the [Google Hacking Database](https://www.exploit-db.com/google-hacking-database):

* Finding Login Pages:
  * `site:example.com inurl:login`
  * `site:example.com (inurl:login OR inurl:admin)`
* Identifying Exposed Files:
  * `site:example.com filetype:pdf`
  * `site:example.com (filetype:xls OR filetype:docx)`
* Uncovering Configuration Files:
  * `site:example.com inurl:config.php`
  * `site:example.com (ext:conf OR ext:cnf)` (searches for extensions commonly used for configuration files)
* Locating Database Backups:
  * `site:example.com inurl:backup`
  * `site:example.com filetype:sql`
