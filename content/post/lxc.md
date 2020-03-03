---
title: "Vagrant con LXC"
date: 2020-02-21T08:50:52+01:00
draft: false
---

Explica los pasos necesarios para configurar Vagrant para que utilice LXC como proveedor y una vez configurado el sistema crea un Vagrantfile que lance dos contenedores sobre LXC en los que se instale una aplicación web en dos capas (contenedor1 con servidor web y contenedor2 con servidor de BBDD).

La configuración completa de la aplicación se realizará utilizando ansible como sistema de aprovisionamiento de Vagrant

***

Voy a realizar una nueva instancia en OpenStack para realizar la instalación.

Lo primero que tenemos que hacer es instalar **`vagrant-lxc`**.

Para instalar vagrant-lxc en Debian ejecutaremos lel siguiente comando:

`sudo apt install vagrant-lxc`

Configuraciones y paquetes necesarios para la red.

```
debian@lxc:~$ sudo nano /etc/lxc/default.conf 

lxc.net.0.type = veth
lxc.net.0.link = virbr0
lxc.net.0.flags = up
lxc.apparmor.profile = generated
lxc.apparmor.allow_nesting = 1
```

```
debian@lxc:~$ sudo apt-get install -qy libvirt-clients libvirt-daemon-system iptables ebtables dnsmasq-base
debian@lxc:~$ sudo virsh net-start default
```

Agregamos lo siguiente al Vagranfile:

```
Vagrant.configure("2") do |config|
    config.vm.box = "isc/lxc-ubuntu-19.10"
    config.vm.box_version = "1"
    config.vm.provider :lxc do |lxc|
    lxc.customize 'cgroup.memory.limit_in_bytes', '1024M'
  end
end
```

Ya podremos levantar la máquina:

    debian@lxc:~$ sudo vagrant up

Entraremos a la máquina con vagrant ssh:

```
debian@lxc:~$ sudo vagrant ssh
Welcome to Ubuntu 19.10 (GNU/Linux 4.19.0-6-cloud-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

vagrant@ubuntu-19:~$ 
```



