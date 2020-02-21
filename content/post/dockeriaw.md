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

Realizaremos la imagen docker de la aplicación a partir de la imagen oficial PHP: 

```
root@bookmedik:~/tarea2# nano Dokerfile

FROM php:7.0-apache

ADD . /var/www/html

RUN docker-php-ext-install mysqli && docker-php-ext-enable mysqli
```

Subimos la imagen a DockerHub.


