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
  Name = directorserranito
  DIRport = 9101
  QueryFile = "/etc/bacula/scripts/query.sql"
  WorkingDirectory = "/var/lib/bacula"
  PidDirectory = "/run/bacula"
  Maximum Concurrent Jobs = 20
  Password = "ernestovazquez11"
  Messages = Daemon
  DirAddress = 10.0.0.11
}

JobDefs {
  Name = "JobDaily"
  Type = Backup
  Level = Incremental
  Client = serranito-fd
  Schedule = "HorarioDiario"
  Pool = Daily
  Storage = FileSerranito
  Messages = Standard
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}


JobDefs {
  Name = "JobWeekly"
  Type = Backup
  Client = serranito-fd
  Schedule = "HorarioSemanal"
  Pool = Weekly
  Storage = FileSerranito
  Messages = Standard
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}


JobDefs {
  Name = "JobMonthly"
  Type = Backup
  Client = serranito-fd
  Schedule = "HorarioMensual"
  Pool = Monthly
  Storage = FileSerranito
  Messages = Standard
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}


Job {
 Name = "BackupDiarioSerranito"
 JobDefs = "JobDaily"
 Client = "serranito-fd"
 FileSet= "CopiaSerranito"
}


Job {
 Name = "BackupDiarioCroqueta"
 JobDefs = "JobDaily"
 Client = "croqueta-fd"
 FileSet= "CopiaCroqueta"
}


Job {
 Name = "BackupDiarioTortilla"
 JobDefs = "JobDaily"
 Client = "tortilla-fd"
 FileSet= "CopiaTortilla"
}


Job {
 Name = "BackupDiarioSalmorejo"
 JobDefs = "JobDaily"
 Client = "salmorejo-fd"
 FileSet= "CopiaSalmorejo"
}


Job {
 Name = "BackupSemanalSerranito"
 JobDefs = "JobWeekly"
 Client = "serranito-fd"
 FileSet= "CopiaSerranito"
}


Job {
 Name = "BackupSemanalCroqueta"
 JobDefs = "JobWeekly"
 Client = "croqueta-fd"
 FileSet= "CopiaCroqueta"
}

Job {
 Name = "BackupSemanalTortilla"
 JobDefs = "JobWeekly"
 Client = "tortilla-fd"
 FileSet= "CopiaTortilla"
}

Job {
 Name = "BackupSemanalSalmorejo"
 JobDefs = "JobWeekly"
 Client = "salmorejo-fd"
 FileSet= "CopiaSalmorejo"
}


Job {
 Name = "BackupMensualSerranito"
 JobDefs = "JobMonthly"
 Client = "serranito-fd"
 FileSet= "CopiaSerranito"
}


Job {
 Name = "BackupMensualCroqueta"
 JobDefs = "JobMonthly"
 Client = "croqueta-fd"
 FileSet= "CopiaCroqueta"
}


Job {
 Name = "BackupMensualTortilla"
 JobDefs = "JobMonthly"
 Client = "tortilla-fd"
 FileSet= "CopiaTortilla"
}


Job {
 Name = "BackupMensualSalmorejo"
 JobDefs = "JobMonthly"
 Client = "salmorejo-fd"
 FileSet= "CopiaSalmorejo"
}


Job {
 Name = "Serranitorestaurar"
 Type = Restore
 Client=serranito-fd
 FileSet= "CopiaSerranito"
 Storage = FileSerranito
 Pool = Vol-Backup
 Messages = Standard
}

Job {
 Name = "Croquetarestaurar"
 Type = Restore
 Client=croqueta-fd
 FileSet= "CopiaCroqueta"
 Storage = FileSerranito
 Pool = Vol-Backup
 Messages = Standard
}

Job {
 Name = "Tortillarestaurar"
 Type = Restore
 Client=tortilla-fd
 FileSet= "CopiaTortilla"
 Storage = FileSerranito
 Pool = Vol-Backup
 Messages = Standard
}

Job {
 Name = "Salmorejorestaurar"
 Type = Restore
 Client=salmorejo-fd
 FileSet= "CopiaSalmorejo"
 Storage = FileSerranito
 Pool = Vol-Backup
 Messages = Standard
}


FileSet {
 Name = "CopiaSerranito"
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
 Name = "CopiaCroqueta"
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
 Name = "CopiaTortilla"
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
 Name = "CopiaSalmorejo"
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
 Name = "HorarioDiario"
 Run = Level=Incremental Pool=Daily daily at 21:30
}

Schedule {
 Name = "HorarioSemanal"
 Run = Level=Full Pool=Weekly sat at 20:00
}

Schedule {
 Name = "HorarioMensual"
 Run = Level=Full Pool=Monthly 1st sun at 22:00 
}


Client {
 Name = serranito-fd
 Address = 10.0.0.11
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
 Name = FileSerranito
 Address = 10.0.0.11
 SDPort = 9103
 Password = "ernestovazquez11"
 Device = FileAutochanger1
 Media Type = File
 Maximum Concurrent Jobs = 10
}


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
 Label Format = "Volumenlabel"
}


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
  Password = "fIw4a5kGo9eA-lUCL9vyFEpbrANkwG1K4"
  CommandACL = status, .status
}
```

Asignamos el **volumen** nuevo en OpenStack.

```
debian@serranito:~$ lsblk -f
NAME   FSTYPE LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINT
vda                                                                     
└─vda1 ext4         6197e068-a892-45cb-9672-a05813e800ee    7.3G    22% /
vdb                                                                     
```

```
debian@serranito:~$ sudo fdisk /dev/vdb

Welcome to fdisk (util-linux 2.33.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xe688ea79.

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
Disk identifier: 0xe688ea79

Device     Boot Start      End  Sectors Size Id Type
/dev/vdb1        2048 10485759 10483712   5G 83 Linux

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

Formateamos la partición

```
debian@serranito:~$ sudo mkfs.ext4 /dev/vdb1 

mke2fs 1.44.5 (15-Dec-2018)
Creating filesystem with 1310464 4k blocks and 327680 inodes
Filesystem UUID: 92d0b792-c34f-4866-a8fd-48380d4513ad
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 
```

```
debian@serranito:~$ lsblk -f

NAME   FSTYPE LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINT
vda                                                                     
└─vda1 ext4         6197e068-a892-45cb-9672-a05813e800ee    7.3G    22% /
vdb                                                                     
└─vdb1 ext4         92d0b792-c34f-4866-a8fd-48380d4513ad    
```            

Vamos a montarlo, para ello vamos a crear una carpeta donde estarán las copias de seguridad.

```
debian@serranito:~$ sudo mkdir -p /bacula/Copias
```
Editamos el fichero de configuración y agregamos lo siguiente:

```
debian@serranito:~$ sudo nano /etc/fstab 

UUID=92d0b792-c34f-4866-a8fd-48380d4513ad       /bacula/Copias  ext4    defaults        0       0
```

Dicho UUID se obtiene de esta manera:

```
debian@serranito:~$ lsblk -f | egrep "vdb1 *" | cut -d" " -f11
92d0b792-c34f-4866-a8fd-48380d4513ad
```

### Configuración bacula-sd.conf

Editamos el siguiente fichero de configuración:

```
debian@serranito:~$ sudo nano /etc/bacula/bacula-sd.conf 

Storage { 
 Name = serranito-sd
 SDPort = 9103 
 WorkingDirectory = "/var/lib/bacula"
 Pid Directory = "/run/bacula"
 Maximum Concurrent Jobs = 20
 SDAddress = 10.0.0.11
}


Director {
 Name = directorserranito
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
  director = directorserranito = all
}
```

Reiniciamos los servicios

```
debian@serranito:~$ sudo systemctl restart bacula-director.service
debian@serranito:~$ sudo systemctl restart bacula-sd.service
```

Modificamos el fichero **bconsole.conf**:

```
debian@serranito:~$ sudo nano /etc/bacula/bconsole.conf 

Director {
  Name = directorserranito
  DIRport = 9101
  address = 10.0.0.11
  Password = "ernestovazquez11" 
}
```

### Clientes

Vamos a instalar los clientes en los diferentes servidores del cloud.

```
debian@serranito:~$ sudo apt install bacula-client
debian@croqueta:~$ sudo apt install bacula-client
ubuntu@tortilla:~$ sudo apt install bacula-client
[centos@salmorejo ~]$ sudo dnf install bacula-client
```

```
debian@serranito:~$ sudo nano /etc/bacula/bacula-fd.conf 
debian@croqueta:~$ sudo nano /etc/bacula/bacula-fd.conf 
ubuntu@tortilla:~$ sudo nano /etc/bacula/bacula-fd.conf 

Director {
 Name = directorserranito
 Password = "ernestovazquez11"
}

Director {
 Name = serranito-mon
 Password = "ernestovazquez11"
 Monitor = yes
}

FileDaemon {
 Name = serranito-fd
 FDport = 9102
 WorkingDirectory = /var/lib/bacula
 Pid Directory = /run/bacula
 Maximum Concurrent Jobs = 20
 Plugin Directory = /usr/lib/bacula
 FDAddress = 10.0.0.11
}

Messages {
 Name = Standard
 director = directorserranito = all, !skipped, !restored
}
```

```
[centos@salmorejo ~]$ sudo nano /etc/bacula/bacula-fd.conf 

Director {
 Name = directorserranito
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
 director = directorserranito = all, !skipped, !restored
}
```

Reiniciamos los servicios:

```
debian@serranito:~$ sudo systemctl restart bacula-fd.service
debian@croqueta:~$ sudo systemctl restart bacula-fd.service
ubuntu@tortilla:~$ sudo systemctl restart bacula-fd.service
[centos@salmorejo ~]$ sudo systemctl restart bacula-fd.service
```

Reiniciamos los servicios en **serranito**

```
debian@serranito:~$ sudo systemctl restart bacula-sd.service
debian@serranito:~$ sudo systemctl restart bacula-director.service
```

Abrimos el puerto 9102/TCP en OpenstacK  y en salmorejo lo habilitamos mediante **firewall-cmd**:

```
[root@salmorejo ~]# firewall-cmd --zone=public --permanent --add-port 9102/tcp
success

[root@salmorejo ~]# firewall-cmd --reload
success
```

Estados de los clientes.

Serranito:

```
debian@serranito:~$ sudo bconsole
Connecting to Director 10.0.0.11:9101
1000 OK: 103 directorserranito Version: 9.4.2 (04 February 2019)
Enter a period to cancel a command.
*
*status client
The defined Client resources are:
     1: serranito-fd
     2: croqueta-fd
     3: tortilla-fd
     4: salmorejo-fd
Select Client (File daemon) resource (1-4): 1
Connecting to Client serranito-fd at 10.0.0.11:9102

serranito-fd Version: 9.4.2 (04 February 2019)  x86_64-pc-linux-gnu debian buster/sid
Daemon started 22-Jan-20 15:31. Jobs: run=0 running=0.
 Heap: heap=114,688 smbytes=22,024 max_bytes=22,041 bufs=68 max_bufs=68
 Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
 Plugin: bpipe-fd.so 

Running Jobs:
Director connected at: 22-Jan-20 15:40
No Jobs running.
====

Terminated Jobs:
====
```

Croqueta:

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
Daemon started 22-Jan-20 15:30. Jobs: run=0 running=0.
 Heap: heap=114,688 smbytes=22,022 max_bytes=22,039 bufs=68 max_bufs=68
 Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
 Plugin: bpipe-fd.so 

Running Jobs:
Director connected at: 22-Jan-20 15:46
No Jobs running.
====

Terminated Jobs:
====
```

Tortilla:

```
*status client
The defined Client resources are:
     1: serranito-fd
     2: croqueta-fd
     3: tortilla-fd
     4: salmorejo-fd
Select Client (File daemon) resource (1-4): 3
Connecting to Client tortilla-fd at 10.0.0.4:9102

tortilla-fd Version: 9.0.6 (20 November 2017) x86_64-pc-linux-gnu ubuntu 18.04
Daemon started 22-Jan-20 15:30. Jobs: run=0 running=0.
 Heap: heap=110,592 smbytes=21,993 max_bytes=22,010 bufs=68 max_bufs=68
 Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
 Plugin: bpipe-fd.so 

Running Jobs:
Director connected at: 22-Jan-20 15:47
No Jobs running.
====

Terminated Jobs:
====
```

Salmorejo:

```
*status client
The defined Client resources are:
     1: serranito-fd
     2: croqueta-fd
     3: tortilla-fd
     4: salmorejo-fd
Select Client (File daemon) resource (1-4): 4
Connecting to Client salmorejo-fd at 10.0.0.13:9102

salmorejo-fd Version: 9.0.6 (20 November 2017) x86_64-redhat-linux-gnu redhat (Core)
Daemon started 22-Jan-20 15:30. Jobs: run=0 running=0.
 Heap: heap=102,400 smbytes=21,996 max_bytes=22,013 bufs=68 max_bufs=68
 Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
 Plugin: bpipe-fd.so 

Running Jobs:
Director connected at: 22-Jan-20 15:48
No Jobs running.
====

Terminated Jobs:
====
```

Label:

```
debian@serranito:~$ sudo bconsole
Connecting to Director 10.0.0.11:9101
1000 OK: 103 directorserranito Version: 9.4.2 (04 February 2019)
Enter a period to cancel a command.
*label
Automatically selected Catalog: mysql-bacula
Using Catalog "mysql-bacula"
Automatically selected Storage: FileSerranito
Enter new Volume name: backups
Defined Pools:
     1: Daily
     2: Default
     3: File
     4: Monthly
     5: Scratch
     6: Vol-Backup
     7: Weekly
Select the Pool (1-7): 6
Connecting to Storage daemon FileSerranito at 10.0.0.11:9103 ...
Sending label command for Volume "backups" Slot 0 ...
3000 OK label. VolBytes=226 VolABytes=0 VolType=1 Volume="backups" Device="DCopia" (/bacula/Copias)
Catalog record for Volume "backups", Slot 0  successfully created.
Requesting to mount FileAutochanger1 ...
3906 File device ""DCopia" (/bacula/Copias)" is always mounted.
```

Ya esta configurado para que se hagan de manera programada.
Prueba manual:

```
*run
A job name must be specified.
The defined Job resources are:
     1: BackupDiarioSerranito
     2: BackupDiarioCroqueta
     3: BackupDiarioTortilla
     4: BackupDiarioSalmorejo
     5: BackupSemanalSerranito
     6: BackupSemanalCroqueta
     7: BackupSemanalTortilla
     8: BackupSemanalSalmorejo
     9: BackupMensualSerranito
    10: BackupMensualCroqueta
    11: BackupMensualTortilla
    12: BackupMensualSalmorejo
    13: Serranitorestaurar
    14: Croquetarestaurar
    15: Tortillarestaurar
    16: Salmorejorestaurar
Select Job resource (1-16): 2
Run Backup job
JobName:  BackupDiarioCroqueta
Level:    Incremental
Client:   croqueta-fd
FileSet:  CopiaCroqueta
Pool:     Daily (From Job resource)
Storage:  FileSerranito (From Job resource)
When:     2020-01-22 16:06:06
Priority: 10
OK to run? (yes/mod/no): yes
Job queued. JobId=5
You have messages.
```

Vemos el **estado** del mismo:
Primero pasa a estado de "Running Jobs" hasta que termine de realizar la copia de seguridad y pasará a "Terminated Jobs"

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
Daemon started 22-Jan-20 15:30. Jobs: run=0 running=0.
 Heap: heap=114,688 smbytes=48,265 max_bytes=48,282 bufs=113 max_bufs=113
 Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
 Plugin: bpipe-fd.so 

Running Jobs:
JobId 5 Job BackupDiarioCroqueta.2020-01-22_16.06.36_04 is running.
    Full Backup Job started: 22-Jan-20 16:06
    Files=0 Bytes=0 AveBytes/sec=0 LastBytes/sec=0 Errors=0
    Bwlimit=0 ReadBytes=0
    Files: Examined=0 Backed up=0
    SDReadSeqNo=6 fd=5 SDtls=0
Director connected at: 22-Jan-20 16:09
====

Terminated Jobs:
====
```

```
illo
```

