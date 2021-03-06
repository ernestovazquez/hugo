---
title: "Despliegue de CMS java"
date: 2020-01-30T09:04:52+01:00
draft: false
---

En esta práctica vamos a desplegar un CMS escrito en java. Puedes escoger la aplicación que vas a desplegar de CMS escritos en Java o de Aplicaciones Java en Bitnami.

- Indica la aplicación escogida y su funcionalidad.
- Escribe una guía de los pasas fundamentales para realizar la instalación.
- ¿Has necesitado instalar alguna librería?¿Has necesitado instalar un conector de una base de datos?
- Entrega una captura de pantalla donde se vea la aplicación funcionando.
- Realiza la configuración necesaria en apache2 y tomcat (utilizando el protocolo AJP) para que la aplicación sea servida por el servidor web.
- Entrega una captura de pantalla donde se vea la aplicación funcionando servida por apache2.

***

Voy a instalar el CMS **OpenCMS**.
Estará en una máquina Debian con una base de datos MySQL.

Como su propio nombre indica, **OpenCMS** es un software de código abierto. Las caracteristicas que lo diferencian del resto son, su **sencillez de uso** y la **facilidad** con que se pueden elaborar y administrar los contenidos.

## Instalación y configuración:

Primero vamos a instalar los paquetes necesarios:

    vagrant@java:~$ sudo apt install openjdk-11-jre mariadb-server tomcat9 apache2

Los siguiente que tenemos que haces es descargar el CMS:

```
root@java:~# wget http://www.opencms.org/downloads/opencms/opencms-10.5.3.zip


root@java:~# unzip opencms-10.5.3.zip
Archive:  opencms-10.5.3.zip
  inflating: opencms.war             
  inflating: install.html            
  inflating: history.txt             
  inflating: license.txt     
  
root@java:~# ls
history.txt  install.html  license.txt	opencms-10.5.3.zip  opencms.war
```
Copiamos el ***.war*** al su correcto directorio 

    vagrant@java:~/opencms$ sudo cp opencms.war /var/lib/tomcat9/webapps/

Eliminamos ROOT por defecto y añadimos el nuevo.

```
root@java:~# cd /var/lib/tomcat9/webapps/
root@java:/var/lib/tomcat9/webapps# rm -r ROOT/
root@java:~# cp opencms.war /var/lib/tomcat9/webapps/ROOT.war
```

Por último tendremos que **reiniciar tomcat:**

    vagrant@java:~$ sudo systemctl restart tomcat9

Ya podremos entrar desde el navegador.

![](https://i.imgur.com/loDViLc.png)

![](https://i.imgur.com/tKZnfwC.png)

Creamos la base de datos:

```
MariaDB [(none)]> create database opencms;
Query OK, 1 row affected (0.027 sec)

MariaDB [(none)]> CREATE USER 'open'@'localhost' IDENTIFIED BY 'open';
Query OK, 0 rows affected (0.009 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON opencms.* to 'open'@'localhost';
Query OK, 0 rows affected (0.002 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.004 sec)
```

![](https://i.imgur.com/JihUey1.png)

![](https://i.imgur.com/uBsaXM6.png)

Tendremos que editar el siguiente fichero de configuración de MySQL:

```
vagrant@java:~$ sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf 

max_allowed_packet     = 32M
```

Tendremos que reiniciar:

    vagrant@java:~$ sudo systemctl restart mariadb.service 

![](https://i.imgur.com/sRDw5uo.png)

![](https://i.imgur.com/yQOzU7Y.png)

Le damos a continuar y es importarán los modulos necesarios, para su correcto funcionamiento.

![](https://i.imgur.com/Veczp7J.png)

Ya tendremos intalado OpenCMS correctamente:

![](https://i.imgur.com/A2MfqHG.png)

Por último marcamos **"Finish"**, para terminar.

![](https://i.imgur.com/HQkLGgC.png)

Ya tendremos el CMS instalado:

![](https://i.imgur.com/nrXbRMe.png)

## Conexión de Tomcat 9 y Apache2

Para que uestra página web pueda ser consultada a través del puerto por defecto para HTTP, haremos lo siguiente:

Entramos en el siguiente fichero de configuración:

```
root@java:~# nano /etc/tomcat9/server.xml 


    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               URIEncoding="utf8"
               redirectPort="8443" />
```

Esta conexión se realiza a través del módulo proxy_http

```
root@java:~# nano /etc/apache2/sites-enabled/000-default.conf 

        ProxyRequests Off
        ProxyPreserveHost On

        ProxyPass / http://localhost:8080/
        ProxyPassReverse / http://localhost:8080/
```

Reiniciamos los servicios:

```
root@java:~# systemctl restart apache2
root@java:~# systemctl restart tomcat9
```

![](https://i.imgur.com/FdGsJvn.png)

Ya tendremos OpenCMS instalado sobre Debian con MySQL y Tomcat 9 mediante Apache2.


