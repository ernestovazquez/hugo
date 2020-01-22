---
title: "T5. Actualización a CentOS 8."
date: 2019-11-11T13:10:28+01:00
draft: false
---

Actualiza salmorejo a CentOS 8 garantizando que todos los servicios previamente configurados sigan funcionando correctamente.

***

Instalamos los **repositorios** de **EPEL** :

```
[root@salmorejo centos]# yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```

Instalamos **yum-utils** :

```
[root@salmorejo centos]# yum -y install rpmconf yum-utils
[root@salmorejo centos]# rpmconf -a
```

**Limpiamos** paquetes innecesarios: 

```
[root@salmorejo centos]# package-cleanup --leaves
[root@salmorejo centos]# package-cleanup --orphans
```

Instalamos el software **DNF** :

```
[root@salmorejo centos]# yum -y install dnf
```

Eliminamos los paquetes manager:

```
[root@salmorejo centos]# dnf -y remove yum yum-metadata-parser
[root@salmorejo centos]# rm -Rf /etc/yum
```

**Actualizamos** el sistema con DNF:

```
[centos@salmorejo ~]$ sudo dnf -y upgrade
```

Instalamos la **nueva versión** :

```
[root@salmorejo centos]# dnf -y upgrade http://mirror.bytemark.co.uk/centos/8/BaseOS/x86_64/os/Packages/centos-release-8.0-0.1905.0.9.el8.x86_64.rpm
```

**Actualizamos** los **repositorios** de EPEL:

```
[root@salmorejo centos]# dnf -y upgrade https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
```

**Limpieza** ficheros temporales:

```
[root@salmorejo centos]# dnf clean all
```

Eliminamos los kernel:

```
[root@salmorejo centos]# rpm -e `rpm -q kernel`
[root@salmorejo centos]# rpm -e --nodeps sysvinit-tools
```

```
[root@salmorejo centos]# dnf -y --releasever=8 --allowerasing --setopt=deltarpm=false distro-sync
```

Confirmamos la instalación del kernel-core:

```
[root@salmorejo centos]# rpm -e kernel-core
[root@salmorejo centos]# dnf -y install kernel-core
```

Confirmacion del **grub** :

```
[root@salmorejo centos]# ROOTDEV=`ls /dev/*da|head -1`;
[root@salmorejo centos]# echo "Detected root as $ROOTDEV..."
[root@salmorejo centos]# grub2-install $ROOTDEV
```

Instalación de **minimal package**:

```
[root@salmorejo centos]# dnf -y groupupdate "Core" "Minimal Install" --skip-broken
```

Vemos la **versión** de **CentOS**

```
[centos@salmorejo ~]$ cat /etc/centos-release
```

Quitamos este paquete que da problemas de dependencia

```
[centos@salmorejo ~]$ sudo rpm --nodeps -e gdbm-1.10-8.el7.x86_64
```

![centos8](/img/centos8.png)

Vamos a ver si funcionan correctamente los **servicios previos** :

![centos8nginx](/img/centos8nginx.png)

![centos8php](/img/centos8php.png)

![centos8wordpress](/img/centos8wordpress.png)

![centos8nextcloud](/img/centos8nextcloud.png)

Tenemos que solucionar los errores y conflictos que tenemos despues de la actualización.

En mi caso me sale error de la librería libzip para ello, tenemos que borrarla.

	sudo dnf remove libzip
	sudo dnf update

El error de la librería libgdbm, haremos lo siguiente:

	sudo rpm --nodeps -e gdbm
	sudo dnf -y upgrade --best --allowerasing
	sudo dnf update


--------------------------------------------------------------------------------
Instalación del nuevo kernel:

	sudo rpmconf -a

	sudo rpm -e kernel-core
	sudo dnf -y install kernel-core


Cambios para el buen funcionamiento de los servicios instalados antes de la actualización a CentOS 8.

He tenido que instalar de nuevo nginx y php-fpm

Nos dirigimos al fichero de configuración de nextcloud y wordpress y añadimos lo siguiente:

	[centos@salmorejo ~]$ sudo nano /etc/nginx/conf.d/nextcloud.conf 
	[centos@salmorejo ~]$ sudo nano /etc/nginx/conf.d/wordpress.conf 

		fastcgi_pass unix:/var/run/php-fpm/www.sock;

Cambio en el fichero de configuración de php-fpm

	[centos@salmorejo ~]$ nano /etc/php-fpm.d/www.conf 

		listen = /var/run/php-fpm/www.sock;
		user = nginx
		group = nginx
		listen.owner = nginx
		listen.group = nginx

Si la página web no se puede cargar, probablemente deba abrir el puerto 80 y 443 en el firewall.
Cambios en el firewall:

	[root@salmorejo centos]# firewall-cmd --permanent --zone=public --add-service=http
	[root@salmorejo centos]# firewall-cmd --permanent --zone=public --add-service=https
	[root@salmorejo centos]# systemctl reload firewalld

Cambio de permisos 

	[centos@salmorejo ~]$ sudo chown -R nginx:nginx /var/lib/php/session

También como se ha cambiado la red interna del servidor he tenido que cambiar manualmente la configuración del acceso remoto al servidor de la base de datos, en este caso a tortilla. Cambiamos la ip del servidor (10.0.0.14 --> 10.0.0.4)

	[centos@salmorejo ~]$ sudo nano /var/www/nextcloud/config/config.php 

		'dbhost' => '10.0.0.4',

	[centos@salmorejo ~]$ sudo nano /var/www/wordpress/wp-config.php 

		/** MySQL hostname */
		define( 'DB_HOST', '10.0.0.4' );

Reiniciamos los servicios.

	[centos@salmorejo ~]$ sudo systemctl restart php-fpm
	[centos@salmorejo ~]$ sudo systemctl restart nginx

Comprobaciones de actualización:

	[centos@salmorejo ~]$ dnf check-update

Remi's Modular repository for Enterprise Linux 8 - x86_64                                                             222 kB/s | 518 kB     00:02    
Safe Remi's RPM repository for Enterprise Linux 8 - x86_64                                                            428 kB/s | 1.4 MB     00:03    
Remi's RPM repository for Enterprise Linux 8 - x86_64                                                                 820 kB/s | 2.6 MB     00:03    

Comprobaciones del kérnel:

	[root@salmorejo ~]# uname -r

	4.18.0-80.11.2.el8_0.x86_64


Comprobaciones:

![statusphp](/img/statusphp.png)

![statusnginx](/img/statusnginx.png)

![webwordpress](/img/webwordpress.png)

![webcloud](/img/webcloud.png)

