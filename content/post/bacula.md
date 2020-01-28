---
title: "Sistema de copias de seguridad"
date: 2020-01-22T08:04:52+01:00
draft: false
---

Implementar un sistema de copias de seguridad para las instancias del cloud, teniendo en cuenta las siguientes características:

- Selecciona una aplicación para realizar el proceso: bacula, amanda, shell script con tar, rsync, dar, afio, etc.
- Utiliza una de las instancias como servidor de copias de seguridad, añadiéndole un volumen y almacenando localmente las copias de seguridad que consideres adecuadas en él.
- El proceso debe realizarse de forma completamente automática
- Selecciona qué información es necesaria guardar (listado de paquetes, ficheros de configuración, documentos, datos, etc.)
- Realiza semanalmente una copia completa
- Realiza diariamente una copia incremental o diferencial (decidir cual es más adecuada)
- Implementa una planificación del almacenamiento de copias de seguridad para una ejecución prevista de varios años, detallando qué copias completas se almacenarán de forma permanente y cuales se irán borrando
- Añade tu sistema de copias a coconut cuando esté disponible
- Selecciona un directorio de datos "críticos" que deberá almacenarse cifrado en la copia de seguridad, bien encargándote de hacer la copia manualmente o incluyendo la contraseña de cifrado en el sistema
- Incluye en la copia los datos de las nuevas aplicaciones que se vayan instalando durante el resto del curso
- Utiliza saturno u otra opción que se te facilite como equipo secundario para almacenar las copias de seguridad. Solicita acceso o la instalación de las aplicaciones que sean precisas.

La corrección consistirá tanto en la restauración puntual de un fichero en cualquier fecha como la restauración completa de una de las instancias la última semana de curso.

***

Instalación de **mysql**:

```
debian@serranito:~$ sudo apt install mariadb-server mariadb-client
```

Instalación de **bacula**:

```
debian@serranito:~$ sudo apt install bacula bacula-client bacula-common-mysql bacula-director-mysql bacula-server
```

!bacula1.png!

Le damos a **Si** y ponemos la contraseña.

Vamos el fichero de configuración:

```
debian@serranito:~$ sudo nano /etc/bacula/bacula-dir.conf 
```

Vamos a configurar el volumen.

```
debian@serranito:~$ lsblk -f

NAME FSTYPE LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINT
vda                                                                   
└─vda1
     ext4         6197e068-a892-45cb-9672-a05813e800ee      8G    14% /
vdb                                                                   
```

Creamos una partición con:

```
debian@serranito:~$ sudo fdisk /dev/vdb 

Welcome to fdisk (util-linux 2.33.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x73b8d268.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-10485759, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-10485759, default 10485759): 

Created a new partition 1 of type 'Linux' and of size 5 GiB.

Command (m for help): p
Disk /dev/vdb: 5 GiB, 5368709120 bytes, 10485760 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x73b8d268

Device     Boot Start      End  Sectors Size Id Type
/dev/vdb1        2048 10485759 10483712   5G 83 Linux

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

La formateamos con:

```
debian@serranito:~$ sudo mkfs.ext4 /dev/vdb1 

mke2fs 1.44.5 (15-Dec-2018)
Creating filesystem with 1310464 4k blocks and 327680 inodes
Filesystem UUID: d65f629a-be7a-43a9-a93c-a5d2a507d8fc
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 
```

Creamos el directorio para las copias y le cambiamos los permisos:

```
debian@serranito:~$ sudo mkdir -p /bacula/Copias_de_Seguridad
debian@serranito:~$ sudo chown bacula:bacula /bacula -R
debian@serranito:~$ sudo chmod 755 /bacula -R
```

Necesitaremos el identificador siguiente para el fichero /etc/fstab.

```
debian@serranito:~$ lsblk -f | egrep "vdb1 *"
└─vdb1 ext4         d65f629a-be7a-43a9-a93c-a5d2a507d8fc                

debian@serranito:~$ sudo nano /etc/fstab
UUID=d65f629a-be7a-43a9-a93c-a5d2a507d8fc     /bacula/Copias_de_Seguridad     ext4     defaults     0     0
```

```
debian@serranito:~$ lsblk -l
NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda  254:0    0  10G  0 disk 
vda1 254:1    0  10G  0 part /
vdb  254:16   0   5G  0 disk 
vdb1 254:17   0   5G  0 part /bacula/Copias_de_Seguridad
```

Vamos a configurar el otro fichero de configuración bacula-sd.conf

```
debian@serranito:~$ sudo nano /etc/bacula/bacula-sd.conf
``` 

Editamos el siguiente fichero:

```
debian@serranito:~$ sudo nano /etc/bacula/bconsole.conf

Director {
  Name = serranito-dir
  DIRport = 9101
  address = 10.0.0.8
  Password = "ernestovazquez11"
}

```

Instalamos los clientes de bacula:

```
debian@croqueta:~$ sudo apt install bacula-client
ubuntu@tortilla:~$ sudo apt install bacula-client
debian@serranito:~$ sudo apt install bacula-client
[root@salmorejo ~]# sudo dnf -y install bacula-client
```

A continuación pondremos la configuración en el siguiente fichero:

```
debian@croqueta:~$ sudo nano /etc/bacula/bacula-fd.conf

Director {
 Name = serranito-dir
 Password = "ernestovazquez11"
}

Director {
 Name = serranito-mon
 Password = "ernestovazquez11"
 Monitor = yes
}

FileDaemon {
 Name = croqueta-fd
 FDport = 9102
 WorkingDirectory = /var/lib/bacula
 Pid Directory = /run/bacula
 Maximum Concurrent Jobs = 20
 Plugin Directory = /usr/lib/bacula
 FDAddress = 10.0.0.10
}

Messages {
 Name = Standard
 director = serranito-dir = all, !skipped, !restored
}
```

Reinciamos los servicios:

```
debian@serranito:~$ sudo systemctl restart bacula-fd.service
debian@croqueta:~$ sudo systemctl restart bacula-fd.service
ubuntu@tortilla:~$ sudo systemctl restart bacula-fd.service
[root@salmorejo ~]# sudo systemctl restart bacula-fd.service
```

Reiniciamos servicios del servidor en serranito:

```
debian@serranito:~$ sudo systemctl restart bacula-sd.service
debian@serranito:~$ sudo systemctl restart bacula-director.service
```

Abrimos los puertos 9102/TCP en Openstack

Puertos en salmorejo:

```
[root@salmorejo ~]# firewall-cmd --zone=public --permanent --add-port 9102/tcp
[root@salmorejo ~]# firewall-cmd --reload
```

Cambios para el funcionamiento:

Tenemos que quitar la siguiente linea del fichero de configuración del director:

```
debian@serranito:~$ sudo nano /etc/bacula/bacula-dir.conf 

Use Volume Once = yes
```

A continuación vamos a realizar una prueba con el siguiente comando:

```
*run
Using Catalog "mysql-bacula"
A job name must be specified.
The defined Job resources are:
     1: Daily-Backup-Serranito
     2: Daily-Backup-Croqueta
     3: Daily-Backup-Tortilla
     4: Daily-Backup-Salmorejo
     5: Weekly-Backup-Serranito
     6: Weekly-Backup-Croqueta
     7: Weekly-Backup-Tortilla
     8: Weekly-Backup-Salmorejo
     9: Monthly-Backup-Serranito
    10: Monthly-Backup-Croqueta
    11: Monthly-Backup-Tortilla
    12: Monthly-Backup-Salmorejo
    13: Restore-Serranito
    14: Restore-Croqueta
    15: Restore-Tortilla
    16: Restore-Salmorejo
Select Job resource (1-16): 2
Run Backup job
JobName:  Daily-Backup-Croqueta
Level:    Incremental
Client:   croqueta-fd
FileSet:  Copia-Croqueta
Pool:     Daily (From Job resource)
Storage:  Vol-Serranito (From Job resource)
When:     2020-01-24 08:15:11
Priority: 10
OK to run? (yes/mod/no): yes
Job queued. JobId=6
```

```
*status client
The defined Client resources are:
     1: serranito-fd
     2: croqueta-fd
     3: tortilla-fd
     4: salmorejo-fd
Select Client (File daemon) resource (1-4): 2
Connecting to Client croqueta-fd at 10.0.0.10:9102

croqueta-fd Version: 9.4.2 (04 February 2019)  x86_64-pc-linux-gnu debian buster/sid
Daemon started 23-Jan-20 17:22. Jobs: run=1 running=0.
 Heap: heap=114,688 smbytes=164,827 max_bytes=371,949 bufs=125 max_bufs=140
 Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
 Plugin: bpipe-fd.so 

Running Jobs:
JobId 6 Job Daily-Backup-Croqueta.2020-01-24_08.15.15_03 is running.
    Full Backup Job started: 24-Jan-20 08:15
    Files=0 Bytes=0 AveBytes/sec=0 LastBytes/sec=0 Errors=0
    Bwlimit=0 ReadBytes=0
    Files: Examined=0 Backed up=0
    SDReadSeqNo=6 fd=5 SDtls=0
Director connected at: 24-Jan-20 08:15
====
```

Ahora vamos a configurar los **labels**

* Label copia diaria

```
*label
Automatically selected Catalog: mysql-bacula
Using Catalog "mysql-bacula"
Automatically selected Storage: Vol-Serranito
Enter new Volume name: copiadiaria
Defined Pools:
     1: Daily
     2: Default
     3: File
     4: Monthly
     5: Scratch
     6: Vol-Backup
     7: Weekly
Select the Pool (1-7): 1
Connecting to Storage daemon Vol-Serranito at 10.0.0.8:9103 ...
Sending label command for Volume "copiadiaria" Slot 0 ...
3000 OK label. VolBytes=225 VolABytes=0 VolType=1 Volume="copiadiaria" Device="DispositivoCopia" (/bacula/Copias_de_Seguridad)
Catalog record for Volume "copiadiaria", Slot 0  successfully created.
Requesting to mount FileAutochanger1 ...
3001 OK mount requested. Device="DispositivoCopia" (/bacula/Copias_de_Seguridad)
```

* Label copia semanal

```
*label
Automatically selected Storage: Vol-Serranito
Enter new Volume name: copiasemanal
Defined Pools:
     1: Daily
     2: Default
     3: File
     4: Monthly
     5: Scratch
     6: Vol-Backup
     7: Weekly
Select the Pool (1-7): 7
Connecting to Storage daemon Vol-Serranito at 10.0.0.8:9103 ...
Sending label command for Volume "copiasemanal" Slot 0 ...
3000 OK label. VolBytes=227 VolABytes=0 VolType=1 Volume="copiasemanal" Device="DispositivoCopia" (/bacula/Copias_de_Seguridad)
Catalog record for Volume "copiasemanal", Slot 0  successfully created.
Requesting to mount FileAutochanger1 ...
3906 File device ""DispositivoCopia" (/bacula/Copias_de_Seguridad)" is always mounted.
```

* Label copia mensual

```
*label
Automatically selected Storage: Vol-Serranito
Enter new Volume name: copiamensual
Defined Pools:
     1: Daily
     2: Default
     3: File
     4: Monthly
     5: Scratch
     6: Vol-Backup
     7: Weekly
Select the Pool (1-7): 4
Connecting to Storage daemon Vol-Serranito at 10.0.0.8:9103 ...
Sending label command for Volume "copiamensual" Slot 0 ...
3000 OK label. VolBytes=228 VolABytes=0 VolType=1 Volume="copiamensual" Device="DispositivoCopia" (/bacula/Copias_de_Seguridad)
Catalog record for Volume "copiamensual", Slot 0  successfully created.
Requesting to mount FileAutochanger1 ...
3906 File device ""DispositivoCopia" (/bacula/Copias_de_Seguridad)" is always mounted.
```

Ahora ya estarán las copias en funcionamiento como podemos ver a continuación:

```
*status client
The defined Client resources are:
     1: serranito-fd
     2: croqueta-fd
     3: tortilla-fd
     4: salmorejo-fd
Select Client (File daemon) resource (1-4): 2
Connecting to Client croqueta-fd at 10.0.0.10:9102

croqueta-fd Version: 9.4.2 (04 February 2019)  x86_64-pc-linux-gnu debian buster/sid
Daemon started 23-Jan-20 17:22. Jobs: run=2 running=0.
 Heap: heap=114,688 smbytes=283,225 max_bytes=531,545 bufs=95 max_bufs=149
 Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
 Plugin: bpipe-fd.so 

Running Jobs:
Director connected at: 24-Jan-20 08:26
No Jobs running.
====

Terminated Jobs:
 JobId  Level    Files      Bytes   Status   Finished        Name 
===================================================================
     6  Full      3,265    66.46 M  OK       24-Jan-20 08:18 Daily-Backup-Croqueta
====
You have messages.
```

Vemos las copias tras unos dias:

```
debian@serranito:~$ sudo bconsole
Connecting to Director 10.0.0.8:9101
1000 OK: 103 serranito-dir Version: 9.4.2 (04 February 2019)
Enter a period to cancel a command.
*list jobs
Automatically selected Catalog: mysql-bacula
Using Catalog "mysql-bacula"
+-------+-------------------------+---------------------+------+-------+----------+-------------+-----------+
| JobId | Name                    | StartTime           | Type | Level | JobFiles | JobBytes    | JobStatus |
+-------+-------------------------+---------------------+------+-------+----------+-------------+-----------+
|     2 | Daily-Backup-Serranito  | 2020-01-23 20:59:00 | B    | F     |        0 |           0 | f         |
|     3 | Daily-Backup-Croqueta   | 2020-01-23 20:59:00 | B    | I     |        0 |           0 | f         |
|     4 | Daily-Backup-Tortilla   | 2020-01-23 20:59:00 | B    | F     |        0 |           0 | f         |
|     5 | Daily-Backup-Salmorejo  | 2020-01-23 20:59:02 | B    | F     |        0 |           0 | f         |
|     6 | Daily-Backup-Croqueta   | 2020-01-24 08:15:17 | B    | F     |    3,265 |  66,465,322 | T         |
|     7 | Daily-Backup-Tortilla   | 2020-01-24 18:43:48 | B    | F     |    5,081 |  44,487,922 | T         |
|     8 | Daily-Backup-Serranito  | 2020-01-24 20:00:00 | B    | F     |    3,626 | 109,089,625 | T         |
|     9 | Daily-Backup-Croqueta   | 2020-01-24 20:00:00 | B    | I     |      151 |  52,232,068 | T         |
|    10 | Daily-Backup-Tortilla   | 2020-01-24 20:00:00 | B    | I     |       17 |     408,189 | T         |
|    11 | Daily-Backup-Salmorejo  | 2020-01-24 20:00:02 | B    | F     |   43,792 | 382,767,581 | T         |
|    12 | Daily-Backup-Serranito  | 2020-01-25 20:00:00 | B    | I     |       92 |  40,268,522 | T         |
|    13 | Daily-Backup-Croqueta   | 2020-01-25 20:00:00 | B    | I     |       68 |  17,457,020 | T         |
|    14 | Daily-Backup-Tortilla   | 2020-01-25 20:00:00 | B    | I     |       71 |   5,501,461 | T         |
|    15 | Daily-Backup-Salmorejo  | 2020-01-25 20:00:02 | B    | I     |       32 |   1,016,011 | T         |
|    16 | Weekly-Backup-Serranito | 2020-01-25 23:00:00 | B    | F     |    3,644 | 114,646,015 | T         |
|    17 | Weekly-Backup-Croqueta  | 2020-01-25 23:00:00 | B    | F     |    3,343 | 105,841,267 | T         |
|    18 | Weekly-Backup-Tortilla  | 2020-01-25 23:00:00 | B    | F     |    5,081 |  44,519,985 | T         |
|    19 | Weekly-Backup-Salmorejo | 2020-01-25 23:00:02 | B    | F     |   43,792 | 382,765,046 | T         |
|    20 | Daily-Backup-Serranito  | 2020-01-26 20:00:01 | B    | I     |       99 |  30,279,626 | T         |
|    21 | Daily-Backup-Croqueta   | 2020-01-26 20:00:01 | B    | I     |       79 |  23,448,247 | T         |
|    22 | Daily-Backup-Tortilla   | 2020-01-26 20:00:01 | B    | I     |       72 |   2,061,292 | T         |
|    23 | Daily-Backup-Salmorejo  | 2020-01-26 20:00:03 | B    | I     |       44 |   1,031,324 | T         |
|    24 | Daily-Backup-Serranito  | 2020-01-27 20:00:00 | B    | I     |       99 |  36,099,298 | T         |
|    25 | Daily-Backup-Croqueta   | 2020-01-27 20:00:00 | B    | I     |       66 |   4,784,258 | T         |
|    26 | Daily-Backup-Tortilla   | 2020-01-27 20:00:00 | B    | I     |      119 |   7,934,669 | T         |
|    27 | Daily-Backup-Salmorejo  | 2020-01-27 20:00:02 | B    | I     |    2,197 |  14,469,352 | T         |
+-------+-------------------------+---------------------+------+-------+----------+-------------+-----------+
You have messages.
```
