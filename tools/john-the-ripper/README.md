# John the Ripper

## Cracking

```bash
john --wordlist /usr/share/rockyou.txt --keep-guessing "hash.txt"
```

> `--keep-guessing` garantiza que John no se detenga al encontrar una coincidencia y continúe con el cracking de los hashes restantes.
