
---
title: "Creación de un sistema de cuentas de usuarios en GNU/Linux con LDAP, Kerberos 5 y NFS4"
date: 2020-02-10T15:04:52+01:00
draft: true
---

Complementa el directorio instalado en croqueta creando un sistema centralizado de cuentas de usuarios en GNU/Linux con LDAP, Kerberos 5 y NFS4.

Utiliza el manual de http://albertomolina.files.wordpress.com/2014/01/krb_ldap.pdf

Ten en cuenta que en este caso es necesario que el directorio home del usuario esté centralizado y para eso es para lo que se utiliza NFSv4 con kerberos, que no aparece en la documentación.

Forma de corregir:

- Se hace login en tortilla con un usuario que esté en kerberos para lo que hay que facilitar la contraseña del principal.
- Los datos de la cuenta del usuario deben estar en el LDAP de croqueta
- Se comprueba que al hacer login se crea el TGT de kerberos y el TGS de LDAP mediante "klist -5"
- El directorio home del usuario debe estar disponible a través de NFS4y se comprueba creando un fichero
- Se accede por ssh a croqueta de forma transparente (no debe pedir contraseña ni usar clave pública/privada), el fichero creado anteriormente debe estar en el nuevo equipo.
- Se accede de nuevo por ssh a tortilla y se comprueba la generación correcta de los TGS

***

## Introducción

Vamos a realizar un sistema de cuentas de usuarios con **LDAP**, **Kerberos 5** y **NFS4**. 

**Kerberos** es diferente de los métodos de autenticación de nombre de usuario/contraseña. En vez de validar cada usuario para cada servicio de red, **Kerberos** autentifica los usuarios a un conjunto de servicios de red. 

## Configuración del DNS

Para empezar tenemos que hacer algunas **configuraciones previas**, una de ellas es la **configuración del DNS**, en nuestro caso con **Bind9**

Para ello editamos el siguiente fichero:

```
debian@croqueta:~$ sudo nano /var/cache/bind/db.ernesto.gonzalonazareno.org 

kerberos        IN      CNAME   croqueta
ldap            IN      CNAME   croqueta
_kerberos               IN      TXT     "ERNESTO.GONZALONAZARENO.ORG"
_kerberos._udp          IN      SRV     0 0 88          kerberos.ernesto.gonzalonazareno.org.
_kerberos_adm._tcp      IN      SRV     0 0 749         kerberos.ernesto.gonzalonazareno.org.
_ldap._tcp              IN      SRV     0 0 389         kerberos.ernesto.gonzalonazareno.org.
```

Posteriormente reiniciamos y probamos si hace bien la consulta mediante dig:

```
debian@croqueta:~$ sudo systemctl restart bind9

debian@croqueta:~$ dig ldap.ernesto.gonzalonazareno.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> ldap.ernesto.gonzalonazareno.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 42319
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 0e030eed954c2ff892fd7a2e5e53ab6b3c028f8af87f20ca (good)
;; QUESTION SECTION:
;ldap.ernesto.gonzalonazareno.org. IN	A

;; ANSWER SECTION:
ldap.ernesto.gonzalonazareno.org. 604784 IN CNAME croqueta.ernesto.gonzalonazareno.org.
croqueta.ernesto.gonzalonazareno.org. 604784 IN	A 172.22.200.111

;; AUTHORITY SECTION:
ernesto.gonzalonazareno.org. 86400 IN	NS	croqueta.ernesto.gonzalonazareno.org.

;; Query time: 1 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: Mon Feb 24 11:54:35 CET 2020
;; MSG SIZE  rcvd: 142
```

## Servidor de hora

Este paso es muy importante ya que ambas máquinas tienen que tener la misma fecha y hora.

Podremos usar ntp:

    debian@croqueta:~$ sudo apt install ntp

Solamente tendremos que añadir en el fichero `/etc/ntpd.conf` el servidor croqueta.

## Servidor LDAP 

Vamos a crear el usuario y el grupo en un nuevo fichero.

> Tengo que añadir, que las dos unidades organizativas (ou) llamadas People y Group, se han reutilizado de la práctica anterior de LDAP.

```
debian@croqueta:~/kerberos$ nano inicio.ldif

dn: ou=People,dc=ernesto,dc=gonzalonazareno,dc=org
ou: People
objectClass: top
objectClass: organizationalUnit

dn: ou=Group,dc=ernesto,dc=gonzalonazareno,dc=org
ou: Group
objectClass: top
objectClass: organizationalUnit

dn: cn=pruebagroup,ou=Group,dc=ernesto,dc=gonzalonazareno,dc=org
objectClass: posixGroup
objectClass: top
cn: pruebagroup
gidNumber: 2012

dn: uid=pruebauser,ou=People,dc=ernesto,dc=gonzalonazareno,dc=org
objectClass: account
objectClass: posixAccount
objectClass: top
cn: pruebauser
uid: pruebauser
loginShell: /bin/bash
uidNumber: 2510
gidNumber: 2012
homeDirectory: /home/users/pruebauser
```
Lo añadimos con el siguiente comando:

```
debian@croqueta:~/kerberos$ ldapadd -f inicio.ldif -x -D "cn=admin,dc=ernesto,dc=gonzalonazareno,dc=org" -W
Enter LDAP Password: 
adding new entry "cn=pruebagroup,ou=Group,dc=ernesto,dc=gonzalonazareno,dc=org"

adding new entry "uid=pruebauser,ou=People,dc=ernesto,dc=gonzalonazareno,dc=org"
```

A continuación tenemos que editar el fichero `/etc/ldap/ldap.conf.` Tanto en el cliente como en el servidor.

```
debian@croqueta:~$ sudo nano /etc/ldap/ldap.conf

BASE    dc=ernesto,dc=gonzalonazareno,dc=org
URI     ldap://ldap.ernesto.gonzalonazareno.org
```

Ahora vamos a crear el directorio y le asignamos el grupo correcto.

```
debian@croqueta:~$ sudo mkdir -p /home/users/pruebauser

debian@croqueta:~$ sudo cp /etc/skel/.bash_logout /home/users/pruebauser/
debian@croqueta:~$ sudo cp /etc/skel/.bashrc /home/users/pruebauser/
debian@croqueta:~$ sudo cp /etc/skel/.profile /home/users/pruebauser/

debian@croqueta:/home/users/pruebauser$ ls -al
total 20
drwxr-xr-x 2 root root 4096 Feb 24 12:52 .
drwxr-xr-x 3 root root 4096 Feb 24 12:51 ..
-rw-r--r-- 1 root root  220 Feb 24 12:53 .bash_logout
-rw-r--r-- 1 root root 3526 Feb 24 12:53 .bashrc
-rw-r--r-- 1 root root  807 Feb 24 12:53 .profile

debian@croqueta:~$ sudo chown -R 2510:2012 /home/users/pruebauser/

debian@croqueta:/home/users/pruebauser$ ls -al
total 20
drwxr-xr-x 2 2510 2012 4096 Feb 24 12:52 .
drwxr-xr-x 3 root root 4096 Feb 24 12:51 ..
-rw-r--r-- 1 2510 2012  220 Feb 24 12:53 .bash_logout
-rw-r--r-- 1 2510 2012 3526 Feb 24 12:53 .bashrc
-rw-r--r-- 1 2510 2012  807 Feb 24 12:53 .profile
```

## Configuración del cliente LDAP. Name Service Switch (NSS)

Para la autenticación de usuarios con LDAP debemos instalar el paquete `libnss-ldap`.

    debian@croqueta:~$ sudo apt install --no-install-recommends libnss-ldap

![](https://i.imgur.com/js6Rlss.png)

![](https://i.imgur.com/GNRasK7.png)

![](https://i.imgur.com/GPFFTR3.png)

Las siguientes pantallas las podemos ignorar de momento.

Si queremos volver a configurar solamente tendremos que poder lo siguiente:

    debian@croqueta:~$ sudo dpkg-reconfigure libnss-ldap

A continuación nos dirigimos al fichero de configuración `/etc/libnss-ldap.conf` parra comentar la siguiente línea

```
debian@croqueta:~$ sudo nano /etc/libnss-ldap.conf 

#rootbinddn cn=manager,dc=example,dc=net
```

En el caso de que exista contraseña del administrador de LDAP se borraría:

    debian@croqueta:~$ rm -f /etc/libnss-ldap.secret

Ahora vamos a configurar las líneas de passwd y group, donde le indicaremos al sistema que busque en directorios LDAP y files.

```
debian@croqueta:~$ sudo nano /etc/nsswitch.conf

passwd:         ldap files
group:          ldap files
```
Para comprobar el correcto funcionamiento de nss con ldap, realizaremos el mismo listado que antes.

```
debian@croqueta:~/kerberos$ ls -al /home/users/pruebauser/
total 20
drwxr-xr-x 2 pruebauser pruebagroup 4096 Feb 24 12:52 .
drwxr-xr-x 3 root       root        4096 Feb 24 12:51 ..
-rw-r--r-- 1 pruebauser pruebagroup  220 Feb 24 12:53 .bash_logout
-rw-r--r-- 1 pruebauser pruebagroup 3526 Feb 24 12:53 .bashrc
-rw-r--r-- 1 pruebauser pruebagroup  807 Feb 24 12:53 .profile
```

Como podemos apreciar se han cambiado el UID/GID por el correspondiente, obtenido mediante una consulta al directorio LDAP.

Otra forma de hacer la consulta es mediante el uso de `getent`, podemos verlo con el siguiente comando.

```
debian@croqueta:~$ getent passwd pruebauser 
pruebauser:*:2510:2012:pruebauser:/home/users/pruebauser:/bin/bash

debian@croqueta:~$ getent group pruebagroup 
pruebagroup:*:2012:
```

## Instalación del servidor MIT Kerberos 5

Vamos a instalar el servidor Kerberos y a la correspondiente configuración.

    debian@croqueta:~$ apt install krb5-kdc krb5-admin-server

![](https://i.imgur.com/rkeVckH.png)

Al finalizar la instalación nos dirigimos al siguiente fichero de configuración:

```
debian@croqueta:~$ sudo nano /etc/krb5kdc/kdc.conf

[kdcdefaults]
    kdc_ports = 750,88

[realms]
    ERNESTO.GONZALONAZARENO.ORG = {
        database_name = /var/lib/krb5kdc/principal
        admin_keytab = FILE:/etc/krb5kdc/kadm5.keytab
        acl_file = /etc/krb5kdc/kadm5.acl
        key_stash_file = /etc/krb5kdc/stash
        kdc_ports = 750,88
        max_life = 10h 0m 0s
        max_renewable_life = 7d 0h 0m 0s
        master_key_type = des3-hmac-sha1
        #supported_enctypes = aes256-cts:normal aes128-cts:normal
        default_principal_flags = +preauth
    }
```

Ahora tendremos que **quitar el puerto 750.**

```
[kdcdefaults]
    kdc_ports = 88

[realms]
    ERNESTO.GONZALONAZARENO.ORG = {
        database_name = /var/lib/krb5kdc/principal
        admin_keytab = FILE:/etc/krb5kdc/kadm5.keytab
        acl_file = /etc/krb5kdc/kadm5.acl
        key_stash_file = /etc/krb5kdc/stash
        kdc_ports = 88
        max_life = 10h 0m 0s
        max_renewable_life = 7d 0h 0m 0s
        master_key_type = des3-hmac-sha1
        #supported_enctypes = aes256-cts:normal aes128-cts:normal
        default_principal_flags = +preauth
    }
```

En caso de necesitar **desactivar Kerberos 4** nos dirigimos al fichero de configuración `/etc/default/krb5-kdc.`

```
KRB4_MODE = disable
RUN_KRB524D = false
```

Para deshabilitar por completo la utilización de Kerberos4.

Añadimos las siguientes líneas.

```
debian@croqueta:~$ sudo nano /etc/krb5.conf

[libdefaults]
        default_realm = ERNESTO.GONZALONAZARENO.ORG
...
[realms]
        ERNESTO.GONZALONAZARENO.ORG = {
                kdc = kerberos.ernesto.gonzalonazareno.org
                admin_server = kerberos.ernesto.gonzalonazareno.org
        }

...
[domain_realm]
        .ernesto.gonzalonazareno.org = ERNESTO.GONZALONAZARENO.ORG
        ernesto.gonzalonazareno.org = ERNESTO.GONZALONAZARENO.ORG
```

A continuación lo agregamos con el comando `krb5_newrealm`, donde nos pedirá la clave maestra de Kerberos.

```
debian@croqueta:~$ sudo krb5_newrealm

This script should be run on the master KDC/admin server to initialize
a Kerberos realm.  It will ask you to type in a master key password.
This password will be used to generate a key that is stored in
/etc/krb5kdc/stash.  You should try to remember this password, but it
is much more important that it be a strong password than that it be
remembered.  However, if you lose the password and /etc/krb5kdc/stash,
you cannot decrypt your Kerberos database.
Loading random data
Initializing database '/var/lib/krb5kdc/principal' for realm 'ERNESTO.GONZALONAZARENO.ORG',
master key name 'K/M@ERNESTO.GONZALONAZARENO.ORG'
You will be prompted for the database Master Password.
It is important that you NOT FORGET this password.
Enter KDC database master key: 
Re-enter KDC database master key to verify: 


Now that your realm is set up you may wish to create an administrative
principal using the addprinc subcommand of the kadmin.local program.
Then, this principal can be added to /etc/krb5kdc/kadm5.acl so that
you can use the kadmin program on other computers.  Kerberos admin
principals usually belong to a single user and end in /admin.  For
example, if jruser is a Kerberos administrator, then in addition to
the normal jruser principal, a jruser/admin principal should be
created.

Don't forget to set up DNS information so your clients can find your
KDC and admin servers.  Doing so is documented in the administration
guide.
```

Posteriormente reiniciamos los servicios:

```
debian@croqueta:~$ sudo systemctl restart krb5-kdc
debian@croqueta:~$ sudo systemctl restart krb5-admin-server
```

Podremos ver los principales que se generan automáticamente al instalar el servidor **kadmin**:

```
root@croqueta:~# kadmin.local
Authenticating as principal root/admin@ERNESTO.GONZALONAZARENO.ORG with password.
kadmin.local:  list_principals
K/M@ERNESTO.GONZALONAZARENO.ORG
kadmin/admin@ERNESTO.GONZALONAZARENO.ORG
kadmin/changepw@ERNESTO.GONZALONAZARENO.ORG
kadmin/croqueta.ernesto.gonzalonazareno.org@ERNESTO.GONZALONAZARENO.ORG
kiprop/croqueta.ernesto.gonzalonazareno.org@ERNESTO.GONZALONAZARENO.ORG
krbtgt/ERNESTO.GONZALONAZARENO.ORG@ERNESTO.GONZALONAZARENO.ORG
```

A continuación vamos a crear los principales para el usuario **pruebauser**

```
kadmin.local:  add_principal pruebauser
WARNING: no policy specified for pruebauser@ERNESTO.GONZALONAZARENO.ORG; defaulting to no policy
Enter password for principal "pruebauser@ERNESTO.GONZALONAZARENO.ORG": 
Re-enter password for principal "pruebauser@ERNESTO.GONZALONAZARENO.ORG": 
Principal "pruebauser@ERNESTO.GONZALONAZARENO.ORG" created.

kadmin.local:  add_principal -randkey host/croqueta.ernesto.gonzalonazareno.org
WARNING: no policy specified for host/croqueta.ernesto.gonzalonazareno.org@ERNESTO.GONZALONAZARENO.ORG; defaulting to no policy
Principal "host/croqueta.ernesto.gonzalonazareno.org@ERNESTO.GONZALONAZARENO.ORG" created.

kadmin.local:  add_principal -randkey host/tortilla.ernesto.gonzalonazareno.org
WARNING: no policy specified for host/tortilla.ernesto.gonzalonazareno.org@ERNESTO.GONZALONAZARENO.ORG; defaulting to no policy
Principal "host/tortilla.ernesto.gonzalonazareno.org@ERNESTO.GONZALONAZARENO.ORG" created.

kadmin.local:  add_principal -randkey ldap/croqueta.ernesto.gonzalonazareno.org
WARNING: no policy specified for ldap/croqueta.ernesto.gonzalonazareno.org@ERNESTO.GONZALONAZARENO.ORG; defaulting to no policy
Principal "ldap/croqueta.ernesto.gonzalonazareno.org@ERNESTO.GONZALONAZARENO.ORG" created.
```

## Ficheros keytab

En este apartado vamos a configurar los ficheros keytab para las claves cifradas puedan autenticarse contra el servidor Kerberos de forma no interactiva.

```
kadmin.local:  ktadd host/croqueta.ernesto.gonzalonazareno.org
Entry for principal host/croqueta.ernesto.gonzalonazareno.org with kvno 2, encryption type aes256-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.
Entry for principal host/croqueta.ernesto.gonzalonazareno.org with kvno 2, encryption type aes128-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.

kadmin.local:  ktadd ldap/croqueta.ernesto.gonzalonazareno.org
Entry for principal ldap/croqueta.ernesto.gonzalonazareno.org with kvno 2, encryption type aes256-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.
Entry for principal ldap/croqueta.ernesto.gonzalonazareno.org with kvno 2, encryption type aes128-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.
```

## Creación de usuarios para la administración de Kerberos

Para crear un rol de administración con todos los permisos hay que descomentar la siguiente línea.

```
debian@croqueta:~$ sudo nano /etc/krb5kdc/kadm5.acl

# */admin *
```

A continuación accedemos de nuevo a `kadmin.local`.

```
debian@croqueta:~$ sudo kadmin.local

Authenticating as principal root/admin@ERNESTO.GONZALONAZARENO.ORG with password.
kadmin.local:  add_principal ernestovazgar/admin
WARNING: no policy specified for ernestovazgar/admin@ERNESTO.GONZALONAZARENO.ORG; defaulting to no policy
Enter password for principal "ernestovazgar/admin@ERNESTO.GONZALONAZARENO.ORG": 
Re-enter password for principal "ernestovazgar/admin@ERNESTO.GONZALONAZARENO.ORG": 
Principal "ernestovazgar/admin@ERNESTO.GONZALONAZARENO.ORG" created.
```

De esta manera hemos crear un principal para un usuario administrador.

## Configuración del cliente Kerberos

A continuación vamos a instalar el cliente kerberos en tortilla.

```
ubuntu@tortilla:~$ sudo apt install krb5-config krb5-user
```

Editamos el fichero de configuración `/etc/krb5.conf,` al igual que en el servidor.

```
ubuntu@tortilla:~$ sudo nano /etc/krb5.conf 

[libdefaults]
        default_realm = ERNESTO.GONZALONAZARENO.ORG
...
[realms]
        ERNESTO.GONZALONAZARENO.ORG = {
                kdc = kerberos.ernesto.gonzalonazareno.org
                admin_server = kerberos.ernesto.gonzalonazareno.org
        }

...
[domain_realm]
        .ernesto.gonzalonazareno.org = ERNESTO.GONZALONAZARENO.ORG
        ernesto.gonzalonazareno.org = ERNESTO.GONZALONAZARENO.ORG
```

## klist y knit

Para ver los tickets de la sesión de usuario utilizarenos el comanto `klist -5` (Pero antes tenemos que autenticarnos), en caso de no estar autenticados saltaria la siguiente salida: 

`klist: No credentials cache found (ticket cache FILE:/tmp/krb5cc_0)`

Para autenticarnos utilizarenos `kinit` seguido del nombre de usuario.

```
ubuntu@tortilla:~$ klist -5
klist: No credentials cache found (filename: /tmp/krb5cc_1000

root@tortilla:/home/ubuntu# kinit pruebauser
Password for pruebauser@ERNESTO.GONZALONAZARENO.ORG: 

root@tortilla:/home/ubuntu# klist -5
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: pruebauser@ERNESTO.GONZALONAZARENO.ORG

Valid starting     Expires            Service principal
02/25/20 19:20:57  02/26/20 05:20:57  krbtgt/ERNESTO.GONZALONAZARENO.ORG@ERNESTO.GONZALONAZARENO.ORG
	renew until 02/26/20 19:20:53
```

## SASL/GSSAPI

Autentificación simple con LDAP.

LDAP nos permite autenticarnos utilizando Simple Authentication and Security Layer(SASL) y con kerberos mediante GSSAPI.

Para ello vamos a instalar los paquetes necesarios.

    debian@croqueta:~$ sudo apt install libsasl2-modules-gssapi-mit

Vamos a cambiar los permisos al siguiente fichero, para que ldap pueda acceder.

```
root@croqueta:~# chmod 640 /etc/krb5.keytab
root@croqueta:~# chgrp openldap /etc/krb5.keytab
```

A continuación vamos a añadir al fichero de configuración de slapd lo siguiente.

```
debian@croqueta:~$ sudo nano /etc/ldap/sasl2/slapd.conf

mech_list: GSSAPI
```

```
debian@croqueta:~$ sudo nano /etc/ldap/ldap.conf 

SASL_MECH GSSAPI
SASL_REALM ERNESTO.GONZALONAZARENO.ORG
SASL_NOCANON ON
```

Ahora vamos a reiniciar los servicios:

```
root@croqueta:~# systemctl restart slapd
```

Para comprobar si está activado **SASL/GSSAPI** realizamos la siguiente consulta.

```
root@croqueta:~# ldapsearch -x -b "" -s base -LLL supportedSASLMechanisms

dn:
supportedSASLMechanisms: GSSAPI
```

Ahora con el usuario **pruebauser** desde tortilla:

```
ubuntu@tortilla:~$ ldapsearch "gidNumber=2510"

ldap_sasl_interactive_bind_s: Unknown authentication method (-6)
	additional info: SASL(-4): no mechanism available: No worthy mechs found
```

Para que funcione vamos a instalar en tortilla el siguiente paquete.

```
root@tortilla:/etc/ldap# apt install libsasl2-modules-gssapi-mit
```

He tenido que abrir los siguientes puertos: 464 tcp, 464 udp, 88 udp y 749 tcp.

He tenido que realizar el **klist y kinit** en croqueta.

```
root@croqueta:~# kinit pruebauser
Password for pruebauser@ERNESTO.GONZALONAZARENO.ORG: 

root@croqueta:~# klist -5
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: pruebauser@ERNESTO.GONZALONAZARENO.ORG

Valid starting       Expires              Service principal
02/26/2020 12:55:54  02/26/2020 22:55:54  krbtgt/ERNESTO.GONZALONAZARENO.ORG@ERNESTO.GONZALONAZARENO.ORG
	renew until 02/27/2020 12:54:36
    
root@croqueta:~# ldapwhoami
SASL/GSSAPI authentication started
SASL username: pruebauser@ERNESTO.GONZALONAZARENO.ORG
SASL SSF: 256
SASL data security layer installed.
dn:uid=pruebauser,cn=gssapi,cn=auth
```

He tenido que añadir la información al fichero de configuración:

```
root@tortilla:/etc/ldap# nano ldap.conf 

BASE    dc=ernesto,dc=gonzalonazareno,dc=org
URI     ldap://ldap.ernesto.gonzalonazareno.org
...
SASL_MECH GSSAPI
SASL_REALM ERNESTO.GONZALONAZARENO.ORG
SASL_NOCANON ON
```

```
root@tortilla:~# sudo nano /etc/ldap/sasl2/slapd.conf

mech_list GSSAPI
```

Si necesitamos quitar los tickets podemos realizar un `kdestoy`, en mi caso voy a realizar **klist y kinit.**

```
root@tortilla:/etc/ldap# kdestroy 

root@tortilla:/etc/ldap# kinit pruebauser
Password for pruebauser@ERNESTO.GONZALONAZARENO.ORG: 

root@tortilla:~# klist -5
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: pruebauser@ERNESTO.GONZALONAZARENO.ORG

Valid starting     Expires            Service principal
02/26/20 12:56:42  02/26/20 22:56:42  krbtgt/ERNESTO.GONZALONAZARENO.ORG@ERNESTO.GONZALONAZARENO.ORG
	renew until 02/27/20 12:56:39
02/26/20 13:00:43  02/26/20 22:56:42  ldap/croqueta.ernesto.gonzalonazareno.org@ERNESTO.GONZALONAZARENO.ORG
	renew until 02/27/20 12:56:39

root@tortilla:/etc/ldap# ldapwhoami
SASL/GSSAPI authentication started
SASL username: pruebauser@ERNESTO.GONZALONAZARENO.ORG
SASL SSF: 56
SASL data security layer installed.
dn:uid=pruebauser,cn=gssapi,cn=auth
```

## PAM

Para que el sistema pueda usar kerberos es necesario instalar los siguientes paquetes.

```
root@croqueta:~# apt install libpam-krb5
root@tortilla:~# apt install libpam-krb5
```

Vamos a realizar una copia del fichero que vamos a modificar, por seguridad.

```
root@croqueta:~# cp -r /etc/pam.d /etc/pam.d.old
root@tortilla:~# cp -r /etc/pam.d /etc/pam.d.old
```

Vamos configurar los siguientes ficheros:

* **/etc/pam.d/common-auth**

```
auth    sufficient      pam_krb5.so minimum_uid=2000
auth    required        pam_unix.so try_first_pass nullok_secure
```

* **/etc/pam.d/common-session**

```
session optional        pam_krb5.so     minimum_uid=2000
session required        pam_unix.so
```

* **/etc/pam.d/common-account**

```
account	sufficient	pam_krb5.so	minimum_uid=2000
account	required	pam_unix.so
```

* **/etc/pam.d/common-password**

```
password  sufficient  pam_krb5.so minimum_uid=1000
password  required    pam_unix.so nullok obscure sha512
```

Estos pasos lo realizaremos también en tortilla.

Ahora provamos hacer login:

```
root@croqueta:~# login pruebauser
Password: 
Linux croqueta 4.19.0-6-cloud-amd64 #1 SMP Debian 4.19.67-2+deb10u2 (2019-11-11) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.

pruebauser@croqueta:~$ 
```

## Network File System 4 (NFS4)

Vamos a instalar los paquetes para el servidor:

```
debian@croqueta:~$ sudo apt install nfs-kernel-server
```

Vamos a editar el siguiente fichero de configuración para que pueda utilizar kerberos:

```
debian@croqueta:~$ sudo nano /etc/default/nfs-common 

# Do you want to start the idmapd daemon? It is only needed for NFSv4.
NEED_IDMAPD=yes

# Do you want to start the gssd daemon? It is required for Kerberos mount$
NEED_GSSD=yes
```

```
debian@croqueta:~$ sudo nano /etc/default/nfs-kernel-server

# Do you want to start the svcgssd daemon? It is only required for Kerberos
# exports. Valid alternatives are "yes" and "no"; the default is "no".
NEED_SVCGSSD="yes"
```

Descomentamos la siguiente línea y añadimos nuestro dominio.

```
debian@croqueta:~$ sudo nano /etc/idmapd.conf

# set your own domain here, if it differs from FQDN minus hostname
# Domain = localdomain

Domain = ernesto.gonzalonazareno.org
```

```
debian@croqueta:~$ sudo kadmin.local
Authenticating as principal pruebauser/admin@ERNESTO.GONZALONAZARENO.ORG with password.

kadmin.local:  add_principal -randkey nfs/croqueta.ernesto.gonzalonazanareno.org
WARNING: no policy specified for nfs/croqueta.ernesto.gonzalonazanareno.org@ERNESTO.GONZALONAZARENO.ORG; defaulting to no policy
Principal "nfs/croqueta.ernesto.gonzalonazanareno.org@ERNESTO.GONZALONAZARENO.ORG" created.

kadmin.local:  add_principal -randkey nfs/tortilla.ernesto.gonzalonazareno.org
WARNING: no policy specified for nfs/tortilla.ernesto.gonzalonazareno.org@ERNESTO.GONZALONAZARENO.ORG; defaulting to no policy
Principal "nfs/tortilla.ernesto.gonzalonazareno.org@ERNESTO.GONZALONAZARENO.ORG" created.

kadmin.local:  ktadd nfs/croqueta.ernesto.gonzalonazanareno.org
Entry for principal nfs/croqueta.ernesto.gonzalonazanareno.org with kvno 2, encryption type aes256-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.
Entry for principal nfs/croqueta.ernesto.gonzalonazanareno.org with kvno 2, encryption type aes128-cts-hmac-sha1-96 added to keytab FILE:/etc/krb5.keytab.

kadmin.local:  ktadd -k /tmp/krb5.keytab nfs/tortilla.ernesto.gonzalonazareno.org
Entry for principal nfs/tortilla.ernesto.gonzalonazareno.org with kvno 2, encryption type aes256-cts-hmac-sha1-96 added to keytab WRFILE:/tmp/krb5.keytab.
Entry for principal nfs/tortilla.ernesto.gonzalonazareno.org with kvno 2, encryption type aes128-cts-hmac-sha1-96 added to keytab WRFILE:/tmp/krb5.keytab.
```

A continuación vamos a tener copiar dicho fichero en tortilla

Para ello podemos utilizar `scp` y lo meteremos en el directorio `/etc/krb5.keytab`.

Ahora vamos a crear el directorio para guardar el home de los usuarios:

```
root@croqueta:~# mkdir -p /srv/nfs4/homes
root@croqueta:~# cd /srv/nfs4/homes/
root@croqueta:/srv/nfs4/homes# mount --bind /home /srv/nfs4/homes
```

Vamos editar el fichero de exportación y descomentamos las últimas líneas.

```
root@croqueta:/srv/nfs4/homes# nano /etc/exports 

# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
```

A continuación vamos reiniciar los servicios:

```
root@croqueta:~# systemctl restart nfs-kernel-server
root@croqueta:~# systemctl restart nfs-common
Failed to restart nfs-common.service: Unit nfs-common.service is masked.
```

Como se puede apreciar no se puede reiniciar el servicio de nfs-common, para ello vamos a realizar la siguiente instrucción.

```
root@croqueta:~# rm /lib/systemd/system/nfs-common.service 
root@croqueta:~# systemctl daemon-reload 
root@croqueta:~# systemctl restart nfs-common.service 
```

Ya podremos reiniciar los servicios, esto se debe a que la unidad está enmascarada.

```
root@croqueta:~# showmount -e
Export list for croqueta:
/srv/nfs4/homes gss/krb5i
/srv/nfs4       gss/krb5i
```

Con ese comando podremos ver si se han montado correctamente.

## Cliente NFS

En este apartado vamos a configurar el cliente NFS en tortilla

Para ello vamos a instalarlo:

```
ubuntu@tortilla:~$ sudo apt install nfs-common 
```

A continuación tendremos que configurarlo, igual que lo hemos realizado en croquete:

```
ubuntu@tortilla:~$ sudo nano /etc/default/nfs-common 

# Do you want to start the gssd daemon? It is required for Kerberos moun$
NEED_GSSD=yes
NEED_IDMAPD=yes
```

```
ubuntu@tortilla:~$ sudo nano /etc/idmapd.conf

Domain = ernesto.gonzalonazareno.org
```

Volvemos a reiniciar los servicios:

```
root@tortilla:~# rm /lib/systemd/system/nfs-common.service 
root@tortilla:~# systemctl daemon-reload
root@tortilla:~# systemctl restart nfs-common
```

Por último vamos a configurar el fichero `/etc/fstab`:

- **Croqueta**:

```
root@croqueta:~# nano /etc/fstab 

/home           /srv/nfs4/homes                         none            rw,bind         0       0
```

- **Tortilla**:

```
root@tortilla:~# nano /etc/fstab 

croqueta.ernesto.gonzalonazareno.org    /home/nfs4      nfs4    rw,sec=krb5i    0 0
```





