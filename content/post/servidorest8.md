---
title: "T8. Instalación de aplicación python."
date: 2019-12-11T15:29:57+01:00
draft: false
---

En esta tarea vamos a desplegar un CMS python. Hemos elegido Mezzanine, pero puedes elegir otro CMS python basado en django.

- Instala el CMS en el entorno de desarrollo. Debes utilizar un entorno virtual.
- Personaliza la página (cambia el nombre al blog y pon tu nombre) y añade contenido (algún artículo con alguna imagen).
- Guarda los ficheros generados durante la instalación en un repositorio github. Guarda también en ese repositorio la copia de seguridad de la bese de datos.
- Realiza el despliegue de la aplicación en tu entorno de producción (servidor web y servidor de base de datos en el cloud). Utiliza un entorno virtual. Como servidor de aplicación puedes usar gunicorn o uwsgi (crea una unidad systemd para gestionar este servicio). La aplicación será accesible en la url python.tunombre.gonzalonazareno.org.

***

#### Entorno de producción

Creamos el entorno virtual en nuestra máquina.

Instalamos **Mezzanine** con:

```
(produccionmazzine) ernesto@honda:~/Documentos$ pip install mezzanine
```

Creamos el proyecto:

```
(produccionmazzine) ernesto@honda:~/Documentos$ mezzanine-project mezzanineernesto
```

Creamos la base de datos:

```
(produccionmazzine) ernesto@honda:~/Documentos/mezzanineernesto$ python manage.py createdb
```

Probamos si **funciona** en **local** :

```
(produccionmazzine) ernesto@honda:~/Documentos/mezzanineernesto$ python manage.py runserver
```

![runserver](/img/runserver.png)

Subimos el directorio a nuestro **GitHub**

```
(produccionmazzine) ernesto@honda:~/Documentos$ git clone git@github.com:ernestovazquez/mezzanineernesto.git
(produccionmazzine) ernesto@honda:~/Documentos/mezzanineernesto$ git add --all
(produccionmazzine) ernesto@honda:~/Documentos/mezzanineernesto$ git commit -am "produccion foto"
(produccionmazzine) ernesto@honda:~/Documentos/mezzanineernesto$ git push
```

Configuración contenido estático.

```
(produccionmazzine) ernesto@honda:~/Documentos/mezzanineernesto$ python manage.py collectstatic
```

Personalizamos la página cambiándole el nombre 

![nombremezza](/img/nombremezza.png)

Añadimos una foto.

![fotomezza1](/img/fotomezza1.png)

![cambiosmezza](/img/cambiosmezza.png)

Copia de seguridad de la base de datos:

```
(produccionmazzine) ernesto@honda:~/Documentos/mezzanineernesto$ python manage.py dumpdata > db.json

(produccionmazzine) ernesto@honda:~/Documentos/mezzanineernesto$ ls
db.json  deploy  dev.db  fabfile.py  manage.py  mezzanineernesto  README.md  requirements.txt  static

(produccionmazzine) ernesto@honda:~/Documentos/mezzanineernesto$ python manage.py dumpdata admin > admin.json

(produccionmazzine) ernesto@honda:~/Documentos/mezzanineernesto$ ls
admin.json  db.json  deploy  dev.db  fabfile.py  manage.py  mezzanineernesto  README.md  requirements.txt  static
```

Subimos los cambios a GitHub:

```
(produccionmazzine) ernesto@honda:~/Documentos/mezzanineernesto$ git status
En la rama master
Tu rama está actualizada con 'origin/master'.

Cambios no rastreados para el commit:
  (usa "git add <archivo>..." para actualizar lo que será confirmado)
  (usa "git checkout -- <archivo>..." para descartar los cambios en el directorio de trabajo)

	modificado:     dev.db

Archivos sin seguimiento:
  (usa "git add <archivo>..." para incluirlo a lo que se será confirmado)

	admin.json
	db.json

sin cambios agregados al commit (usa "git add" y/o "git commit -a")
```

```
(produccionmazzine) ernesto@honda:~/Documentos/mezzanineernesto$ git add --all

(produccionmazzine) ernesto@honda:~/Documentos/mezzanineernesto$ git commit -am "copia de seguridad bd"
[master 79aa9ef] copia de seguridad bd
 3 files changed, 2 insertions(+)
 create mode 100644 admin.json
 create mode 100644 db.json

(produccionmazzine) ernesto@honda:~/Documentos/mezzanineernesto$ git push
Enumerando objetos: 7, listo.
Contando objetos: 100% (7/7), listo.
Compresión delta usando hasta 4 hilos
Comprimiendo objetos: 100% (5/5), listo.
Escribiendo objetos: 100% (5/5), 3.57 KiB | 3.57 MiB/s, listo.
Total 5 (delta 2), reusado 0 (delta 0)
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
To github.com:ernestovazquez/mezzanineernesto.git
   a262af4..79aa9ef  master -> master
```

#### Trabajamos en el entorno de **desarrollo**.

Clonamos el repositorio en el **DocumentRoot**

```
[centos@salmorejo ~]$ cd /var/www/
[centos@salmorejo www]$ sudo git clone https://github.com/ernestovazquez/iaw_mezzanine.git
```

Instalamos **gunicorn** y sus paquetes necesarios.

```
[centos@salmorejo ~]$ sudo dnf install python36 python36-devel
```

Entorno virtual:

```
[root@salmorejo]# python3 -m venv produccionmezz

[root@salmorejo]# source produccionmezz/bin/activate
(produccionmezz) [root@salmorejo mezzanineernesto]# 
```

Instalamos los paquetes necesarios del fichero **requirements.txt** 

```
(produccionmezz) [centos@salmorejo iaw_mezzanine]$ pip install -r requirements.txt
```

Creamos la base de datos:

```
ubuntu@tortilla:~$ sudo mysql -u root -p

MariaDB [(none)]> CREATE DATABASE mezzaninedb;
Query OK, 1 row affected (0.04 sec)

MariaDB [(none)]> CREATE USER mezzanine identified by 'mezzanine';
Query OK, 0 rows affected (0.03 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON mezzaninedb.* to mezzanine;
Query OK, 0 rows affected (0.01 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.02 sec)

MariaDB [(none)]> 
```

Configuración para conectarse a la base de datos:

```
(produccionmezz) [centos@salmorejo iaw_mezzanine]$ sudo nano iaw_mezzanine/settings.py 

DATABASES = {
    "default": {
        # Add "postgresql_psycopg2", "mysql", "sqlite3" or "oracle".
        "ENGINE": "mysql.connector.django",
        # DB name or path to database file if using sqlite3.
        "NAME": "mezzaninedb",
        # Not used with sqlite3.
        "USER": "mezzanine",
        # Not used with sqlite3.
        "PASSWORD": "mezzanine",
        # Set to empty string for localhost. Not used with sqlite3.
        "HOST": "sql.ernesto.gonzalonazareno.org",
        # Set to empty string for default. Not used with sqlite3.
        "PORT": "",
    }
}
```

Configuramos el **Allowed_hosts** para permitir la dirección:

```
ALLOWED_HOSTS = ['python.ernesto.gonzalonazareno.org']
``` 

Conector python-mysql y eliminamos el anterior para que no tenga conflictos:

```
(produccionmezz) [centos@salmorejo iaw_mezzanine]$ pip install mysql-connector-python

(desarrollo) [root@salmorejo mezzanineernesto]# sudo rm /var/www/mezzanineernesto/dev.db 
```

Borramos el siguiente fichero:

Antes de borrar este fichero tendremos que copiar la **SECRET_KEY** y la **NEVERCACHE_KEY** en el fichero de configuración **settings.py** .

```
(desarrollo) [root@salmorejo mezzanineernesto]# rm local_settings.py 
```

Migración de la base de datos:

```
(produccionmezz) [centos@salmorejo iaw_mezzanine]$ python3 manage.py migrate

Operations to perform:
  Apply all migrations: admin, auth, blog, conf, contenttypes, core, django_comments, forms, galleries, generic, pages, redirects, sessions, sites, twitter
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying sites.0001_initial... OK
  Applying blog.0001_initial... OK
  Applying blog.0002_auto_20150527_1555... OK
  Applying blog.0003_auto_20170411_0504... OK
  Applying conf.0001_initial... OK
  Applying core.0001_initial... OK
  Applying core.0002_auto_20150414_2140... OK
  Applying django_comments.0001_initial... OK
  Applying django_comments.0002_update_user_email_field_length... OK
  Applying django_comments.0003_add_submit_date_index... OK
  Applying pages.0001_initial... OK
  Applying forms.0001_initial... OK
  Applying forms.0002_auto_20141227_0224... OK
  Applying forms.0003_emailfield... OK
  Applying forms.0004_auto_20150517_0510... OK
  Applying forms.0005_auto_20151026_1600... OK
  Applying forms.0006_auto_20170425_2225... OK
  Applying galleries.0001_initial... OK
  Applying galleries.0002_auto_20141227_0224... OK
  Applying generic.0001_initial... OK
  Applying generic.0002_auto_20141227_0224... OK
  Applying generic.0003_auto_20170411_0504... OK
  Applying pages.0002_auto_20141227_0224... OK
  Applying pages.0003_auto_20150527_1555... OK
  Applying pages.0004_auto_20170411_0504... OK
  Applying redirects.0001_initial... OK
  Applying sessions.0001_initial... OK
  Applying sites.0002_alter_domain_unique... OK
  Applying twitter.0001_initial... OK
```

```
(produccionmezz) [centos@salmorejo iaw_mezzanine]$ python3 manage.py loaddata CopiaBaseDatos.json

Installed 151 object(s) from 1 fixture(s)
```

Ya podremos instalar **gunicorn**

```
(produccionmezz) [centos@salmorejo iaw_mezzanine]$ pip install gunicorn

Collecting gunicorn
  Using cached https://files.pythonhosted.org/packages/69/ca/926f7cd3a2014b16870086b2d0fdc84a9e49473c68a8dff8b57f7c156f43/gunicorn-20.0.4-py2.py3-none-any.whl
Requirement already satisfied: setuptools>=3.0 in /home/centos/produccionmezz/lib/python3.6/site-packages (from gunicorn) (39.2.0)
Installing collected packages: gunicorn
Successfully installed gunicorn-20.0.4
```

VirtualHost:

```
(entornoproduccion) [centos@salmorejo iaw_mezzanine1]$ sudo nano /etc/nginx/conf.d/wordpress.conf 

server {
  listen 80;

  root /var/www/wordpress;
  index index.html index.php;

  server_name www.ernesto.gonzalonazareno.org;


  location ~ \.php$ {
    fastcgi_pass unix:/var/run/php-fpm/www.sock;
#    fastcgi_pass 127.0.0.1:9000;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_index index.php;
    include fastcgi_params;
  }
}
```

Cambiamos los **permisos** al directorio **static**:

```
[centos@salmorejo iaw_mezzanine1]$ sudo chown -R centos:centos /var/www/iaw_mezzanine1
[centos@salmorejo iaw_mezzanine1]$ python3 manage.py collectstatic
[centos@salmorejo iaw_mezzanine1]$ sudo chown -R nginx:nginx /var/www/iaw_mezzanine1
```

Reglas de SELinux:

```
[centos@salmorejo iaw_mezzanine1]$ sudo setsebool -P httpd_can_network_connect on
[centos@salmorejo iaw_mezzanine1]$ sudo find /var/www/iaw_mezzanine1 -type f -exec chmod 0644 {} \;
[centos@salmorejo iaw_mezzanine1]$ sudo find /var/www/iaw_mezzanine1 -type d -exec chmod 0755 {} \;
[centos@salmorejo iaw_mezzanine1]$ sudo chcon -t httpd_sys_rw_content_t /var/www/iaw_mezzanine1 -R
```

Y ponemos el servidor **gunicorn** en funcionamiento:

```
(entornoproduccion) [centos@salmorejo iaw_mezzanine1]$ gunicorn -w 2 -b :8080 iaw_mezzanine.wsgi:application &
```

Añadimos una nueva entrada al **DNS**

```
python          IN      CNAME   salmorejo
```

![pythonernestogonzalo](https://raw.githubusercontent.com/ernestovazquez/hugo/gh-pages/img/pythonernestogonzalo.png)

![pythonernesto](https://raw.githubusercontent.com/ernestovazquez/hugo/gh-pages/img/pythonernesto.png)

