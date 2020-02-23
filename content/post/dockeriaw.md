---
title: "Implantación de aplicaciones web PHP en docker"
date: 2020-02-18T11:50:52+01:00
draft: false
---

Queremos ejecutar en un contenedor docker la aplicación web escrita en PHP: [bookMedik](https://github.com/evilnapsis/bookmedik).

***

## Tarea 1


**Ejecución de la aplicación web PHP bookMedik en docker**

* Queremos ejecutar en un contenedor docker la aplicación web escrita en PHP: bookMedik (https://github.com/evilnapsis/bookmedik).
* Es necesario tener un contenedor con mariadb donde vamos a crear la base de datos y los datos de la aplicación. El script para generar la base de datos y los registros lo encuentras en el repositorio y se llama schema.sql. Debes crear un usuario con su contraseña en la base de datos. La base de datos se llama bookmedik y se crea al ejecutar el script.
* Ejecuta el contenedor mariadb y carga los datos del script schema.sql. Para más información.
* El contenedor mariadb debe tener un volumen para guardar la base de datos.
* Crea una imagen docker con la aplicación desde una imagen base de debian o ubuntu. Ten en cuenta que el fichero de configuración de la base de datos (core\controller\Database.php) lo tienes que configurar utilizando las variables de entorno del contenedor mariadb. (Nota: Para obtener las variables de entorno en PHP usar la función getenv. Para más infomación).
* La imagen la tienes que crear en tu máquina con el comando docker build.
* Crea un contenedor a partir de la imagen anterior, enlazado con el contenedor mariadb, y comprueba que está funcionando (Usuario: admin, contraseña: admin)
* El contenedor que creas debe tener un volumen para guardar los logs de apache2.

***

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

**Ejecución de una aplicación web PHP con imagenes de PHP y apache2 de DockerHub**

* Realiza la imagen docker de la aplicación a partir de la imagen oficial PHP que encuentras en docker hub. Lee la documentación de la imagen para configurar una imagen con apache2 y php, además seguramente tengas que instalar alguna extensión de php.
* Crea esta imagen en docker hub.
* Crea un script con docker compose que levante el escenario con los dos contenedores.

***

Vamos a editar el Dockerfile:

```
root@docker:~/tarea2# nano Dockerfile

FROM php:7.4.3-apache
ENV MARIADB_USER bookmedik
ENV MARIADB_PASS bookmedik
ENV MARIADB_HOST servidor_mysqltarea2
RUN docker-php-ext-install pdo pdo_mysql mysqli json
RUN a2enmod rewrite
EXPOSE 80
WORKDIR /var/www/html
COPY ./bookmedik /var/www/html
ADD script.sh /usr/local/bin/script.sh

RUN chmod +x /usr/local/bin/script.sh

CMD ["/usr/local/bin/script.sh"]
```

A continuación realizamos la imagen:

    root@docker:~/tarea2# docker build -t ernestovazquez/bookmediktarea2:v1 .

**Contenedores**:

```
root@docker:~/tarea2# docker run -d --name servidor_mysqltarea2 --network bookmedik -v /opt/bbdd_mariadb:/var/lib/mysql -e MYSQL_DATABASE=bookmedik -e MYSQL_USER=bookmedik -e MYSQL_PASSWORD=bookmedik -e MYSQL_ROOT_PASSWORD=asdasd mariadb
412d52fe93ed2892431e02d02c3713b01dee73fc7af0a0c536d1d3e001f7b57f

root@docker:~/tarea2# docker run -d --name bookmediktarea2 --network bookmedik -v /opt/logs_apache2:/var/log/apache2 -p 80:80 ernestovazquez/bookmediktarea2:v1
11ed11ddd0ec41c00c19ed8ef4ac26e8eec0b75a34bd16a2a4129baaa896e87a
```

![](https://i.imgur.com/IWSzcDF.png)

Para crear esta imagen en docker hub haremos lo siguiente:

    root@docker:~/tarea2# docker push ernestovazquez/bookmediktarea2:v1 


Ahora vamos a crear un script con docker compose que levante el escenario con los dos contenedores.

```
version: '3.1'

services:
  bookmedikphp:
    container_name: bookmediktarea2
    image: php:7.4.3-apache
    restart: always
    environment:
      MARIADB_USER: bookmedik
      MARIADB_PASS: bookmedik
      MARIADB_HOST: servidor_mysqltarea2
    ports:
      - 80:80
    volumes:
      - /opt/logs_apache2:/var/log/apache2
      - ./script.sh:/usr/local/bin/script.sh
      - ./bookmedik:/var/www/html
    command: >
      bash /usr/local/bin/script.sh
  servidor_mysql2:
    container_name: servidor_mysqltarea2
    image: mariadb
    restart: always
    environment:
      MYSQL_DATABASE: bookmedik
      MYSQL_USER: bookmedik
      MYSQL_PASSWORD: bookmedik
      MYSQL_ROOT_PASSWORD: asdasd
    volumes:
      - /opt/bbdd_mariadb:/var/lib/mysql
```

Añadimos la siguiente linea al script que hemos creado anteriormente:

    docker-php-ext-install pdo pdo_mysql mysqli json

Levantamos docker-compose:

```
root@docker:~/tarea2# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

root@docker:~/tarea2# docker-compose up -d
Creating network "tarea2_default" with the default driver
Creating bookmediktarea2      ... done
Creating servidor_mysqltarea2 ... done

root@docker:~/tarea2# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
01e16d302760        mariadb             "docker-entrypoint.s…"   4 seconds ago       Up 3 seconds        3306/tcp             servidor_mysqltarea2
1574cc3753ff        php:7.4.3-apache    "docker-php-entrypoi…"   4 seconds ago       Up 3 seconds        0.0.0.0:80->80/tcp   bookmediktarea2
```

![](https://i.imgur.com/vAlZ0cQ.png)

https://github.com/ernestovazquez/docker-tarea2

## Tarea 3


**Ejecución de un CMS en docker**


* En este caso queremos usar un contenedor que utilice nginx para servir la aplicación PHP. Puedes crear la imagen desde una imagen base debian o ubuntu o desde la imagen oficial de nginx.
* Vamos a crear otro contenedor que sirva php-fpm.
* Y finalmente nuestro contenedor con la aplicación.
* Crea un script con docker compose que levante el escenario con los tres contenedores.

A lo mejor te puede ayudar el siguiente enlace: Dockerise your PHP application with Nginx and PHP7-FPM


***

## Tarea 4


* A partir de una imagen base (que no sea una imagen con el CMS), genera una imagen que despliegue un CMS PHP (que no sea wordpress). El contenedor que se crea a partir de esta imagen se tendrá que enlazar con un contenedor mariadb o postgreSQL.
* Crea los volúmenes necesarios para que la información que se guarda sea persistente.

***

## Tarea 5

**Ejecución de un CMS en docker con una imagen de DockerHub**

* Busca una imagen oficial de un CMS PHP en docker hub (distinto al que has instalado en la tarea anterior, ni wordpress), y crea los contenedores necesarios para servir el CMS, siguiendo la documentación de docker hub.

***

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


