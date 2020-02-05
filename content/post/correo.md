---
title: "Servidor de correos en los servidores cloud"
date: 2020-02-05T08:50:52+01:00
draft: false
---

Esta tarea consiste en instalar y configurar un servidor de correo similar al de cualquier organización, capaz de enviar y recibir directamente correo, almacenar los usuarios en LDAP, filtrar el correo en busca de virus o spam y servirlo a sus usuarios a través de los protocolos POP, IMAP y configurar un Webmail.

- El servidor de correos se va a instalar en croqueta (debian)
- Los servidores que necesites para realizar la práctica serán los del cloud: salmorejo (servidor web) y croqueta (servidor dns y ldap).

***

## Tarea 1: Documenta en redmine una prueba de funcionamiento, donde envíes desde tu servidor local al exterior. Muestra el log donde se vea el envío. Muestra el correo que has recibido.

Primero vamos a necesitar instalar los paquetes necesarios para el uso del servidor de correo.

```
debian@croqueta:~$ sudo apt install postfix
debian@croqueta:~$ sudo apt install mailutils
```

A partir de aquí ya podemos configurar postfix para su funcionamiento.

Tendremos que abrir el **puerto 25** en el cloud.

![](https://i.imgur.com/jXvII3O.png)


Colocamos el **relayhost** en el siguiente fichero de configuración:

```
debian@croqueta:~$ sudo nano /etc/postfix/main.cf

relayhost = babuino-smtp.gonzalonazareno.org
```

Vamos a enviar un correo:

```
debian@croqueta:~$ mail vazquezgarciaernesto@gmail.com
Cc: 
Subject: Prueba Practica Correo
Esto es una prueba para la practica de servicios

```

Para ver el log del envio pondremos lo siguiente:

```
debian@croqueta:~$ sudo tail -f /var/log/mail.log 

Feb  5 09:06:32 croqueta postfix/pickup[31377]: B3D2A20ECC: uid=1000 from=<debian@croqueta.ernesto.gonzalonazareno.org>
Feb  5 09:06:32 croqueta postfix/cleanup[31440]: B3D2A20ECC: message-id=<20200205080632.B3D2A20ECC@croqueta.ernesto.gonzalonazareno.org>
Feb  5 09:06:32 croqueta postfix/qmgr[30358]: B3D2A20ECC: from=<debian@croqueta.ernesto.gonzalonazareno.org>, size=476, nrcpt=1 (queue active)
Feb  5 09:06:32 croqueta postfix/smtp[31442]: B3D2A20ECC: to=<vazquezgarciaernesto@gmail.com>, relay=babuino-smtp.gonzalonazareno.org[192.168.203.3]:25, delay=0.22, delays=0.15/0.01/0.04/0.02, dsn=2.0.0, status=sent (250 2.0.0 Ok: queued as D4094C9D82)
Feb  5 09:06:32 croqueta postfix/qmgr[30358]: B3D2A20ECC: removed
```

![](https://i.imgur.com/UnoBeli.png)

Ahora nos comprobamos si hemos recibido el correo.

![](https://i.imgur.com/QUOpJzj.png)

## Tarea 2: Documenta en redmine una prueba de funcionamiento, donde envíes un correo desde el exterior (gmail, hotmail,…) a tu servidor local. Muestra el log donde se vea el envío. Muestra cómo has leído el correo.

Vamos a responder al correo que hemos recibido para que se envie desde el exterior a nuestro servidor para posteriormente poder leer el correo.

![](https://i.imgur.com/aq6uZTP.png)

Para leer la respuesta solamente tendremos que poner lo siguiente:

```
debian@croqueta:~$ mail
"/var/mail/debian": 1 message 1 new
>N   1 Ernesto Vázquez G Wed Feb  5 09:15  87/4902  Re: Prueba Practica
? 1
```

Vamos a ver el log una vez se ha enviado la respuesta.

```
debian@croqueta:~$ sudo tail -f /var/log/mail.log

Feb  5 09:13:25 croqueta postfix/smtpd[31455]: connect from babuino-smtp.gonzalonazareno.org[192.168.203.3]
Feb  5 09:13:25 croqueta postfix/smtpd[31455]: 127BC20ECA: client=babuino-smtp.gonzalonazareno.org[192.168.203.3]
Feb  5 09:13:25 croqueta postfix/cleanup[31463]: 127BC20ECA: message-id=<CAEXXkz_Uw+jM2svcCkSmZhp1a+uujhJ46Mf7Ou=QL1qryOpX_A@mail.gmail.com>
Feb  5 09:13:25 croqueta postfix/smtpd[31455]: disconnect from babuino-smtp.gonzalonazareno.org[192.168.203.3] ehlo=1 mail=1 rcpt=1 data=1 quit=1 commands=5
Feb  5 09:13:25 croqueta postfix/qmgr[30358]: 127BC20ECA: from=<vazquezgarciaernesto@gmail.com>, size=4005, nrcpt=1 (queue active)
Feb  5 09:13:25 croqueta postfix/local[31464]: 127BC20ECA: to=<debian@croqueta.ernesto.gonzalonazareno.org>, relay=local, delay=0.22, delays=0.18/0.01/0/0.03, dsn=2.0.0, status=sent (delivered to mailbox)
Feb  5 09:13:25 croqueta postfix/qmgr[30358]: 127BC20ECA: removed
```

## Tarea 3: Documenta en redmine una prueba de funcionamiento, donde envíes desde tu cliente de correos al exterior. ¿Cómo se llama el servidor para enviar el correo? (Muestra la configuración).

Debemos habilitar el envío de correo desde cliente de nuestra red. Para ello vamos a añadir nuestra red local en la **directiva mynetworks**:

```
debian@croqueta:~$ sudo nano /etc/postfix/main.cf

mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128 172.22.0.0/16
```

Una vex configurado la red local, es hora de usar un cliente de correos en mi caso voy a utilizar **evolution.**

Tendremos que indicarle el nombre de nuestro servidor de correos.

Vamos a ver la **configuración**:

![](https://i.imgur.com/oW7mqCs.png)

![](https://i.imgur.com/j8PhUhe.png)

![](https://i.imgur.com/ockMFz5.png)

![](https://i.imgur.com/tTOZsZ5.png)

![](https://i.imgur.com/2HTaSyY.png)

