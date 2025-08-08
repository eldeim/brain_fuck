# Cron Jobs

`ps -aux | grep cron`

or

`cat /etc/crontab`

<figure><img src="../../.gitbook/assets/Pasted image 20250120175054.png" alt=""><figcaption></figcaption></figure>

```
* * * * * comando_a_ejecutar
- - - - -
| | | | |
| | | | +--- Día de la semana (0 - 7, donde 0 y 7 son domingo)
| | | +----- Mes (1 - 12)
| | +------- Día del mes (1 - 31)
| +--------- Hora (0 - 23)
+----------- Minuto (0 - 59)
```

Ejemplo de explotación:&#x20;

El script backup.sh fue configurado para ejecutarse cada minuto (en la captura). Vemos su contenido:

<figure><img src="../../.gitbook/assets/Pasted image 20250120175353.png" alt=""><figcaption></figcaption></figure>

Podemos modificar el contenido y dar **permiso +x**

<figure><img src="../../.gitbook/assets/Pasted image 20250120175414.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
