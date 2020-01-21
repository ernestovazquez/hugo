---
title: "T3. Instalación aplicaciones web."
date: 2019-10-30T12:20:59+01:00
draft: false
---

Creamos la base de datos llamada **wordpress**

```
root@tortilla:/home/ubuntu# mysql -u root
```

```
MariaDB [(none)]> CREATE DATABASE wordpress;
MariaDB [(none)]> use wordpress;
MariaDB [wordpress]> GRANT ALL PRIVILEGES ON *.* TO 'salmorejo'@'10.0.0.7' IDENTIFIED BY 'salmorejo' WITH GRANT OPTION;
```

Nos conectamos remotamente:

```
[centos@salmorejo ~]$ mysql --host=10.0.0.14 -u salmorejo -p wordpress
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 39
Server version: 10.1.41-MariaDB-0ubuntu0.18.04.1 Ubuntu 18.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [wordpress]> 
```

Configuración de **wordpress.conf**

```
[centos@salmorejo ~]$ sudo nano /etc/nginx/conf.d/wordpress.conf 

server {
  listen 80;

  root /var/www/wordpress;
  index index.html index.php;

  server_name www.ernesto.gonzalonazareno.org;


  location ~ \.php$ {
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_index index.php;
    include fastcgi_params;
  }
}

```

Instalamos **Wordpress**:
Posteriormente he modificado el directorio donde está **ubicado** la aplicación **wordpress** y ahora está situado en **/var/www/wordpress**

```
[centos@salmorejo]$ cd /usr/share/nginx/html/
[centos@salmorejo ~]$ wget https://wordpress.org/latest.tar.gz
[centos@salmorejo ~]$ tar xzvf latest.tar.gz
```

Permisos:

```
[centos@salmorejo html]$ sudo chown nginx:nginx -R /usr/share/nginx/html/wordpress/
[centos@salmorejo ~]$ cd /usr/share/nginx/html/
[centos@salmorejo html]$ cd wordpress/
[centos@salmorejo wordpress]$ sudo mv * ..
```

```
[centos@salmorejo html]$ sudo find /usr/share/nginx/html -type f -exec chmod 0644 {} \;
[centos@salmorejo html]$ sudo find /usr/share/nginx/html -type d -exec chmod 0755 {} \;
```

![wordpress](/img/wordpress.png)

Activamos Selinux:

```
[centos@salmorejo ~]$ setsebool -P httpd_can_network_connect_db=1
[centos@salmorejo ~]$ getsebool -a | grep httpd
    httpd_can_network_connect_db --> on
```

![instalawordpress](/img/instalawordpress.png)

![instalawordpress2](/img/instalawordpress2.png)

![instalawordpress3](/img/instalawordpress3.png)

Agregamos en el fichero de configuración lo siguiente:

```
        location / {
            index index.php index.htm;
        }
```

Ya tendremos **wordpress instalado** y funcionando.

![wordpress8](/img/wordpress8.png)

Creamos la base de datos llamada **nextcloud**

```
root@tortilla:/home/ubuntu# mysql -u root -p 
```

```
MariaDB [(none)]> CREATE DATABASE nextcloud;
MariaDB [(none)]> use nextcloud;
MariaDB [nextcloud]> GRANT ALL PRIVILEGES ON nextcloud.* TO salmorejo@10.0.0.7;
```

Nos conectamos remotamente:

```
[centos@salmorejo ~]$ mysql --host=10.0.0.14 -u salmorejo -p nextcloud
Enter password:

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 87
Server version: 10.1.41-MariaDB-0ubuntu0.18.04.1 Ubuntu 18.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [nextcloud]> 
```

Descargamos **nextcloud** y lo descomprimimos 

```
[root@salmorejo centos]# cd /tmp/
[root@salmorejo tmp]# wget https://download.nextcloud.com/server/releases/nextcloud-17.0.1.zip
[root@salmorejo tmp]# unzip nextcloud-17.0.1.zip -d /usr/share/nginx/
```

Archivo de **configuración** de **nextcloud**:

```
[root@salmorejo ~]# nano /etc/nginx/conf.d/nextcloud.conf
```

```
server {
    listen 80;
    server_name cloud.ernesto.gonzalonazareno.org;

    # Add headers to serve security related headers
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header X-Download-Options noopen;
    add_header X-Permitted-Cross-Domain-Policies none;
    add_header Referrer-Policy no-referrer;

    #I found this header is needed on Debian/Ubuntu/CentOS/RHEL, but not on Arch Linux.
    add_header X-Frame-Options "SAMEORIGIN";

    # Path to the root of your installation
    root /var/www/nextcloud/;

    access_log /var/log/nginx/nextcloud.access;
    error_log /var/log/nginx/nextcloud.error;

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    # The following 2 rules are only needed for the user_webfinger app.
    # Uncomment it if you're planning to use this app.
    #rewrite ^/.well-known/host-meta /public.php?service=host-meta last;
    #rewrite ^/.well-known/host-meta.json /public.php?service=host-meta-json
    # last;

    location = /.well-known/carddav {
        return 301 $scheme://$host/remote.php/dav;
    }
    location = /.well-known/caldav {
       return 301 $scheme://$host/remote.php/dav;
    }

    location ~ /.well-known/acme-challenge {
      allow all;
    }

    # set max upload size
    client_max_body_size 512M;
    fastcgi_buffers 64 4K;

    # Disable gzip to avoid the removal of the ETag header
    gzip off;

    # Uncomment if your server is build with the ngx_pagespeed module
    # This module is currently not supported.
    #pagespeed off;

    error_page 403 /core/templates/403.php;
    error_page 404 /core/templates/404.php;

    location / {
       rewrite ^ /index.php$uri;
    }

    location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)/ {
       deny all;
    }
    location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console) {
       deny all;
     }

    location ~ ^/(?:index|remote|public|cron|core/ajax/update|status|ocs/v[12]|updater/.+|ocs-provider/.+|core/templates/40[34])\.php(?:$|/) {
       include fastcgi_params;
       fastcgi_split_path_info ^(.+\.php)(/.*)$;
       fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
       fastcgi_param PATH_INFO $fastcgi_path_info;
       #Avoid sending the security headers twice
       fastcgi_param modHeadersAvailable true;
       fastcgi_param front_controller_active true;
       fastcgi_pass unix:/var/run/php-fpm.sock;
#     fastcgi_pass 127.0.0.1:9000;
       fastcgi_intercept_errors on;
       fastcgi_request_buffering off;
       fastcgi_read_timeout 300;
    }

    location ~ ^/(?:updater|ocs-provider)(?:$|/) {
       try_files $uri/ =404;
       index index.php;
    }

    # Adding the cache control header for js and css files
    # Make sure it is BELOW the PHP block
    location ~* \.(?:css|js)$ {
        try_files $uri /index.php$uri$is_args$args;
        add_header Cache-Control "public, max-age=7200";
        # Add headers to serve security related headers (It is intended to
        # have those duplicated to the ones above)
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Robots-Tag none;
        add_header X-Download-Options noopen;
        add_header X-Permitted-Cross-Domain-Policies none;
        # Optional: Don't log access to assets
        access_log off;
   }

   location ~* \.(?:svg|gif|png|html|ttf|woff|ico|jpg|jpeg)$ {
        try_files $uri /index.php$uri$is_args$args;
        # Optional: Don't log access to other assets
        access_log off;
   }
}
```

Probamos con un **test** y si todo va bien activamos los servicios de **nginx**:

```
[root@salmorejo ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

[root@salmorejo ~]# systemctl restart nginx
```

Ejecutamos el siguientes comandos para instalar los módulos PHP requeridos o recomendados por NextCloud.

```
[root@salmorejo ~]# yum install php-common php-gd php-json php-curl php-zip php-xml php-mbstring php-bz2 php-intl
```

Configuramos **SELinux** para que permita que PHP-FPM use **execmem**.

```
[root@salmorejo ~]# setsebool -P httpd_execmem 1
[root@salmorejo ~]# systemctl reload php-fpm
```

Configuración de **permisos**

Permisos de lectura y escritura sobre el directorio de trabajo

```
chcon -t httpd_sys_rw_content_t /usr/share/nginx/nextcloud/ -R
setsebool -P httpd_can_network_connect 1
```

Cambio de propietarios en estos 3 archivos, de apache a nginx.

```
setfacl -R -m u:nginx:rwx /var/lib/php/opcache/
setfacl -R -m u:nginx:rwx /var/lib/php/session/
setfacl -R -m u:nginx:rwx /var/lib/php/wsdlcache/
```

```
[root@salmorejo www]# chown -R nginx: nextcloud/
```

Configuración **SELinux:**

```
[root@salmorejo nextcloud]# semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/nextcloud/data(/.*)?'
[root@salmorejo nextcloud]# semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/nextcloud/config(/.*)?'
[root@salmorejo nextcloud]# semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/nextcloud/apps(/.*)?'
[root@salmorejo nextcloud]# semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/nextcloud/assets(/.*)?'
[root@salmorejo nextcloud]# semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/nextcloud/.htaccess'
[root@salmorejo nextcloud]# semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/nextcloud/.user.ini'
[root@salmorejo nextcloud]# restorecon -Rv '/var/www/nextcloud/'
```

![nextcloud](/img/nextcloud.png)

Instalamos el **módulo PHP** :

```
[root@salmorejo ~]# yum install php73-php-pecl-zip.x86_64
```

![nextcloud2](/img/nextcloud2.png)

Nos da el error **504 Gateway Timeout error**:

```
[root@salmorejo ~]# sudo nano /etc/php.ini

    max_execution_time = 300
```

```
[root@salmorejo ~]# sudo nano /etc/php-fpm.d/www.conf

    request_terminate_timeout = 300
```

```
[root@salmorejo ~]# sudo nano /etc/nginx/conf.d/nextcloud.conf

    fastcgi_read_timeout 300;
```

Tras el error 404 añadimos lo siguiente en el fichero de configuración de **nextcloud**

```
  location / {
    try_files $uri $uri/ /index.php?$args;
    }
```

Cuando intento **iniciar sesión** , no entra en mi cuenta administrador:

![loginnextcloud](/img/loginnextcloud.png)

Para que funcione correctamente y entre he modificado lo siguiente:

```
[root@salmorejo ~]# nano /etc/php-fpm.d/www.conf 

user = nginx
group = nginx

;listen = 127.0.0.1:9000
listen = /var/run/php-fpm.sock

listen.owner = nginx
listen.group = nginx
```

En el archivo de configuración de nextcloud he cambiado lo siguiente:

```
fastcgi_pass unix:/var/run/php-fpm.sock;
```

También he intentado a reinstalar el nextcloud con lo siguiente:

```
[root@salmorejo ~]# rm /var/www/nextcloud/config/config.php
[root@salmorejo ~]# touch /var/www/nextcloud/config/CAN_INSTALL
[root@salmorejo ~]# chown nginx:nginx /var/www/nextcloud/config/CAN_INSTALL 
```

```
[root@salmorejo ~]# semanage fcontext -a -t httpd_sys_rw_content_t '/var/lib/php/'
[root@salmorejo ~]# restorecon -Rv '/var/lib/php/'
```

Ya estaría configurado y **funcionando** el servicio de **nextcloud** en nuestro **servidor web nginx**

![nextcloudend](/img/nextcloudend.png)

![nextcloudend](/img/nextcloudend1.png)
