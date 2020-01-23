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

![bacula1](https://raw.githubusercontent.com/ernestovazquez/hugo/gh-pages/img/bacula1.png)

Le damos a **Yes** y ponemos la contraseña.

Vamos al fichero de configuración:

```
debian@serranito:~$ sudo cat /etc/bacula/bacula-dir.conf

Director {
  Name = serranito-dir
  DIRport = 9101
  QueryFile = "/etc/bacula/scripts/query.sql"
  WorkingDirectory = "/var/lib/bacula"
  PidDirectory = "/run/bacula"
  Maximum Concurrent Jobs = 20
  Password = "ernestovazquez11"
  Messages = Daemon
  DirAddress = 10.0.0.8
}


JobDefs {
  Name = "Tarea-Diaria"
  Type = Backup
  Level = Incremental
  Client = serranito-fd
  Schedule = "Programa-Diario"
  Pool = Daily
  Storage = Vol-Serranito
  Messages = Standard
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}


JobDefs {
  Name = "Tarea-Semanal"
  Type = Backup
  Client = serranito-fd
  Schedule = "Programa-Semanal"
  Pool = Weekly
  Storage = Vol-Serranito
  Messages = Standard
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}


JobDefs {
  Name = "Tarea-Mensual"
  Type = Backup
  Client = serranito-fd
  Schedule = "Programa-Mensual"
  Pool = Monthly
  Storage = Vol-Serranito
  Messages = Standard
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}



# Diariamente

Job {
 Name = "Daily-Backup-Serranito"
 JobDefs = "Tarea-Diaria"
 Client = "serranito-fd"
 FileSet= "Copia-Serranito"
}

Job {
 Name = "Daily-Backup-Croqueta"
 JobDefs = "Tarea-Diaria"
 Client = "croqueta-fd"
 FileSet= "Copia-Croqueta"
}

Job {
 Name = "Daily-Backup-Tortilla"
 JobDefs = "Tarea-Diaria"
 Client = "tortilla-fd"
 FileSet= "Copia-Tortilla"
}

Job {
 Name = "Daily-Backup-Salmorejo"
 JobDefs = "Tarea-Diaria"
 Client = "salmorejo-fd"
 FileSet= "Copia-Salmorejo"
}

# Semanalmente

Job {
 Name = "Weekly-Backup-Serranito"
 JobDefs = "Tarea-Semanal"
 Client = "serranito-fd"
 FileSet= "Copia-Serranito"
}

Job {
 Name = "Weekly-Backup-Croqueta"
 JobDefs = "Tarea-Semanal"
 Client = "croqueta-fd"
 FileSet= "Copia-Croqueta"
}

Job {
 Name = "Weekly-Backup-Tortilla"
 JobDefs = "Tarea-Semanal"
 Client = "tortilla-fd"
 FileSet= "Copia-Tortilla"
}

Job {
 Name = "Weekly-Backup-Salmorejo"
 JobDefs = "Tarea-Semanal"
 Client = "salmorejo-fd"
 FileSet= "Copia-Salmorejo"
}

# Mensualmente

Job {
 Name = "Monthly-Backup-Serranito"
 JobDefs = "Tarea-Mensual"
 Client = "serranito-fd"
 FileSet= "Copia-Serranito"
}

Job {
 Name = "Monthly-Backup-Croqueta"
 JobDefs = "Tarea-Mensual"
 Client = "croqueta-fd"
 FileSet= "Copia-Croqueta"
}

Job {
 Name = "Monthly-Backup-Tortilla"
 JobDefs = "Tarea-Mensual"
 Client = "tortilla-fd"
 FileSet= "Copia-Tortilla"
}

Job {
 Name = "Monthly-Backup-Salmorejo"
 JobDefs = "Tarea-Mensual"
 Client = "salmorejo-fd"
 FileSet= "Copia-Salmorejo"
}


Job {
 Name = "Restore-Serranito"
 Type = Restore
 Client=serranito-fd
 FileSet= "Copia-Serranito"
 Storage = Vol-Serranito
 Pool = Vol-Backup
 Messages = Standard
}

Job {
 Name = "Restore-Croqueta"
 Type = Restore
 Client=croqueta-fd
 FileSet= "Copia-Croqueta"
 Storage = Vol-Serranito
 Pool = Vol-Backup
 Messages = Standard
}

Job {
 Name = "Restore-Tortilla"
 Type = Restore
 Client=tortilla-fd
 FileSet= "Copia-Tortilla"
 Storage = Vol-Serranito
 Pool = Vol-Backup
 Messages = Standard
}

Job {
 Name = "Restore-Salmorejo"
 Type = Restore
 Client=salmorejo-fd
 FileSet= "Copia-Salmorejo"
 Storage = Vol-Serranito
 Pool = Vol-Backup
 Messages = Standard
}


FileSet {
 Name = "Copia-Serranito"
 Include {
    Options {
        signature = MD5
        compression = GZIP
    }
    File = /home
    File = /etc
    File = /var
    File = /bacula
 }
 Exclude {
    File = /nonexistant/path/to/file/archive/dir
    File = /proc
    File = /var/cache
    File = /var/tmp
    File = /tmp
    File = /sys
    File = /.journal
    File = /.fsck
 }
}

FileSet {
 Name = "Copia-Croqueta"
 Include {
    Options {
        signature = MD5
        compression = GZIP
    }
    File = /home
    File = /etc
    File = /var
 }
 Exclude {
    File = /var/lib/bacula
    File = /nonexistant/path/to/file/archive/dir
    File = /proc
    File = /var/tmp
    File = /tmp
    File = /sys
    File = /.journal
    File = /.fsck
 }
}

FileSet {
 Name = "Copia-Tortilla"
 Include {
    Options {
        signature = MD5
        compression = GZIP
    }
    File = /home
    File = /etc
    File = /var
 }
 Exclude {
    File = /var/lib/bacula
    File = /nonexistant/path/to/file/archive/dir
    File = /proc
    File = /var/cache
    File = /var/tmp
    File = /tmp
    File = /sys
    File = /.journal
    File = /.fsck
 }
}

FileSet {
 Name = "Copia-Salmorejo"
 Include {
    Options {
        signature = MD5
        compression = GZIP
    }
    File = /home
    File = /etc
    File = /var
    File = /usr/share/nginx
 }
 Exclude {
    File = /var/lib/bacula
    File = /nonexistant/path/to/file/archive/dir
    File = /proc
    File = /var/cache
    File = /var/tmp
    File = /tmp
    File = /sys
    File = /.journal
    File = /.fsck
 }
}


Schedule {
 Name = "Programa-Diario"
 Run = Level=Incremental Pool=Daily daily at 20:59
}

Schedule {
 Name = "Programa-Semanal"
 Run = Level=Full Pool=Weekly sat at 23:50
}

Schedule {
 Name = "Programa-Mensual"
 Run = Level=Full Pool=Monthly 1st sun at 23:50 
}

# Indicar los cliente

Client {
 Name = serranito-fd
 Address = 10.0.0.8
 FDPort = 9102
 Catalog = mysql-bacula
 Password = "ernestovazquez11"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}

Client {
 Name = croqueta-fd
 Address = 10.0.0.10
 FDPort = 9102
 Catalog = mysql-bacula
 Password = "ernestovazquez11"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}

Client {
 Name = tortilla-fd
 Address = 10.0.0.4
 FDPort = 9102
 Catalog = mysql-bacula
 Password = "ernestovazquez11"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}

Client {
 Name = salmorejo-fd
 Address = 10.0.0.13
 FDPort = 9102
 Catalog = mysql-bacula
 Password = "ernestovazquez11"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}



Storage {
 Name = Vol-Serranito
 Address = 10.0.0.8
 SDPort = 9103
 Password = "ernestovazquez11"
 Device = FileAutochanger1
 Media Type = File
 Maximum Concurrent Jobs = 10
}

# Indicar la base de datos

Catalog {
 Name = mysql-bacula
 dbname = "bacula"; DB Address = "localhost"; dbuser = "bacula"; dbpassword = "ernestovazquez11"
}


Pool {
 Name = Daily
 Use Volume Once = yes
 Pool Type = Backup
 AutoPrune = yes
 VolumeRetention = 10d
 Recycle = yes
}


Pool {
 Name = Weekly
 Use Volume Once = yes
 Pool Type = Backup
 AutoPrune = yes
 VolumeRetention = 30d
 Recycle = yes
}


Pool {
 Name = Monthly
 Use Volume Once = yes
 Pool Type = Backup
 AutoPrune = yes
 VolumeRetention = 365d
 Recycle = yes
}

Pool {
 Name = Vol-Backup
 Pool Type = Backup
 Recycle = yes 
 AutoPrune = yes
 Volume Retention = 365 days 
 Maximum Volume Bytes = 50G
 Maximum Volumes = 100
 Label Format = "Remoto"
}


# Por defecto
# -------------------------------------------------------

# Reasonable message delivery -- send most everything to email address
#  and to the console
Messages {
  Name = Standard
#
# NOTE! If you send to two email or more email addresses, you will need
#  to replace the %r in the from field (-f part) with a single valid
#  email address in both the mailcommand and the operatorcommand.
#  What this does is, it sets the email address that emails would display
#  in the FROM field, which is by default the same email as they're being
#  sent to.  However, if you send email to more than one address, then
#  you'll have to set the FROM address manually, to a single address.
#  for example, a 'no-reply@mydomain.com', is better since that tends to
#  tell (most) people that its coming from an automated source.

#
  mailcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula: %t %e of %c %l\" %r"
  operatorcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula: Intervention needed for %j\" %r"
  mail = root = all, !skipped
  operator = root = mount
  console = all, !skipped, !saved
#
# WARNING! the following will create a file that you must cycle from
#          time to time as it will grow indefinitely. However, it will
#          also keep all your messages if they scroll off the console.
#
  append = "/var/log/bacula/bacula.log" = all, !skipped
  catalog = all
}


#
# Message delivery for daemon messages (no job).
Messages {
  Name = Daemon
  mailcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula daemon message\" %r"
  mail = root = all, !skipped
  console = all, !skipped, !saved
  append = "/var/log/bacula/bacula.log" = all, !skipped
}

# Default pool definition
Pool {
  Name = Default
  Pool Type = Backup
  Recycle = yes                       # Bacula can automatically recycle Volumes
  AutoPrune = yes                     # Prune expired volumes
  Volume Retention = 365 days         # one year
  Maximum Volume Bytes = 50G          # Limit Volume size to something reasonable
  Maximum Volumes = 100               # Limit number of Volumes in Pool
}

# Scratch pool definition
Pool {
  Name = Scratch
  Pool Type = Backup
}

#
# Restricted console used by tray-monitor to get the status of the director
#
Console {
  Name = serranito-mon
  Password = "6URHdHljsXACPlryk6so_xCbGqrmkYgFb"
  CommandACL = status, .status
}
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

Storage { 
 Name = serranito-sd
 SDPort = 9103 
 WorkingDirectory = "/var/lib/bacula"
 Pid Directory = "/run/bacula"
 Maximum Concurrent Jobs = 20
 SDAddress = 10.0.0.8
}


Director {
 Name = serranito-dir
 Password = "ernestovazquez11"
}


Director {
 Name = serranito-mon
 Password = "bacula"
 Monitor = yes
}


Autochanger {
 Name = FileAutochanger1
 Device = DispositivoCopia
 Changer Command = ""
 Changer Device = /dev/null
}


Device {
 Name = DispositivoCopia
 Media Type = File
 Archive Device = /bacula/Copias_de_Seguridad
 LabelMedia = yes;
 Random Access = Yes;
 AutomaticMount = yes;
 RemovableMedia = no;
 AlwaysOpen = no;
 Maximum Concurrent Jobs = 5
}


Messages {
  Name = Standard
  director = serranito-dir = all
}
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

```
[root@salmorejo ~]# sudo nano /etc/bacula/bacula-fd.conf

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
  Name = salmorejo-fd
  FDport = 9102
  WorkingDirectory = /var/spool/bacula
  Pid Directory = /var/run
  Maximum Concurrent Jobs = 20
  Plugin Directory = /usr/lib64/bacula
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


