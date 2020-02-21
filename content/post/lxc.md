---
title: "Vagrant con LXC"
date: 2020-02-21T08:50:52+01:00
draft: false
---

Explica los pasos necesarios para configurar Vagrant para que utilice LXC como proveedor y una vez configurado el sistema crea un Vagrantfile que lance dos contenedores sobre LXC en los que se instale una aplicación web en dos capas (contenedor1 con servidor web y contenedor2 con servidor de BBDD).

La configuración completa de la aplicación se realizará utilizando ansible como sistema de aprovisionamiento de Vagrant

***

Lo primero que tenemos que hacer es instalar **`vagrant-lxc`**.

Para instalar vagrant-lxc en Debian ejecutaremos lel siguiente comando:

`ernesto@honda:~/Documentos/vagrant/lxc$ sudo apt install vagrant-lxc`

A continuación vamos a crear un **Vagrantfile** que lance dos contenedores sobre **LXC**.


