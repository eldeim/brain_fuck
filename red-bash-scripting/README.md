# 🍎 Red - Bash Scripting

### Procmon.sh

```bash
#!/bin/bash
function ctrl_c(){
        echo -e "\n\nSaliendo !! \n"
        tput cnorm; exit 1
}

#CTRL_C
trap ctrl_c INT

tput civis

old_process=$(ps -eo command)

while true; do

    new_process=$(ps -eo command)
    diff <(echo "$old_process") <(echo "$new_process") | grep "[\>\<]" | grep -vE "procmon|command|kworker"
    old_process=$new_process

done
tput cnorm
```
