# Gobuster

### Web Enumeration

```bash
gobuster dir -u http://example.com -w wordlist.txt -x php,txt,html -t 80
```

> `dir` Indica que se realizará un ataque de enumeración de directorios.
>
> `-u` http://example.com: Especifica la URL objetivo que se va a enumerar
>
> `-w`  wordlist.txt: Indica que se utilizará el archivo wordlist.txt
>
> `-x`  php,txt: Especifica las extensiones de archivo que se probarán durante el ataque.&#x20;
>
> > En este caso, Gobuster intentará enumerar directorios con extensiones .php y .txt.
>
> `-t`  Hilos a utilizar

### Sub-Domain Enumeration

```bash
gobuster vhost -u http://example.com -w wordlist.txt --append-domain -t 100
```

> `vhost`  Utiliza VHOST para la fuerza bruta&#x20;
>
> `-w`  Ruta a la lista de palabras&#x20;
>
> `-u`  Especifica la URL&#x20;
>
> `--append-domain`  Para añadir el dominio base
>
> `-t`  Hilos a utilizar

### Proxy

```
gobuster dir -u http://example.com -w wordlist.txt -x php,txt,html --proxy socks5://127.0.0.1:1080
```

> `--proxy socks5` Especifica el proxy a utilizar
