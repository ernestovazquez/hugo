
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

A continuación tenemos que editar el fichero debian@croqueta:~$ sudo nano `/etc/ldap/ldap.conf.`

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
debian@croqueta:~$ chgrp openldap /etc/krb5.keytab 
debian@croqueta:~$ sudo chgrp openldap /etc/krb5.keytab 
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

