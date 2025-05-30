# BloondHound & Ingestor

## How to Install

```bash
##Install Docker
sudo apt install docker.io && sudo apt install docker-compose

##Add u user to the Docker group
sudo usermod -aG docker $USER

##Install Bloodhound
#### Download the bloodhound Docker compose file
curl -L https://ghst.ly/getbhce > docker-compose.yml
#### Launch Bloodhound
sudo docker-compose pull && docker-compose up
###### In this step we Will search into the logs of compose the temp-password


##Detener todos los contenedores en ejecución
docker stop $(docker ps -q)
##Eliminar todos los contenedores
docker rm $(docker ps -aq)
##Eliminar todos los volúmenes
docker volume rm $(docker volume ls -q)
##Eliminar todas las imágenes (opcional, si quieres limpiar completamente)
docker rmi $(docker images -q)


##Install bloodhound-python
pipx install bloodhound
####Use
bloodhound-python -u 'USUARIO' -p 'CONTRASEÑA' -d 'DOMINIO.LOCAL' -c All -v --zip

##Import bloodhound dates
1ºVe a http://localhost:8080/ui/login
2ºInicia sesión con las credenciales de admin
3ºArrastra y sube el archivo ZIP generado por bloodhound-python
```

> Edition 2025

```
# Update & Upgrade System
sudo apt update && sudo apt upgrade

# Install Docker
sudo apt install docker.io && sudo apt install docker-compose
##Add u user to the Docker group
sudo usermod -aG docker $USER

```
