COMANDOS BÁSICOS

# Esse comando serve para criar os container
sudo docker compose up -d

# Esse comando serve para ver o container
sudo docker ps

# Se for necessário ver os logs basta dar o comando baixo
# sudo docker logs nomedocontainer

#Em alguns casos, será necessário rodar o comando abaixo:
#docker inspect zabbix-agent2 | grep "IPAddress\": "

#Caso queria ver os últimos 50 logs.

docker logs --tail 50 --follow --timestamps zabbix-agent2

#Para entrar no bash de algum container,
docker exec -it id-do-seu-container /bin/bash
