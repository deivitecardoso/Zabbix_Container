Automatizando Zabbix com Docker em Container

Este script faz a instalação do Docker, cria a pasta onde ficará o arquivo de configuração do Docker Compose, e em seguida gera esse arquivo. Alguns campos no Docker Compose precisam ser ajustados conforme o seu ambiente:

DB_SERVER_HOST: Coloque o IP ou DNS do seu servidor Zabbix.

ZBX_PASSIVESERVERS: Coloque o IP ou DNS do seu servidor Zabbix.

Salve o script com o nome que preferir, mas recomendo algo como zabbix.sh (não esqueça do ".sh"). Também será necessário dar permissão para ele funcionar.

Passos para criar o script:

Para criar o arquivo:

vim zabbix.sh
Para dar permissão ao script:


chmod +x zabbix.sh

Conteúdo do script:

#!/bin/bash

# Instala Docker e atualiza o sistema
sudo curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo apt update

# Cria a pasta onde ficará o arquivo de configuração
sudo mkdir -p /home/zabbix/

# Cria o arquivo de configuração Docker Compose
cat <<EOF > /home/zabbix/docker-compose.yaml
version: '3.5'
services:
  zabbix-server:
    container_name: "zabbix-server"
    image: zabbix/zabbix-server-pgsql:alpine-trunk
    restart: always
    ports:
      - 10051:10051
    networks:
      - zabbix7
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro 
    environment:
      ZBX_CACHESIZE: 4096M # Se a máquina tiver menos de 1G de RAM, mude para 512M
      ZBX_HISTORYCACHESIZE: 1024M
      ZBX_HISTORYINDEXCACHESIZE: 1024M
      ZBX_TRENDCACHESIZE: 1024M
      ZBX_VALUECACHESIZE: 1024M
      DB_SERVER_HOST: "SEUIP" # Coloque aqui o IP do seu Zabbix
      DB_PORT: 5432
      POSTGRES_USER: "zabbix"
      POSTGRES_PASSWORD: "zabbix123"
      POSTGRES_DB: "zabbix_db"
    stop_grace_period: 30s

  zabbix-web-nginx-pgsql:
    container_name: "zabbix-web"
    image: zabbix/zabbix-web-nginx-pgsql:alpine-trunk
    restart: always
    ports:
      - 8080:8080
      - 8443:8443
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - ./cert/:/usr/share/zabbix/conf/certs/:ro
    networks:
      - zabbix7
    environment:
      DB_SERVER_HOST: "SEUIP" # Ajuste aqui com o IP do seu Zabbix
      DB_PORT: 5432
      POSTGRES_USER: "zabbix"
      POSTGRES_PASSWORD: "zabbix123"
      POSTGRES_DB: "zabbix_db"
      ZBX_MEMORYLIMIT: "1024M"

  zabbix-db-agent2:
    container_name: "zabbix-agent2"
    image: zabbix/zabbix-agent2:alpine-trunk
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - /run/docker.sock:/var/run/docker.sock
    environment:
      ZBX_HOSTNAME: "zabbix7"
      ZBX_SERVER_HOST: "127.0.0.1"
      ZBX_PASSIVESERVERS: "SEUIP" # Coloque o IP do seu Zabbix
    privileged: true
    ports:
      - 10050:10050
      - 31999:31999 

  db:
    container_name: "zabbix_db"
    image: postgres:15.6-bullseye
    restart: always
    volumes:
     - zbx_db15:/var/lib/postgresql/data
    ports:
     - 5432:5432
    networks:
     - zabbix7
    environment:
     POSTGRES_USER: "zabbix"
     POSTGRES_PASSWORD: "zabbix123"
     POSTGRES_DB: "zabbix_db"

networks:
  zabbix7:
   driver: bridge
volumes:
  zbx_db15:
EOF
