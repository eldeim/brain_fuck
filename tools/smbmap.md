# Smbmap

### Conecction

<pre class="language-bash"><code class="lang-bash">smbmap -H IP_TARGET
------------------------------------------------------------------------
[+] Guest session   	IP: IP_TARGET:445	Name: VICTIM                                    
<strong>    Disk                      Permissions	Comment
</strong><strong>    ---             	       -------------	-------
</strong>    print$                	NO ACCESS	Printer Drivers
    helios                	NO ACCESS	Helios personal share
    anonymous        	        READ ONLY	
    IPC$              	        NO ACCESS	IPC Service (Samba VERSION)
</code></pre>

```bash
smbmap -H IP_TARGET -r anonymous
------------------------------------------------------------------------
[+] Guest session   	IP: IP_TARGET:445	Name: VICTIM                                    
    Disk                      Permissions	Comment
    ---             	    -------------	-------
anonymous                     READ ONLY	
.\anonymous\*
dr--r--r--                0 Sat Jun 29 03:14:49 2019	.
dr--r--r--                0 Sat Jun 29 03:12:15 2019	..
fr--r--r--              154 Sat Jun 29 03:14:49 2019	attention.txt
```

```bash
smbmap -H 192.168.18.214 -u USER -p PASSWD
```

### Download

```bash
smbmap -H 192.168.18.213 --download anonymous/attention.txt
```
