---
title: "Implantación de aplicaciones web PHP en docker"
date: 2020-02-18T11:50:52+01:00
draft: false
---

Queremos ejecutar en un contenedor docker la aplicación web escrita en PHP: [bookMedik](https://github.com/evilnapsis/bookmedik).

***

## Tarea 1

Creamos la red del contenedor:

```
root@bookmedik:~/tarea1# docker network create bookmedik
156229b4609abda50303cc7cd7e355bbdf317087f4c511e1a6ca93addd592b89
```

Contenedor de la base de datos:

```
root@bookmedik:~/tarea1# docker run -d --name servidor_mysql --network bookmedik -e MYSQL_DATABASE=bookmedik -e MYSQL_USER=bookmedik -e MYSQL_PASSWORD=bookmedik -e MYSQL_ROOT_PASSWORD=asdasd mariadb
```

Cargamos los datos en la base de datos

```
root@bookmedik:~/tarea1/bookmedik# cat schema.sql | docker exec -i servidor_mysql /usr/bin/mysql -u root --password=asdasd bookmedik
```

```
root@bookmedik:~/tarea1# nano Dockerfile 

FROM debian
RUN apt-get update && apt-get install -y apache2 libapache2-mod-php7.3 php7.3 php7.3-mysql && apt-get clean && rm -rf /var/lib/apt/lists/*
RUN rm /var/www/html/index.html
ENV APACHE_SERVER_NAME www.bookmedikernesto.com
ENV MARIADB_USER bookmedik
ENV MARIADB_PASS bookmedik
ENV MARIADB_HOST servidor_mysql

EXPOSE 80

COPY ./bookmedik /var/www/html
ADD script.sh /usr/local/bin/script.sh

RUN chmod +x /usr/local/bin/script.sh

CMD ["/usr/local/bin/script.sh"]
```

Script:

```
root@docker:~# cat script.sh 
#!/bin/bash
sed -i 's/$this->user="root";/$this->user="'${MARIADB_USER}'";/g' /var/www/html/core/controller/Database.php
sed -i 's/$this->pass="";/$this->pass="'${MARIADB_PASS}'";/g' /var/www/html/core/controller/Database.php
sed -i 's/$this->host="localhost";/$this->host="'${MARIADB_HOST}'";/g' /var/www/html/core/controller/Database.php
apache2ctl -D FOREGROUND
```

Imagen y contenedor:

```
root@docker:~# docker build -t ernestovazquez/bookmedik:v1 .

root@bookmedik:~/tarea1# docker run -d --name bookmedik --network bookmedik -p 80:80 ernestovazquez/bookmedik:v1
```

![](https://i.imgur.com/KAsftEG.png)

Volumen de la base de datos:

```
root@docker:~# docker run -d --name servidor_mysql --network bookmedik -v /opt/bbdd_mariadb:/var/lib/mysql -e MYSQL_DATABASE=bookmedik -e MYSQL_USER=bookmedik -e MYSQL_PASSWORD=bookmedik -e MYSQL_ROOT_PASSWORD=bookmedik mariadb
```

Log para apache

```
root@docker:~# docker run -d --name bookmedik --network bookmedik -v /opt/logs_apache2:/var/log/apache2 -p 80:80 ernestovazquez/bookmedik:v1

root@docker:~/bookmedik# cat schema.sql | docker exec -i servidor_mysql /usr/bin/mysql -u root --password=asdasd bookmedik
```

Vemos de nuevo la web:

![](https://i.imgur.com/8uQH0Ae.png)

```
root@docker:~# docker ps
CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS              PORTS                NAMES
e82d5e331fab        ernestovazquez/bookmedik:v1   "/usr/local/bin/scri…"   2 minutes ago       Up 2 minutes        0.0.0.0:80->80/tcp   bookmedik
f25dbff87579        mariadb                       "docker-entrypoint.s…"   3 minutes ago       Up 3 minutes        3306/tcp             servidor_mysql

root@docker:~# docker rm -f f25dbff87579
f25dbff87579
root@docker:~# docker rm -f e82d5e331fab
e82d5e331fab

root@docker:~# docker run -d --name servidor_mysql --network bookmedik -v /opt/bbdd_mariadb:/var/lib/mysql -e MYSQL_DATABASE=bookmedik -e MYSQL_USER=bookmedik -e MYSQL_PASSWORD=bookmedik -e MYSQL_ROOT_PASSWORD=asdasd mariadb
a59e9e77e97f3881bfec19853d0cad9ca7a738f42e2ee329310bc60fee4b659f

root@docker:~# docker run -d --name bookmedik --network bookmedik -v /opt/logs_apache2:/var/log/apache2 -p 80:80 ernestovazquez/bookmedik:v1
ae3d89739b7ad7ed843caa7ec4657f4699d2cd85e86f0b29625129209af2e4d6
```

![](https://i.imgur.com/t47Jnjj.png)

## Tarea 2

## Tarea 3

## Tarea 4

## Tarea 5

Vamos a crear el docker-compose para crear el contenedor de la base de datos para owncloud

```
root@docker:~/owncloud# nano docker-compose.yml

version: '3.1'

services:
  db:
    container_name: owncloud_db
    image: mariadb
    restart: always
    environment:
      MYSQL_DATABASE: oc_db
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - /opt/docker/volt5/db:/var/lib/mysql

  owncloud:
    container_name: owncloud
    image: owncloud
    restart: always
    ports:
      - 80:80
    volumes:
      - /opt/docker/volt5/owncloud/apps:/var/www/html/apps
      - /opt/docker/volt5/owncloud/config:/var/www/html/config
      - /opt/docker/volt5/owncloud/data:/var/www/html/data
```

Activamos el docker-compose:

```
root@docker:~/owncloud# docker-compose up -d
Creating network "owncloud_default" with the default driver
Pulling owncloud (owncloud:)...
latest: Pulling from library/owncloud
177e7ef0df69: Pull complete
9bf89f2eda24: Pull complete
350207dcf1b7: Pull complete
a8a33d96b4e7: Pull complete
c0421d5b63d6: Pull complete
f76e300fbe72: Pull complete
af9ff1b9ce5b: Pull complete
d9f072d61771: Pull complete
a6c512d0c2db: Pull complete
5a99458af5f8: Pull complete
8f2842d661a0: Pull complete
3c71c5361f06: Pull complete
baeacbad0a0c: Pull complete
e60049bf081a: Pull complete
0619078e32d3: Pull complete
a8e482ee2313: Pull complete
174d1b06857d: Pull complete
4a86c437f077: Pull complete
5e9ed4c3df2d: Pull complete
8a1479477c8e: Pull complete
8ab262044e9e: Pull complete
Digest: sha256:173811cb4c40505401595a45c39a802b89fb476885b3f6e8fe327aae08d20fe8
Status: Downloaded newer image for owncloud:latest
Creating owncloud_db ... done
Creating owncloud    ... done
```

Creamos un usuario y accedemos dentro

Hemos creado una carpeta de prueba para probar el volumen.

![](https://i.imgur.com/iGpdi64.png)

Borramos el contenedor y lo creamos de nuevo y vemos si siguen los archivos que hemos creado previamente

```
root@docker:~/owncloud# docker-compose stop
Stopping owncloud    ... done
Stopping owncloud_db ... done

root@docker:~/owncloud# docker-compose rm
Going to remove owncloud, owncloud_db
Are you sure? [yN] y
Removing owncloud    ... done
Removing owncloud_db ... done

root@docker:~/owncloud# docker-compose up -d
Creating owncloud_db ... done
Creating owncloud    ... done
root@docker:~/owncloud# 
```

![](https://i.imgur.com/yJXnaui.png)

Como podemos apreciar la carpeta sigue, ya que está en el volumen que hemos configurado con anterioridad para que la información sea persistente.
