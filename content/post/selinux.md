---
title: "Configuración/activación de SELinux"
date: 2020-02-13T19:04:52+01:00
draft: false
---

Habilita SELinux en salmorejo y asegúrate de que todas las aplicaciones web funcionan correctamente con una configuración estricta y segura de SELinux

***

## Intoducción

**SELinux** es una característica de seguridad de Linux que provee una variedad de políticas de seguridad  que otorga a los administradores mayor control sobre las personas que pueden acceder al sistema y con el objetivo de proteger sus recursos.

**SELinux** ha sido integrado a la rama principal del núcleo Linux desde la versión 2.6, agosto de 2003. Mientras que en CentOS y RHEL está habilitado por defecto.

Lo primero que vamos a ver es si tenemos habilitado SELinux.
 
```
[centos@salmorejo ~]$ /usr/sbin/getenforce
Enforcing
```

Este comando va a chequear el estado y nos va a devolver la salida de **Enforcing**, **Disabled** o **Permissive**. En nuestro caso está en **"Enforcing"**. 

- **Enforcing:** Cuando SELinux está habilitado y las reglas de la política de SELinux son aplicadas. 
- **Disabled:** En el caso de que SELinux está desactivado.
- **Permissive:** SELinux aplica las políticas pero no toma acciones, únicamente registra y alerta al administrador de que se ha violado alguna regla.

El comando **sestatus** devuelve el **estado** de SELinux y la **política** de SELinux que se está usando: 

```
[centos@salmorejo ~]$ getenforce
Enforcing

[centos@salmorejo ~]$ sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      31

```

Para ver la lista de usuario pondremos lo siguiente:

```
[centos@salmorejo ~]$ sudo semanage user -l

                Etiquetado MLS/       MLS/                          
Usuario SELinux  Prefijo    Nivel MCS  Rango MCS                      Roles SELinux

guest_u         user       s0         s0                             guest_r
root            user       s0         s0-s0:c0.c1023                 staff_r sysadm_r system_r unconfined_r
staff_u         user       s0         s0-s0:c0.c1023                 staff_r sysadm_r system_r unconfined_r
sysadm_u        user       s0         s0-s0:c0.c1023                 sysadm_r
system_u        user       s0         s0-s0:c0.c1023                 system_r unconfined_r
unconfined_u    user       s0         s0-s0:c0.c1023                 system_r unconfined_r
user_u          user       s0         s0                             user_r
xguest_u        user       s0         s0                             xguest_r
```

Un **usuario SELinux** está asociado a un conjunto de usuarios UNIX, en está lista podremos encontrar dichos usuarios.

## Habilitar SELinux

Vamos a ver como activar o desactivar SELinux.

Para ello nos dirigimos al siguiente fichero de configuración:

```
[centos@salmorejo ~]$ nano /etc/selinux/config 

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

En el caso de querer **desactivar SELinux**, solamente tendremos que ponerlo en **disabled**.

```
...
SELINUX=disabled
...
```

<div id='id1' />

## SELinux Booleans

Nos permiten cambiar partes de la política de SELinux en tiempo de ejecución, sin ningún conocimiento sobre la escritura de políticas de SELinux. 

Esto permite cambios, como permitir el acceso, sin recargar o recompilar la política de SELinux. 

Podemos ver la lista con el siguiente comando:

```
[centos@salmorejo ~]$ getsebool -a | grep httpd
httpd_anon_write --> off
httpd_builtin_scripting --> on
httpd_can_check_spam --> off
httpd_can_connect_ftp --> off
httpd_can_connect_ldap --> off
httpd_can_connect_mythtv --> off
httpd_can_connect_zabbix --> off
httpd_can_network_connect --> on
httpd_can_network_connect_cobbler --> off
httpd_can_network_connect_db --> on
httpd_can_network_memcache --> off
httpd_can_network_relay --> off
httpd_can_sendmail --> off
httpd_dbus_avahi --> off
httpd_dbus_sssd --> off
httpd_dontaudit_search_dirs --> off
httpd_enable_cgi --> on
httpd_enable_ftp_server --> off
httpd_enable_homedirs --> off
httpd_execmem --> on
httpd_graceful_shutdown --> off
httpd_manage_ipa --> off
httpd_mod_auth_ntlm_winbind --> off
httpd_mod_auth_pam --> off
httpd_read_user_content --> off
httpd_run_ipa --> off
httpd_run_preupgrade --> off
httpd_run_stickshift --> off
httpd_serve_cobbler_files --> off
httpd_setrlimit --> off
httpd_ssi_exec --> off
httpd_sys_script_anon_write --> off
httpd_tmp_exec --> off
httpd_tty_comm --> off
httpd_unified --> off
httpd_use_cifs --> off
httpd_use_fusefs --> off
httpd_use_gpg --> off
httpd_use_nfs --> off
httpd_use_openstack --> off
httpd_use_sasl --> off
httpd_verify_dns --> off
```

Si queremos ver solamente los que están activos pondremos lo siguiente:

```
[centos@salmorejo ~]$ getsebool -a | egrep ' on' | egrep 'httpd'
httpd_builtin_scripting --> on
httpd_can_network_connect --> on
httpd_can_network_connect_db --> on
httpd_enable_cgi --> on
httpd_execmem --> on
```

Para habilitar la conexión solamente tendremos que poner la siguiente regla.

    [centos@salmorejo ~]$ sudo setsebool -P httpd_can_network_connect on

Para desactivarlo tendremos que ponerlo en ***off***. También podremos hacerlo ejecutando **(0/1).**

Si queremos ver el estado de uno en concreto solamente tendremos que ponerlo con **getsebool**.

```
[centos@salmorejo ~]$ sudo getsebool httpd_can_network_connect
httpd_can_network_connect --> on
```

Referencia: [Documentación de CentOS sobre SelinuxBooleans](https://wiki.centos.org/TipsAndTricks/SelinuxBooleans)

## Contextos de SELinux - Etiquetado de Archivos

En este apartado vamos a ver **como gestiona SELinux los contextos**, ya que los **procesos y archivos** son etiquetados con información de seguridad relevante.

Nos seria útil para cambiar la localización de los servicios que viene por defecto en el sistema.

Para listar todos los contextos podemos utilizar el siguiente comando:

    [centos@salmorejo ~]$ sudo semanage fcontext -l

Vamos a ver el mismo **ejemplo** de antes pero con sus **respectivos contextos**, lo podriamos hacer filtrando el comando con **grep** y ponemos la **regla deseada**.

```
[centos@salmorejo ~]$ sudo semanage fcontext -l | grep httpd_sys_content_t

/etc/htdig(/.*)?                                   all files          system_u:object_r:httpd_sys_content_t:s0 
/srv/([^/]*/)?www(/.*)?                            all files          system_u:object_r:httpd_sys_content_t:s0 
/srv/gallery2(/.*)?                                all files          system_u:object_r:httpd_sys_content_t:s0 
/usr/share/doc/ghc/html(/.*)?                      all files          system_u:object_r:httpd_sys_content_t:s0 
/usr/share/drupal.*                                all files          system_u:object_r:httpd_sys_content_t:s0 
/usr/share/glpi(/.*)?                              all files          system_u:object_r:httpd_sys_content_t:s0 
/usr/share/htdig(/.*)?                             all files          system_u:object_r:httpd_sys_content_t:s0 
/usr/share/icecast(/.*)?                           all files          system_u:object_r:httpd_sys_content_t:s0 
/usr/share/nginx/html(/.*)?                        all files          system_u:object_r:httpd_sys_content_t:s0 
/usr/share/ntop/html(/.*)?                         all files          system_u:object_r:httpd_sys_content_t:s0 
/usr/share/openca/htdocs(/.*)?                     all files          system_u:object_r:httpd_sys_content_t:s0 
/usr/share/phpmyadmin(/.*)?                        all files          system_u:object_r:httpd_sys_content_t:s0 
/usr/share/selinux-policy[^/]*/html(/.*)?          all files          system_u:object_r:httpd_sys_content_t:s0 
/usr/share/z-push(/.*)?                            all files          system_u:object_r:httpd_sys_content_t:s0 
/var/lib/cacti/rra(/.*)?                           all files          system_u:object_r:httpd_sys_content_t:s0 
/var/lib/htdig(/.*)?                               all files          system_u:object_r:httpd_sys_content_t:s0 
/var/lib/trac(/.*)?                                all files          system_u:object_r:httpd_sys_content_t:s0 
/var/www(/.*)?                                     all files          system_u:object_r:httpd_sys_content_t:s0 
/var/www/icons(/.*)?                               all files          system_u:object_r:httpd_sys_content_t:s0 
/var/www/svn/conf(/.*)?                            all files          system_u:object_r:httpd_sys_content_t:s0 
```

En **la salida del comando** podemos apreciar el **directorio** que va a poder escribir la aplicación, los **ficheros** y por último la **gestión de usuarios**, donde tendremos, en primer lugar el **usuario**, a continuación el **objeto** y por último el **tipo de acceso**.

Ahora que hemos **listado los diferentes contextos** de una regla, vamos a ver como se puede **añadir más reglas**.

Para ello pondremos el siguiente comando:

    [centos@salmorejo ~]$ sudo semanage fcontext -a -t  httpd_sys_content_t "/var/www(/.*)?"
    
Como podemos apreciar, tendriamos que poner el nombre de la regla, en este caso `httpd_sys_content_t`, seguido del directorio que queremos que escriba la aplicacion web. Con ese simple comando podemos agregar un nuevo contexto.

## Configuración para las aplicaciones webs

### phpMyAdmin

La **primera regla de SELinux** que vamos a configurar va a ser la de **phpMyAdmin**, para su correcto funcionamiento con http.

```
[centos@salmorejo ~]$ sudo semanage fcontext -a -t httpd_sys_content_t "/usr/share/phpMyAdmin(/.*)?" 
[centos@salmorejo ~]$ sudo restorecon -Rv /usr/share/phpMyAdmin
```

Tengo que añadir que tenemos que tener activado httpd, como hemos hecho anteriormente en [SELinux Booleans](#id1).

### Proftpd

Para el correcto funcionamiento vamos a activar las reglas para **Proftpd**.

Para ellos vamos a documentarnos sobre las políticas de SELinux para demonios ftp. Para permitir el acceso de usuarios ftp a los directorios.

```
[centos@salmorejo ~]$ sudo setsebool -P allow_ftpd_anon_write=1
[centos@salmorejo ~]$ sudo setsebool -P allow_ftpd_full_access=1
[centos@salmorejo ~]$ sudo setsebool -P allow_ftpd_use_cifs=1
[centos@salmorejo ~]$ sudo setsebool -P allow_ftpd_use_nfs=1
[centos@salmorejo ~]$ sudo setsebool -P ftpd_connect_all_unreserved=1
[centos@salmorejo ~]$ sudo setsebool -P ftpd_connect_db=1
[centos@salmorejo ~]$ sudo systemctl restart proftpd
```
Referencia: [Documentación de CentOS sobre ftpd_selinux](http://www.polarhome.com/service/man/?qf=ftpd_selinux&tf=2&of=CentOS&sf=8)


### Mezzanine

A continuación vamos a ver las reglas que he configurado para el uso de mezzanine.

```
[centos@salmorejo iaw_mezzanine1]$ sudo setsebool -P httpd_can_network_connect on
[centos@salmorejo iaw_mezzanine1]$ sudo chcon -t httpd_sys_rw_content_t /var/www/iaw_mezzanine1 -R
```
Al igual que antes tenemos que tener activado `httpd_can_network_connect`, como hemos relizado anteriormente.

Lo que hemos realizado es permitir al usuario, leer y escribir sobre los directorios.

Lo he realizado con **chcon**, este es un cambio temporal, el cual cambia el contexto SELinux de los archivos.

Para el **uso con semanage** solamente tendremos que poner el siguiente comando:

    [centos@salmorejo ~]$ sudo semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/iaw_mezzanine1(/.*)?"


### Wordpress

Para wordpress vamos a permitir el acceso de aplicaciones web a objetos y a la base de datos.

```
[root@salmorejo ~]# setsebool -P httpd_can_network_connect 1
[root@salmorejo ~]$ setsebool -P httpd_can_network_connect_db=1
[root@salmorejo ~]$ getsebool -a | grep httpd
    httpd_can_network_connect_db --> on

[centos@salmorejo ~]$ sudo semanage fcontext -a -t httpd_sys_content_t "/var/www/wordpress(/.*)?"
```

### Nextcloud

Para nextcloud vamos a activar `httpd_execmem`.
```
[root@salmorejo ~]# setsebool -P httpd_execmem 1
```
Este permite que httpd ejecute programas que requieren direcciones de memoria que sean tanto ejecutables como grabables.

```
[centos@salmorejo ~]$ sudo setsebool -P httpd_can_network_connect 1
```

Directorios de Nextcloud:

```
[root@salmorejo nextcloud]# semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/nextcloud/data(/.*)?'
[root@salmorejo nextcloud]# semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/nextcloud/config(/.*)?'
[root@salmorejo nextcloud]# semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/nextcloud/apps(/.*)?'
[root@salmorejo nextcloud]# semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/nextcloud/assets(/.*)?'
[root@salmorejo nextcloud]# semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/nextcloud/.htaccess'
[root@salmorejo nextcloud]# semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/nextcloud/.user.ini'
[root@salmorejo nextcloud]# restorecon -Rv '/var/www/nextcloud/'
[root@salmorejo ~]# semanage fcontext -a -t httpd_sys_rw_content_t '/var/lib/php/'
[root@salmorejo ~]# restorecon -Rv '/var/lib/php/'
```

## Conclusión

**SELinux** es un gran sistema de control obligatorio de acceso con una gestión de permisos completamente distinta a la de los sistemas Unix tradicionales que estamos acostumbrados.

Tambíen he relizado parte de la práctica con **AppArmor** y tengo que destacar que me ha parecido más útil el uso de **SELinux**, aunque las dos son herramientas de seguridad con similitudes. Una **diferencia prácticamente notable** entre los dos sistemas está en cómo se aplican las reglas.

- **AppArmor** tienes que **crear perfiles** mientras escuchas el proceso. Funcionan directamente con rutas. 
- **SELinux** aplica etiquetas de seguridad a cada objeto y las reglas de control de acceso se escriben para esas etiquetas.

Estos métodos son pesados de implantar y de mantener. Ya que tienes que ir uno a uno para un correcto y completo método de seguridad. 

Seria interesante entretenerse un poco con ellos y aprender mucho más sobre esta potente herramienta de seguridad.
