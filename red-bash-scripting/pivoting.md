# Pivoting

### Host Discovery

```bash
#!/bin/bash
function ctrl_c(){
        echo -e "\n\nSaliendo !! \n"
        tput cnorm; exit 1
}

#CTRL_C
trap ctrl_c INT

tput civis

for i in $(seq 1 254);do

        timeout 2 bash -c "ping -c 1 10.10.0.$i" &> /dev/null && echo "!Host activo! 10.10.0.$i" &

done; wait
tput cnorm
```

### Port Discovery & Host Discovery

```bash
#!/bin/bash
function ctrl_c(){
        echo -e "\n\nSaliendo !! \n"
        tput cnorm; exit 1
}

#CTRL_C
trap ctrl_c INT

tput civis
for i in $(seq 1 254); do
        for port in 21 22 25 443 8080 80 5985 3060; do
                timeout 2 bash -c "echo '' > /dev/tcp/10.10.0.$i/$port" &> /dev/null && echo "host 10.10.0.$i - port $port" &
        done
done; wait
tput cnorm
```

### Port Exactly Host

```bash
#!/bin/bash

function ctrl_c(){
        echo -e "\n\nSaliendo !! \n"
        tput cnorm; exit 1
}

#CTRL_C
trap ctrl_c INT


tput civis

for port in $(seq 1 65535); do

        timeout 1 bash -c "echo '' > /dev/tcp/10.10.0.132/$port" &> /dev/null && echo "port 10.10.0.132:$port - OPEN" &

done; wait

tput cnorm
```
