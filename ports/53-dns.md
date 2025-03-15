# 53 - DNS

```
dig @10.10.11.166 trick.htb ns
```

> `--nc` enumerar "name servers"

```
dig @10.10.11.166 trick.htb mx
```

> `--mx` enumerar "servidores de correo"

```
dig @10.10.11.166 trick.htb axfr
```

> `--axfr` ataque de transferencia de zona
