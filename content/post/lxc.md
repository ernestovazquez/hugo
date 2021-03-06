---
title: "Vagrant con LXC"
date: 2020-02-21T08:50:52+01:00
draft: false
---

Explica los pasos necesarios para configurar Vagrant para que utilice LXC como proveedor y una vez configurado el sistema crea un Vagrantfile que lance dos contenedores sobre LXC en los que se instale una aplicación web en dos capas (contenedor1 con servidor web y contenedor2 con servidor de BBDD).

La configuración completa de la aplicación se realizará utilizando ansible como sistema de aprovisionamiento de Vagrant

***

Voy a utilizar una máquina vagrant para realizar la instalación.

## Instalación de Vagrant-LXC

Lo primero que tenemos que hacer es instalar **`vagrant-lxc`**.

Para instalar **vagrant-lxc** en debian ejecutaremos el siguiente comando:

`ernesto@honda:~$ sudo apt install vagrant-lxc`

Configuraciones y paquetes necesarios para la red.

```
ernesto@honda:~$ sudo nano /etc/lxc/default.conf 

lxc.net.0.type = veth
lxc.net.0.link = virbr0
lxc.net.0.flags = up
lxc.apparmor.profile = generated
lxc.apparmor.allow_nesting = 1
```

Paquetes necesarios

```
ernesto@honda:~$ sudo apt-get install -qy libvirt-clients libvirt-daemon-system iptables ebtables dnsmasq-base
ernesto@honda:~$ sudo virsh net-start default
```

## Vagrantfile

Agregamos lo siguiente al Vagranfile:

```
ernesto@honda:~/Documentos/vagrant/lxc$ nano Vagrantfile 

Vagrant.configure("2") do |config|
  config.vm.define "db" do |ubun|
  ubun.vm.box = "sagiru/buster-amd64"
  ubun.vm.network "private_network", ip: "192.168.122.100", lxc__bridge_name: 'virbr0'
  ubun.vm.provider :lxc do |lxc|
    lxc.container_name = "servidordb"
  end
 end

  config.vm.define "servidorweb" do |ubun|
  ubun.vm.box = "sagiru/buster-amd64"
  ubun.vm.network "private_network", ip: "192.168.122.101", lxc__bridge_name: 'virbr0'
  ubun.vm.provider :lxc do |lxc|
    lxc.container_name = "servidorweb"
  end
 end
end
```

## Ansible

Instalamos ansible en la máquina.

    ernesto@honda:~$ sudo apt install ansible

Creamos la receta: [**Receta ansible**](https://github.com/ernestovazquez/lxc-ansible)

Tendremos que cambiar los siguientes ficheros de configuración del mismo repositorio para que pueda tener conexión entre ellos:

```
ernesto@honda:~/GitHub/lxc-ansible$ nano hosts 

[servidores_web]
nodo1 ansible_ssh_host=192.168.122.101 ansible_ssh_private_key_file="../.vagrant/machines/servidorweb/lxc/private_key"

[db]
nodo2 ansible_ssh_host=192.168.122.100 ansible_ssh_private_key_file="../.vagrant/machines/db/lxc/private_key"
```

```
ernesto@honda:~/GitHub/lxc-ansible$ nano group_vars/all 

wordpress_bd: wordpress_bd
wordpress_user: userwp
wordpress_pass: userwp
wordpress_host: 192.168.122.100
mariadb_host: '%'
```

Clonamos la receta

```
ernesto@honda:~/Documentos/vagrant/lxc$ git clone git@github.com:ernestovazquez/lxc-ansible.git
Clonando en 'lxc-ansible'...
remote: Enumerating objects: 44, done.
remote: Counting objects: 100% (44/44), done.
remote: Compressing objects: 100% (27/27), done.
remote: Total 44 (delta 2), reused 43 (delta 1), pack-reused 0
Recibiendo objetos: 100% (44/44), 7.13 KiB | 3.57 MiB/s, listo.
Resolviendo deltas: 100% (2/2), listo.
ernesto@honda:~/Documentos/vagrant/lxc$ cd lxc-ansible/

ernesto@honda:~/Documentos/vagrant/lxc/lxc-ansible$ ls
ansible.cfg  group_vars  hosts  README.md  roles  site.yaml
```

Ahora solamente tendremos que ejecutar la instrucción de ansible para que lo lance.

```
ernesto@honda:~/Documentos/vagrant/lxc/lxc-ansible$ ansible-playbook -b site.yaml 

PLAY [all] *******************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [nodo2]
ok: [nodo1]

TASK [commons : Ensure system is updated] ************************************************************************************************************
 [WARNING]: Could not find aptitude. Using apt-get instead.

ok: [nodo2]
ok: [nodo1]

PLAY [servidores_web] ********************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [nodo1]

TASK [apache2 : install apache2+php] *****************************************************************************************************************
ok: [nodo1]

TASK [apache2 : Copy index.html] *********************************************************************************************************************
ok: [nodo1]

TASK [apache2 : Copy info.php] ***********************************************************************************************************************
ok: [nodo1]

PLAY [db] ********************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [nodo2]

TASK [mariadb : ensure mariadb is installed] *********************************************************************************************************
ok: [nodo2]

TASK [mariadb : create database wordpress] ***********************************************************************************************************
ok: [nodo2]

TASK [mariadb : create user mysql wordpress] *********************************************************************************************************
ok: [nodo2]

TASK [mariadb : ensure mariadb binds to internal interface] ******************************************************************************************
ok: [nodo2]

PLAY [servidores_web] ********************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [nodo1]

TASK [wordpress : install unzip] *********************************************************************************************************************
ok: [nodo1]

TASK [wordpress : download wordpress] ****************************************************************************************************************
ok: [nodo1]

TASK [wordpress : unzip wordpress] *******************************************************************************************************************
changed: [nodo1]

TASK [wordpress : copy wp-config.php] ****************************************************************************************************************
ok: [nodo1]

PLAY RECAP *******************************************************************************************************************************************
nodo1                      : ok=11   changed=1    unreachable=0    failed=0   
nodo2                      : ok=7    changed=0    unreachable=0    failed=0   
```

![](https://i.imgur.com/wk0s5Lu.png)

## Ansible en el Vagrantfile

Para que se inicie con el Vagrantfile solamente tendremos que añadir los siguiente al fichero de configuración:

```
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "lxc-ansible/site.yaml"
  end
```

Tambien he tenido que cambiar el nombre de la máquina a **servidores_web.**

Quedaria de la siguiente forma:

```
Vagrant.configure("2") do |config|
  config.vm.define "db" do |ubun|
  ubun.vm.box = "sagiru/buster-amd64"
  ubun.vm.network "private_network", ip: "192.168.122.100", lxc__bridge_name: 'virbr0'
  ubun.vm.provider :lxc do |lxc|
    lxc.container_name = "servidordb"
  end
 end
  config.vm.define "servidores_web" do |ubun|
  ubun.vm.box = "sagiru/buster-amd64"
  ubun.vm.network "private_network", ip: "192.168.122.101", lxc__bridge_name: 'virbr0'
  ubun.vm.provider :lxc do |lxc|
    lxc.container_name = "servidores_web"
  end
 end
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "lxc-ansible/site.yaml"
  end
end
```

Vamos a ver como realizaria el despligue:

```
ernesto@honda:~/Documentos/vagrant/lxc$ vagrant up
Bringing machine 'db' up with 'lxc' provider...
Bringing machine 'servidores_web' up with 'lxc' provider...
==> db: Importing base box 'sagiru/buster-amd64'...
==> servidores_web: Importing base box 'sagiru/buster-amd64'...
==> servidores_web: Checking if box 'sagiru/buster-amd64' version '0.2' is up to date...
==> db: Checking if box 'sagiru/buster-amd64' version '0.2' is up to date...
==> db: Fixed port collision for 22 => 2222. Now on port 2200.
==> servidores_web: Setting up mount entries for shared folders...
==> db: Setting up mount entries for shared folders...
    servidores_web: /vagrant => /home/ernesto/Documentos/vagrant/lxc
    db: /vagrant => /home/ernesto/Documentos/vagrant/lxc
==> servidores_web: Starting container...
==> db: Starting container...
==> servidores_web: Waiting for machine to boot. This may take a few minutes...
==> db: Waiting for machine to boot. This may take a few minutes...
    servidores_web: SSH address: 192.168.122.43:22
    servidores_web: SSH username: vagrant
    servidores_web: SSH auth method: private key
    db: SSH address: 192.168.122.123:22
    db: SSH username: vagrant
    db: SSH auth method: private key
    servidores_web: 
    servidores_web: Vagrant insecure key detected. Vagrant will automatically replace
    servidores_web: this with a newly generated keypair for better security.
    db: 
    db: Vagrant insecure key detected. Vagrant will automatically replace
    db: this with a newly generated keypair for better security.
    db: 
    db: Inserting generated public key within guest...
    servidores_web: 
    servidores_web: Inserting generated public key within guest...
    db: Removing insecure key from the guest if it's present...
    servidores_web: Removing insecure key from the guest if it's present...
    db: Key inserted! Disconnecting and reconnecting using new SSH key...
    servidores_web: Key inserted! Disconnecting and reconnecting using new SSH key...
==> servidores_web: Machine booted and ready!
==> servidores_web: Setting up private networks...
==> db: Machine booted and ready!
==> db: Setting up private networks...
==> servidores_web: Running provisioner: ansible...
==> db: Running provisioner: ansible...
Vagrant has automatically selected the compatibility mode '2.0'
according to the Ansible version installed (2.7.7).

Alternatively, the compatibility mode can be specified in your Vagrantfile:
https://www.vagrantup.com/docs/provisioning/ansible_common.html#compatibility_mode

Vagrant has automatically selected the compatibility mode '2.0'
according to the Ansible version installed (2.7.7).

Alternatively, the compatibility mode can be specified in your Vagrantfile:
https://www.vagrantup.com/docs/provisioning/ansible_common.html#compatibility_mode

    servidores_web: Running ansible-playbook...
    db: Running ansible-playbook...

PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************

PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [servidores_web]

TASK [commons : Ensure system is updated] **************************************
ok: [db]

TASK [commons : Ensure system is updated] **************************************
 [WARNING]: Could not find aptitude. Using apt-get instead.

changed: [servidores_web]

PLAY [servidores_web] **********************************************************

TASK [Gathering Facts] *********************************************************
 [WARNING]: Could not find aptitude. Using apt-get instead.

changed: [db]

PLAY [servidores_web] **********************************************************
skipping: no hosts matched

PLAY [db] **********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [servidores_web]

TASK [apache2 : install apache2+php] *******************************************
ok: [db]

TASK [mariadb : ensure mariadb is installed] ***********************************
changed: [db]

TASK [mariadb : create database wordpress] *************************************
changed: [db]

TASK [mariadb : create user mysql wordpress] ***********************************
changed: [db]

TASK [mariadb : ensure mariadb binds to internal interface] ********************
changed: [db]

RUNNING HANDLER [mariadb : restart mariadb] ************************************
changed: [servidores_web]

TASK [apache2 : Copy index.html] ***********************************************
changed: [servidores_web]

TASK [apache2 : Copy info.php] *************************************************
changed: [servidores_web]

RUNNING HANDLER [apache2 : restart apache2] ************************************
changed: [servidores_web]

PLAY [db] **********************************************************************
skipping: no hosts matched

PLAY [servidores_web] **********************************************************

TASK [Gathering Facts] *********************************************************
ok: [servidores_web]

TASK [wordpress : install unzip] ***********************************************
changed: [db]

PLAY [servidores_web] **********************************************************
skipping: no hosts matched

PLAY RECAP *********************************************************************
db                         : ok=8    changed=6    unreachable=0    failed=0   

changed: [servidores_web]

TASK [wordpress : download wordpress] ******************************************
changed: [servidores_web]

TASK [wordpress : unzip wordpress] *********************************************
changed: [servidores_web]

TASK [wordpress : copy wp-config.php] ******************************************
changed: [servidores_web]

RUNNING HANDLER [wordpress : restart apache2] **********************************
changed: [servidores_web]

PLAY RECAP *********************************************************************
servidores_web             : ok=13   changed=10   unreachable=0    failed=0   
```

![](https://i.imgur.com/9Nyj2su.png)

