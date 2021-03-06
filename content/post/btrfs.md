---
title: "Sistemas de ficheros avanzados Btrfs"
date: 2020-01-23T21:20:52+01:00
draft: false
---

Elige uno de los dos sistemas de ficheros "avanzados" que hemos visto en clase.

 - Crea un escenario que incluya una máquina y varios discos asociados a ella.
 - Instala si es necesario el software de ZFS/Btrfs
 - Gestiona los discos adicionales con ZFS/Btrfs.
 - Configura los discos en RAID, haciendo pruebas de fallo de algún disco y sustitución, restauración del RAID. Comenta ventajas e inconvenientes respecto al uso de RAID software con mdadm
 - Realiza ejercicios con pruebas de funcionamiento de las principales funcionalidades: compresión, cow, deduplicación, cifrado, etc.

***

Vamos a crear una máquina Debian con tres discos asociados a ella.

Los discos serán de 1GB, 2GB y de 3GB.

Vagranfile:

```
Vagrant.configure("2") do |config|
disco1='.vagrant/disco01.vdi'
disco2='.vagrant/disco02.vdi'
disco3='.vagrant/disco03.vdi'
 config.vm.define :btrfs do |btrfs|
  btrfs.vm.box = "debian/buster64"
  btrfs.vm.hostname = "btrfs"
  btrfs.vm.provider :virtualbox do |v|
  v.customize ["createhd", "--filename", disco1, "--size", 1024]
  v.customize ["storageattach", :id, "--storagectl", "SATA Controller",
  "--port", 1, "--device", 0, "--type", "hdd",
  "--medium", disco1]
  v.customize ["createhd", "--filename", disco2, "--size", 2048]
  v.customize ["storageattach", :id, "--storagectl", "SATA Controller",
  "--port", 2, "--device", 0, "--type", "hdd",
  "--medium", disco2]
  v.customize ["createhd", "--filename", disco3, "--size", 3072]
  v.customize ["storageattach", :id, "--storagectl", "SATA Controller",
  "--port", 3, "--device", 0, "--type", "hdd",
  "--medium", disco3]
  end
end
end
```

#### Gestión de discos adicionales

Primero tenemos que instalar btrfs:

```
root@btrfs:~# apt install btrfs-tools
```

Vamos a formatear los discos con Btrfs:

```
root@btrfs:~# mkfs.btrfs /dev/sdc 
btrfs-progs v4.20.1 
See http://btrfs.wiki.kernel.org for more information.

Label:              (null)
UUID:               6dcf0480-3dbd-4497-b4b8-658ca3509c99
Node size:          16384
Sector size:        4096
Filesystem size:    2.00GiB
Block group profiles:
  Data:             single            8.00MiB
  Metadata:         DUP             102.38MiB
  System:           DUP               8.00MiB
SSD detected:       no
Incompat features:  extref, skinny-metadata
Number of devices:  1
Devices:
   ID        SIZE  PATH
    1     2.00GiB  /dev/sdc
```

Para montar un disco haremos lo siguiente:

```
root@btrfs:~# mount /dev/sdb /mnt
```

Para ver la salida del disco que hemos montado pondremos lo siguiente:
Donde pondremos la ruta de montaje del disco.

```
root@btrfs:~# btrfs fi usage /mnt
Overall:
    Device size:		   1.00GiB
    Device allocated:		 126.38MiB
    Device unallocated:		 897.62MiB
    Device missing:		     0.00B
    Used:			 256.00KiB
    Free (estimated):		 905.62MiB	(min: 456.81MiB)
    Data ratio:			      1.00
    Metadata ratio:		      2.00
    Global reserve:		  16.00MiB	(used: 0.00B)

Data,single: Size:8.00MiB, Used:0.00B
   /dev/sdb	   8.00MiB

Metadata,DUP: Size:51.19MiB, Used:112.00KiB
   /dev/sdb	 102.38MiB

System,DUP: Size:8.00MiB, Used:16.00KiB
   /dev/sdb	  16.00MiB

Unallocated:
   /dev/sdb	 897.62MiB
```

Vamos añadir un nuevo disco a dicha ruta de montaje (/mnt):
La opción **-f** es necesaria ya que dicho disco ya tiene **Btrfs**.
Ahora aumentará el tamaño disponible de la ruta **/mnt**

```
root@btrfs:~# btrfs device add -f /dev/sdc /mnt
```

Vemos la salida de la ruta de montaje:

```
root@btrfs:~# btrfs fi usage /mnt
Overall:
    Device size:		   3.00GiB
    Device allocated:		 126.38MiB
    Device unallocated:		   2.88GiB
    Device missing:		     0.00B
    Used:			 256.00KiB
    Free (estimated):		   2.88GiB	(min: 1.45GiB)
    Data ratio:			      1.00
    Metadata ratio:		      2.00
    Global reserve:		  16.00MiB	(used: 0.00B)

Data,single: Size:8.00MiB, Used:0.00B
   /dev/sdb	   8.00MiB

Metadata,DUP: Size:51.19MiB, Used:112.00KiB
   /dev/sdb	 102.38MiB

System,DUP: Size:8.00MiB, Used:16.00KiB
   /dev/sdb	  16.00MiB

Unallocated:
   /dev/sdb	 897.62MiB
   /dev/sdc	   2.00GiB
```

Vamos a crear un fichero de pruebas de 2GB, dentro de **/mnt**

```
root@btrfs:~# fallocate -l 2G /mnt/prueba
```

Ahora ya estaría el disco **/dev/sdb** con un tamaño de **3GB**. 
**/dev/sdb** y **/dev/sdc** están en la misma ruta.

Se puede ver con el siguiente comando:

```
root@btrfs:~# df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            228M     0  228M   0% /dev
tmpfs            49M  3.3M   46M   7% /run
/dev/sda1        19G  1.1G   17G   6% /
tmpfs           242M     0  242M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           242M     0  242M   0% /sys/fs/cgroup
tmpfs            49M     0   49M   0% /run/user/1000
/dev/sdb        3.0G  2.1G  904M  70% /mnt
```

Si queremos quitar alguno de estos, de dicha ruta en este caso, /mnt solamente tendremos que ejecutar el siguiente comando 

```
root@btrfs:~# btrfs device del /dev/sdc /mnt
```


#### RAID

Crear **RAID 1**:

```
root@btrfs:~# btrfs balance start -v -mconvert=raid1 -dconvert=raid1 /mnt
Dumping filters: flags 0x7, state 0x0, force is off
  DATA (flags 0x100): converting, target=16, soft is off
  METADATA (flags 0x100): converting, target=16, soft is off
  SYSTEM (flags 0x100): converting, target=16, soft is off
Done, had to relocate 3 out of 3 chunks
```

El siguiente comando nos muestra la salida para ver si disponemos de un RAID 

```
root@btrfs:~# btrfs fi df /mnt

Data, RAID1: total=320.00MiB, used=128.00KiB
System, RAID1: total=32.00MiB, used=16.00KiB
Metadata, RAID1: total=256.00MiB, used=112.00KiB
GlobalReserve, single: total=16.00MiB, used=0.00B
```

```
root@btrfs:~# btrfs fi usage /mnt

Overall:
    Device size:		   3.00GiB
    Device allocated:		   1.19GiB
    Device unallocated:		   1.81GiB
    Device missing:		     0.00B
    Used:			 512.00KiB
    Free (estimated):		   1.22GiB	(min: 1.22GiB)
    Data ratio:			      2.00
    Metadata ratio:		      2.00
    Global reserve:		  16.00MiB	(used: 0.00B)

Data,RAID1: Size:320.00MiB, Used:128.00KiB
   /dev/sdb	 320.00MiB
   /dev/sdc	 320.00MiB

Metadata,RAID1: Size:256.00MiB, Used:112.00KiB
   /dev/sdb	 256.00MiB
   /dev/sdc	 256.00MiB

System,RAID1: Size:32.00MiB, Used:16.00KiB
   /dev/sdb	  32.00MiB
   /dev/sdc	  32.00MiB

Unallocated:
   /dev/sdb	 416.00MiB
   /dev/sdc	   1.41GiB
```

En el caso de querer otro disco en el RAID pondremos lo siguiente:

```
root@btrfs:~# btrfs device add -f /dev/sdd /mnt
```

En el caso de querer volver al normal pondremos lo siguiente:

```
root@btrfs:~# btrfs balance start -dconvert=single -mconvert=raid1 /mnt
```

#### Restauración del RAID

Vamos a ver la manera de restaurar el RAID 1 que hemos montado. En el caso de que falle algun disco lo pondremos en **modo degraded**:

```
root@btrfs:~# mount -o degraded /dev/sdb /mnt
```

A continuación, y una vez que tengamos el siguiente disco listo pondremos el siguiente comando para sustituir el anterior:

Aquí podremos configurar la información que va a obtener del anterior disco, en el caso de no querer pondremos la opción (-r).

```
root@btrfs:~# btrfs replace start /dev/sdb /dev/sdd /mnt
```

#### Ventajas e inconvenientes respecto al uso de RAID software con mdadm

La principal diferencia es puede crear un **RAID 1** con **Btrfs** con varios discos de diferentes tamaños, incluso con la posibilidad de ampliar discos, mientras que con **mdadm** es necesario el mismo tamaño en los dos discos.

Una ventaja de RAID con mdadm es que se entiende bien, es estable y tiene un mejor rendimiento 

Btrfs tiene una mejor seguridad de los datos.

En conclusión el uso de RAID con Btrfs tiene muchas características útiles que mdadm no tiene y es una mejor opción.

#### Compresión

Ya que por defecto no usa ningun tipo de compresión, tendremos que indicarle el deseado.

Hay diferentes métodos de compresión: **ZLIB, LZO, ZSTD...**

Pruebas:

- ZLIB 

Este método de compresión es capaz de comprimir un gran fichero en poco tamaño.

Montamos con:

```
root@btrfs:~# mount -o compress=zlib /dev/sdb /mnt
```

Creamos y comprobamos el fichero con:

```
root@btrfs:~# dd if=/dev/zero of=/mnt/prueba bs=2048
10838301+0 records in
10838300+0 records out
22196838400 bytes (22 GB, 21 GiB) copied, 102.862 s, 216 MB/s

root@btrfs:~# ls -hal /mnt/prueba 
-rw-r--r-- 1 root root 21G Jan 23 09:17 /mnt/prueba

root@btrfs:~# btrfs fi usage /mnt
Overall:
    Device size:		   1.00GiB
    Device allocated:		1023.00MiB
    Device unallocated:		   1.00MiB
    Device missing:		     0.00B
    Used:			 750.12MiB
    Free (estimated):		     0.00B	(min: 0.00B)
    Data ratio:			      1.00
    Metadata ratio:		      2.00
    Global reserve:		  16.00MiB	(used: 0.00B)

Data,single: Size:692.00MiB, Used:692.00MiB
   /dev/sdb	 692.00MiB

Metadata,DUP: Size:157.50MiB, Used:29.05MiB
   /dev/sdb	 315.00MiB

System,DUP: Size:8.00MiB, Used:16.00KiB
   /dev/sdb	  16.00MiB

Unallocated:
   /dev/sdb	   1.00MiB
```

Vemos que es capaz de comprimir un fichero de **21G** en **750.12MiB**.

- LZO

Montamos con:

```
root@btrfs:~# mount -o compress=lzo /dev/sdb /mnt
```

Creamos y comprobamos el fichero con:

```
root@btrfs:~# dd if=/dev/zero of=/mnt/prueba bs=2048
dd: error writing '/mnt/prueba': No space left on device
9367395+0 records in
9367394+0 records out
19184422912 bytes (19 GB, 18 GiB) copied, 69.657 s, 275 MB/s

root@btrfs:~# ls -hal /mnt/prueba 
-rw-r--r-- 1 root root 18G Jan 23 09:30 /mnt/prueba

root@btrfs:~# btrfs fi usage /mnt
Overall:
    Device size:		   1.00GiB
    Device allocated:		1023.00MiB
    Device unallocated:		   1.00MiB
    Device missing:		     0.00B
    Used:			 751.06MiB
    Free (estimated):		     0.00B	(min: 0.00B)
    Data ratio:			      1.00
    Metadata ratio:		      2.00
    Global reserve:		  16.08MiB	(used: 0.00B)

Data,single: Size:692.00MiB, Used:692.00MiB
   /dev/sdb	 692.00MiB

Metadata,DUP: Size:157.50MiB, Used:29.52MiB
   /dev/sdb	 315.00MiB

System,DUP: Size:8.00MiB, Used:16.00KiB
   /dev/sdb	  16.00MiB

Unallocated:
   /dev/sdb	   1.00MiB
```

Este método de compresión es menos eficiente que el anterior, **ZLIB**. Pero sigue dando buenos resultados.

#### COW

Este sistema de ficheros como dice en el nombre, **Copy-on-write (COW)**, creará una copia del fichero para modificarlo y guardarlo.

Vamos a crear un fichero de **200 MiB:**

```
root@btrfs:~# dd if=/dev/zero of=/mnt/prueba bs=2048 count=100k
102400+0 records in
102400+0 records out
209715200 bytes (210 MB, 200 MiB) copied, 0.533322 s, 393 MB/s

root@btrfs:~# btrfs fi usage /mnt
Overall:
    Device size:		   1.00GiB
    Device allocated:		1023.00MiB
    Device unallocated:		   1.00MiB
    Device missing:		     0.00B
    Used:			 832.00KiB
    Free (estimated):		 691.44MiB	(min: 691.44MiB)
    Data ratio:			      1.00
    Metadata ratio:		      2.00
    Global reserve:		  16.00MiB	(used: 0.00B)
```

Tiene un tamaño de **691.44MiB**

Copia:
Vamos a ponerle la opción **reflink** para que permita controlar dichas copias

```
root@btrfs:~# cp --reflink=always /mnt/prueba /mnt/pruebacow

root@btrfs:~# btrfs fi usage /mnt
Overall:
    Device size:		   1.00GiB
    Device allocated:		 339.00MiB
    Device unallocated:		 685.00MiB
    Device missing:		     0.00B
    Used:			 876.00KiB
    Free (estimated):		 692.39MiB	(min: 349.89MiB)
    Data ratio:			      1.00
    Metadata ratio:		      2.00
    Global reserve:		  16.00MiB	(used: 0.00B)
```

Una vez realizada dicha copia, podemos ver el cambio en las copias y que estas no ocupan el espacio en el disco, ya que no se ha modificado.

#### Deduplicación

Identifica cuándo los datos se han escrito dos veces y combinándolos en una misma extensión del disco, con el objetivo de optimizar al máximo el espacio de almacenamiento utilizado, eliminando copias duplicadas o repetidas de datos.


#### Balance

Se puede hacer que el peso del fichero no quede en un solo disco, de esta manera el peso del fichero quedaria repartido entre los discos del directorio montado.

```
root@btrfs:~# btrfs fi balance --full-balance /mnt
Done, had to relocate 4 out of 4 chunks
```

#### Subvolumen

Vamos a ver el uso de los subvolúmenes

Con el siguiente comando podemos ver como se crean dichos subvolúmenes:
También vamos a crear un fichero llamado **pruebasub**. 

```
root@btrfs:/mnt# btrfs subvolume create subvolumen
Create subvolume './subvolumen'

root@btrfs:/mnt# touch subvolumen/pruebasub
```

El subvolumen es un directorio con más funcionalidades que un directorio común.

Podremos cambiar la ruta de montaje en la que está dicho subvolumen.

Primero necesitaremos el identificador y posteriormente ya podremos cambiarlo.

```
root@btrfs:/mnt# btrfs subvolume list /mnt
ID 265 gen 182 top level 5 path subvolumen
```

Con este ID, ya podemos ejecutar el cambio:

```
root@btrfs:~# mount -o subvolid=265 /dev/sdb /subvol

root@btrfs:~# cd /subvol/
root@btrfs:/subvol# ls -l
total 0
-rw-r--r-- 1 root root 0 Jan 23 09:49 pruebasub
```

Podremos borrar el subvolumen con:

```
root@btrfs:~# btrfs subvolume delete /mnt/subvolumen/
```

#### Snapshots

Vamos a realizar una snapshot, una copia instantanea del estado del sistema.

Voy a crear un escenario de 2 subvolúmenes, este primero con un par de ficheros con un tamaño especifico. para el ejemplo.

Creación de los subvolúmenes:

```
root@btrfs:/mnt# btrfs subvolume create subvolumen1
Create subvolume './subvolumen1'

root@btrfs:/mnt# btrfs subvolume create subvolumen2
Create subvolume './subvolumen2'
```

Creamos los dos ficheros del mismo tamaño cada uno:

```
root@btrfs:/mnt/subvolumen1# dd if=/dev/zero of=archivo1.txt bs=1024 count=209715
209715+0 records in
209715+0 records out
214748160 bytes (215 MB, 205 MiB) copied, 1.058 s, 203 MB/s

205M -rw-r--r-- 1 root root 205M Jan 23 19:14 archivo2.txt
root@btrfs:/mnt/subvolumen1# dd if=/dev/zero of=archivo2.txt bs=1024 count=209715
209715+0 records in
209715+0 records out
214748160 bytes (215 MB, 205 MiB) copied, 0.89513 s, 240 MB/s

root@btrfs:/mnt/subvolumen1# ls -lsh
total 410M
205M -rw-r--r-- 1 root root 205M Jan 23 19:14 archivo1.txt
205M -rw-r--r-- 1 root root 205M Jan 23 19:14 archivo2.txt
```

```
root@btrfs:/mnt/subvolumen1# btrfs fi usage /mnt
Overall:
    Device size:		   2.00GiB
    Device allocated:		 644.75MiB
    Device unallocated:		   1.37GiB
    Device missing:		     0.00B
    Used:			 410.97MiB
    Free (estimated):		   1.38GiB	(min: 715.81MiB)
    Data ratio:			      1.00
    Metadata ratio:		      2.00
    Global reserve:		  16.00MiB	(used: 0.00B)
```

Vemos que tiene **410.97MiB**

Vamos a realizar un **snapshot** del subvolumen:

```
root@btrfs:/mnt# btrfs subvolume snapshot /mnt/subvolumen1 /mnt/snapshotprueba
Create a snapshot of '/mnt/subvolumen1' in '/mnt/snapshotprueba'
```

Una vez hemos creado la snapshot vemos su contenido, tiene que ser el mismo que teniamos en el subvolumen ya que este es una copia suya, pero esta snapshot no ocupa espacio en el disco.

```
root@btrfs:/mnt# ls
snapshotprueba	subvolumen1  subvolumen2

root@btrfs:/mnt# cd snapshotprueba/

root@btrfs:/mnt/snapshotprueba# ls -lsh
total 410M
205M -rw-r--r-- 1 root root 205M Jan 23 19:14 archivo1.txt
205M -rw-r--r-- 1 root root 205M Jan 23 19:14 archivo2.txt
   0 drwxr-xr-x 1 root root    0 Jan 23 19:16 directorioprueba
```

```
root@btrfs:/mnt# btrfs fi usage /mnt
Overall:
    Device size:		   2.00GiB
    Device allocated:		 644.75MiB
    Device unallocated:		   1.37GiB
    Device missing:		     0.00B
    Used:			 411.00MiB
    Free (estimated):		   1.38GiB	(min: 715.81MiB)
    Data ratio:			      1.00
    Metadata ratio:		      2.00
    Global reserve:		  16.00MiB	(used: 0.00B)
```

Como hemos comentado antes, esta snapshot no ocupa sitio (tenemos practicamente el mismo espacio **411.00MiB**), esto se debe a que la copia ocupará su espacio una vez que los ficheros se modifiquen.

