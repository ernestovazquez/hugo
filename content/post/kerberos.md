
---
title: "Creación de un sistema de cuentas de usuarios en GNU/Linux con LDAP, Kerberos 5 y NFS4"
date: 2020-02-10T15:04:52+01:00
draft: false
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

