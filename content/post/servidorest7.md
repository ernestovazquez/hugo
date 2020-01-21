---
title: "T7. Hosting."
date: 2019-12-11T14:29:52+01:00
draft: false
---

Queremos que diferentes usuarios, puedan gestionar una página web en vuestro servidor que esté gestionada por medio de un FTP. También se creará una base de datos para cada usuario.

Por ejemplo, el usuario josedom quiere hacer una página cuyo nombre será servicios:

- La página que vamos a crear será accesible en https://servicios.tunombre.gonzalonazareno.org.
- Se creará un usuario user_josedom, que tendrá una contraseña, para que accediendo a ftp.tunombre.gonzalonazareno.org, pueda gestionar los ficheros de su página web.
- Se creará un usuario en la base de datos llamado myjosedom. Este usuario tendrá una contraseña distinta a la del usuario del servidor FTP.
- Se creará una bases de datos para el usuario anteriormente creado. Para que los usuarios gestionen su base de datos se puede instalar la aplicación phpmyadmin a la que se accederá con la URL https://sql.tunombre.gonzalonmazareno.org.

Tarea: Configura manualmente los distintos servicios para crear un nuevo usuario que gestione su propia página web y tenga una base de datos a su disposición. Instala un CMS.
Mejora 1: Modifica la configuración del sistema para que se usen usuarios virtuales para el acceso por FTP, cuya información este guardada en vuestro directorio ldap.
Mejora 2: Realiza un script que automatice la creación/borrado de nuevos usuarios en el hosting.



Vamos a instalar **phpMyAdmin**:

Extensión para conectar php con la **base de datos**:

```
[centos@salmorejo ~]$ sudo dnf install -y php-json php-mbstring
```

Instalamos **phpMyAdmin**

```
[centos@salmorejo ~]$ sudo wget https://files.phpmyadmin.net/phpMyAdmin/4.9.0.1/phpMyAdmin-4.9.0.1-all-languages.tar.gz
[centos@salmorejo ~]$ sudo tar -zxvf phpMyAdmin-4.9.0.1-all-languages.tar.gz 
[centos@salmorejo ~]$ mv phpMyAdmin-4.9.0.1-all-languages /usr/share/phpMyAdmin
```

Configuramos php:

```
[centos@salmorejo ~]$ sudo cp -pr /usr/share/phpMyAdmin/config.sample.inc.php /usr/share/phpMyAdmin/config.inc.php
[centos@salmorejo ~]$ sudo mkdir -p /var/lib/phpmyadmin/tmp
[centos@salmorejo ~]$ sudo chown -R nginx:nginx /var/lib/phpmyadmin
[centos@salmorejo ~]$ sudo nano /usr/share/phpMyAdmin/config.inc.ph
```

Añadimos lo siguiente en el fichero:

```
$cfg['blowfish_secret'] = 'o,]G;E8c,.O.EkhANLhLhd0e}g]L}XaN'; /* YOU MUST FILL IN THIS FOR COOKIE AUTH! */
$cfg['TempDir'] = '/var/lib/phpmyadmin/tmp';
$cfg['Servers'][$i]['host'] = '10.0.0.4';
```

Para generar el **blowfish**, lo hacemos mediante el siguiente generador: https://phpsolved.com/phpmyadmin-blowfish-secret-generator/?g=5cecac771c51c

Importamos la base de datos:

```
[centos@salmorejo ~]$ mysql < /usr/share/phpMyAdmin/sql/create_tables.sql --host=10.0.0.4 -u salmorejo -p phpmyadmin
```
Virtual Host para **phpMyAdmin**:

```
[centos@salmorejo ~]$ sudo nano /etc/nginx/conf.d/phpMyAdmin.conf

server {
   listen 80;
   server_name sql.ernesto.gonzalonazareno.org;
   root /usr/share/phpMyAdmin;

   location / {
      index index.php;
   }

## Images and static content is treated different
   location ~* ^.+.(jpg|jpeg|gif|css|png|js|ico|xml)$ {
      access_log off;
      expires 30d;
   }

   location ~ /\.ht {
      deny all;
   }

   location ~ /(libraries|setup/frames|setup/libs) {
      deny all;
      return 404;
   }

   location ~ \.php$ {
      include /etc/nginx/fastcgi_params;
#      fastcgi_pass 127.0.0.1:9000;
      fastcgi_pass unix:/var/run/php-fpm/www.sock;
      fastcgi_index index.php;
      fastcgi_param SCRIPT_FILENAME /usr/share/phpMyAdmin$fastcgi_script_name;
   }
}
```

Creamos el directorio **tmp** y le cambiamos los permisos:

```
[centos@salmorejo ~]$ sudo mkdir /usr/share/phpMyAdmin/tmp
[centos@salmorejo ~]$ sudo chmod 777 /usr/share/phpMyAdmin/tmp
[centos@salmorejo ~]$ sudo chown -R nginx:nginx /usr/share/phpMyAdmin
```

Reiniciamos los servicios:

```
[centos@salmorejo ~]$ sudo systemctl restart nginx 
[centos@salmorejo ~]$ sudo systemctl restart php-fpm
```

Reglas del firewall para http:

```
[centos@salmorejo ~]$ sudo semanage fcontext -a -t httpd_sys_content_t "/usr/share/phpMyAdmin(/.*)?"
[centos@salmorejo ~]$ sudo restorecon -Rv /usr/share/phpMyAdmin
```

```
[centos@salmorejo ~]$ sudo firewall-cmd --permanent --add-service=http
Warning: ALREADY_ENABLED: http
success

[centos@salmorejo ~]$ sudo firewall-cmd --reload
success
```

![phpmyadmin](/img/phpmyadmin.png)

Instalamos **FTP** :

Repositorios EPEL:

```
[centos@salmorejo ~]$ sudp wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
[centos@salmorejo ~]$ sudo dnf install epel-release-latest-7.noarch.rpm -y --allowerasing
```

Instalamos **proftpd** y los paquetes necesarios:

```
[centos@salmorejo ~]$ sudo wget http://mirror.centos.org/centos/7/os/x86_64/Packages/GeoIP-1.5.0-14.el7.x86_64.rpm
[centos@salmorejo ~]$ sudo dnf install GeoIP-1.5.0-14.el7.x86_64.rpm  --allowerasing

[centos@salmorejo ~]$ wget http://mirror.centos.org/centos/7/os/x86_64/Packages/tcp_wrappers-7.6-77.el7.x86_64.rpm
[centos@salmorejo ~]$ sudo dnf install tcp_wrappers-7.6-77.el7.x86_64.rpm

[centos@salmorejo ~]$ sudo dnf install proftpd -y
```

Puertos del firewall:

```
[centos@salmorejo ~]$ sudo firewall-cmd --add-service=ftp --permanent --zone=public
success

[centos@salmorejo ~]$ sudo firewall-cmd --reload
success
```

Iniciamos los servicios:

```
[centos@salmorejo ~]$ sudo systemctl start proftpd

[centos@salmorejo ~]$ sudo systemctl enable proftpd
Created symlink /etc/systemd/system/multi-user.target.wants/proftpd.service → /usr/lib/systemd/system/proftpd.service.
```

Creamos el **Virtual Host**:

```
[centos@salmorejo ~]$ sudo nano /etc/nginx/conf.d/user_ernesto.conf

server {
  listen 80;

  root /var/www/user_ernesto;
  index index.html index.php;

  server_name informatica.ernesto.gonzalonazareno.org;

  location / {
      try_files $uri $uri/ /index.php?$args;
      autoindex on;
      disable_symlinks if_not_owner;
  }

  location ~ \.php$ {
    fastcgi_pass unix:/var/run/php-fpm/www.sock;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_index index.php;
    include fastcgi_params;
  }
}
```

Creamos el directorio:

```
[centos@salmorejo ~]$ sudo mkdir /usr/share/nginx/html/user_ernesto
[centos@salmorejo ~]$ sudo chown -R nginx:nginx /usr/share/nginx/html/user_ernesto/
[centos@salmorejo ~]$ sudo find /usr/share/nginx/html/user_ernesto -type f -exec chmod 0644 {} \;
[centos@salmorejo ~]$ sudo find /usr/share/nginx/html/user_ernesto -type d -exec chmod 0755 {} \;
[centos@salmorejo ~]$ sudo chcon -t httpd_sys_content_t /usr/share/nginx/html/user_ernesto -R
```

Cambiamos el directorio del usuario:

```
[centos@salmorejo ~]$ sudo mv /usr/share/nginx/html/user_ernesto /var/www/
```

Reiniciamos el servicio:

```
[centos@salmorejo ~]$ sudo systemctl restart nginx
[centos@salmorejo ~]$ sudo systemctl restart php-fpm
```

Creamos el usuario (passwd: evazgar123):

```
[centos@salmorejo ~]$ sudo useradd user_ernesto
[centos@salmorejo ~]$ sudo passwd user_ernesto
```

Configuración FTP:

```
[centos@salmorejo ~]$ sudo nano /etc/proftpd.conf 

DefaultRoot                     /var/www/%u
```

Reiniciamos:

```
[centos@salmorejo ~]$ sudo systemctl restart proftpd
```

![informaticaphp](/img/informaticaphp.png)

Agregamos el regla del grupo de seguridad de openstack: **TCP 	21** 

Añadimos la regla SELinux:

```
[centos@salmorejo ~]$ sudo firewall-cmd --add-service=ftp --permanent
Warning: ALREADY_ENABLED: ftp
success

[centos@salmorejo ~]$ sudo firewall-cmd --reload
success

[centos@salmorejo ~]$ sudo setsebool -P allow_ftpd_anon_write=1
[centos@salmorejo ~]$ sudo setsebool -P allow_ftpd_full_access=1
[centos@salmorejo ~]$ sudo setsebool -P allow_ftpd_use_cifs=1
[centos@salmorejo ~]$ sudo setsebool -P allow_ftpd_use_nfs=1
[centos@salmorejo ~]$ sudo setsebool -P ftpd_connect_all_unreserved=1
[centos@salmorejo ~]$ sudo setsebool -P ftpd_connect_db=1
[centos@salmorejo ~]$ sudo systemctl restart proftpd
[centos@salmorejo ~]$ sudo systemctl enable proftpd
[centos@salmorejo ~]$ sudo systemctl start proftpd
```

Probamos si en localhost funciona:

```
[centos@salmorejo ~]$ ftp localhost

Trying ::1...
Connected to localhost (::1).
220 FTP Server ready.
Name (localhost:centos): user_ernesto
331 Password required for user_ernesto
Password:

230 User user_ernesto logged in
Remote system type is UNIX.
Using binary mode to transfer files.

ftp> ls
229 Entering Extended Passive Mode (|||54923|)
150 Opening ASCII mode data connection for file list
-rw-r--r--   1 user_ernesto user_ernesto       69 Dec 16 08:41 phpinfo.php
-rw-r--r--   1 user_ernesto user_ernesto       20 Dec 17 09:07 prueba.txt
226 Transfer complete

ftp> 221 Goodbye.
```

Nos conectamos mediante nuestra máquina:

```
ernesto@honda:~$ ftp ftp.ernesto.gonzalonazareno.org

Connected to salmorejo.ernesto.gonzalonazareno.org.
220 FTP Server ready.
Name (ftp.ernesto.gonzalonazareno.org:ernesto): user_ernesto
331 Password required for user_ernesto
Password:

230 User user_ernesto logged in
Remote system type is UNIX.
Using binary mode to transfer files.

ftp> ls
200 PORT command successful
150 Opening ASCII mode data connection for file list
-rw-r--r--   1 user_ernesto user_ernesto       69 Dec 16 08:41 phpinfo.php
-rw-r--r--   1 user_ernesto user_ernesto       20 Dec 17 09:07 prueba.txt
226 Transfer complete

ftp> 
```

Prueba desde **filezilla**:

![user_ernestofilezilla](/img/user_ernestofilezilla.png)

Descargamos wordpress para hacer la prueba de la funcion del ftp desde el equipo:

Subimos wordpress y ya podremos instalarlo en el directorio del usuario:

![wordpressuser_ernesto](/img/wordpressuser_ernesto.png)

Creamos los usuarios y la base de datos:

```
MariaDB [(none)]> CREATE USER "my_ernesto" IDENTIFIED BY "my_ernesto";
Query OK, 0 rows affected (0.03 sec)

MariaDB [(none)]> CREATE DATABASE db_ernesto;
Query OK, 1 row affected (0.04 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON db_ernesto.* TO my_ernesto;
Query OK, 0 rows affected (0.04 sec)
```

Lo añadimos para crear un blog nuevo:

![loginuser_ernesto](/img/loginuser_ernesto.png)

Y tendriamos creado el blog de este usuario.

![accesouser_ernesto](/img/accesouser_ernesto.png)

![enduser_ernesto](/img/enduser_ernesto.png)
