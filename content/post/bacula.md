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


