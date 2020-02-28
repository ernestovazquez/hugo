---
title: "Práctica 7 Movimiento de datos"
date: 2020-02-28T15:04:52+01:00
draft: false
---

## Alumno 1 (Ernesto Vázquez García)

**1. Realiza una exportación del esquema de SCOTT usando la consola Enterprise Manager, con las siguientes condiciones:**
**• Exporta tanto la estructura de las tablas como los datos de las mismas.**
**• Excluye la tabla BONUS y los departamentos sin empleados.**
**• Realiza una estimación previa del tamaño necesario para el fichero de exportación.**
**• Programa la operación para dentro de 5 minutos.**
**• Genera un archivo de log en el directorio raíz.
Realiza ahora la operación con Oracle Data Pump.**

Para realizar la exportación primero tenemos que entrar en la consola de Enterprise Manager.

Nos dirigimos a la pestaña de **Movimiento de Datos** y entramos en **Exportar a Archivos de Exportación**

![](https://i.imgur.com/CDFjUSe.png)


A continuación vamos a marcar el **tipo de exportación**:

![](https://i.imgur.com/cxE91Do.png)

En esta pantalla vamos a elegir que esquema queremos exportar en nuestro caso seleccionamos el **esquema de SCOTT**:

![](https://i.imgur.com/SFhR3ek.png)

A continuación vamos a desplegar las opciones avanzadas para poder seleccionar la tabla bonus.

Para ello, seleccionamos lo siguiente:

- Tipo de Objeto: **TABLE**
- Expresión de Nombre de Objeto: **='BONUS'**

![](https://i.imgur.com/etrojbm.png)

Ahora vamos a darle nombre al trabajo y podemos planificar la hora y fecha de inicio. Aquí podremos programar la operación para dentro de 5 minutos.

![](https://i.imgur.com/oy4mEPJ.png)

Una vez le demos a **Siguiente** nos aparecerá un **resumen** de los datos que vamos a realizar.

![](https://i.imgur.com/5zhoCam.png)

Por último nos aparecerá una pantalla de carga, y una vez finalize el proceso nos pararecerá la siguiente pantalla.

![](https://i.imgur.com/Dzq7yxR.png)

Para verificar que se ha creado el fichero de importación nos vamos a la siguiente carpeta:

![](https://i.imgur.com/KtBVHQQ.png)

![](https://i.imgur.com/AA46Aul.png)

Como podemos apreciar se ha exportado las tablas deseadas.

Para exportarlo con Oracle Data Pump podremos hacerlo con el siguiente comando:

    expdp system/dios dumpfile=EXP_SCOTT.DMP schemas=scott directory=DATA_PUMP_DIR
    
![](https://i.imgur.com/1wmWWnR.png)

**2. Importa el fichero obtenido anteriormente usando Enterprise Manager pero en un usuario distinto de la misma base de datos.**

Este paso se puede hacer por Enterprise Manager o por la terminal.

En nuestro caso vamos a elegir **Enterprise Manager**. Para ello, nos dirigimos a **Importar de Archivos de Exportación.**

Nos conectamos con otro usuario y accedemos a la misma pestaña que antes.

![](https://i.imgur.com/7CIo48M.png)

Como se puede apreciar me he conectado con el usuario: **ERNESTO.** El cual no tiene ninguna tabla introducida.

Ahora entramos en la pestaña de **Importar de Archivos de Exportación**

![](https://i.imgur.com/XpmuAow.png)

En la siguiente pantalla vamos a tener que agregar el esquema anterior.

![](https://i.imgur.com/6yFqdBp.png)

Aqui podremos ver el nombre del fichero de log y el directorio donde se guarda dicho fichero.

![](https://i.imgur.com/Ph0Hc0j.png)

![](https://i.imgur.com/F11FNdH.png)

![](https://i.imgur.com/nDWxxE9.png)

Ya tendremos en el nuevo usuario el esquema de SCOTT realizando una importación.

Para **importar** un esquema de la base de datos con **Oracle Data Pump** podremos lo siguiente.

    impdp ernesto/ernesto dumpfile=EXP_SCOTT.DMP schemas=scott directory=DATA_PUMP_DIR remap_schema=scott:ernesto

**3. Realiza una exportación de la estructura de todas las tablas de la base de datos usando el comando expdp de Oracle Data Pump probando todas las posibles opciones que ofrece dicho comando y documentándolas adecuadamente.**

Primero vamos a vamos a ver las diferentes opciones que nos ofrece el comando expdp, para ello vamos a ver el manual.

    expdp HELP=Y

![](https://i.imgur.com/GB9TmhU.png)

Como podemos apreciar estas son las diferentes opciones que tiene el comando **expdp.**

A continuación vamos a comentar todas las opciones que nos ofrece esta herramienta.

- **ATTACH**: Nombre de un Job existente para conectarte. Esta opción necesita los privilegios **EXP_FULL_DATABASE** para otros esquemas.
- **COMPRESSION**: Comprime el contenido del archivo.
- **CONSISTENT**: Copia de seguridad de lectura consistente.
- **CONTENT**: Especifica datos para cargar.
- **DIRECTORY**: Nombre del objeto que apunta a un directorio.
- **DUMPFILE**: El nombre del archivo de datos de exportación.
- **ESTIMATE**: Solamente calcule el espacio en disco requerido.
- **ESTIMATE_ONLY**: Se utiliza para calcular el espacio en disco solo para datos.
- **EXCLUDE**: Excluye de la exportación de la base de datos un tipo de objeto.
- **FILESIZE**: El tamaño máximo de archivo permitido para cualquier archivo de volcado de exportación.
- **FLASHBACK_TIME**: Al igual que en **CONSISTENT**, permite una exportación consistente de lectura
- **FULL**: Exportación completa de la base de datos. Esta opción necesita los privilegios **EXP_FULL_DATABASE**.
- **INCLUDE**: Al contrario que **EXCLUDE**, esta opción es para incluir específicamente en la exportación.
- **JOB_NAME**: Parecido al parámetro **ATTACH**, Nombre por el que se puede hacer referencia al trabajo de exportación.
- **LOGFILE**: El nombre del archivo de registro de exportación.
- **NOLOGFILE**: Para suprimir la creación del archivo en la exportación.
- **PARALLEL**: El número máximo de subprocesos concurrentes para la exportación.
- **PARFILE**: Nombre del archivo de parámetros específicos del sistema operativo.
- **QUERY**: Filtrado de datos y nombres de objetos durante la exportación.
- **REUSE_DUMPFILES**: Sobrescribir los archivos existentes del volcado de la exportación
- **SAMPLE**: Porcentaje de datos a exportar.
- **SCHEMAS**: El esquema que queremos exportar. Esta opción necesita los privilegios **EXP_FULL_DATABASE**
- **STATUS**: El estado dl trabajo en frecuencia.
- **TABLES**: Lista de tablas para una exportación.
- **TABLESPACES**: Lista de espacios de tabla para una exportación.
- **TRANSPORT_FULL_CHECK**: Verifica los segmentos de almacenamiento de todas las tablas.
- **TRANSPORT_TABLESAPCES**: Lista de espacios de tablas desde los que se descargarán los metadatos.

> Estas opciones se pueden concatenar para tener mayor número de parámetros y opciones para tener la exportación deseada.

Vamos a realizar una exportación de ejemplo

![](https://i.imgur.com/Hk0ckfl.png)

**4. Intenta realizar operaciones similares de importación y exportación con las herramientas proporcionadas con MySQL desde línea de comandos, documentando el proceso.**

Primero tenemos que agregar unas tablas a la base de datos que queremos exportar para realizar la prueba.

Para ello hacemos lo siguiente:

```
MariaDB [(none)]> CREATE DATABASE exportacion;
Query OK, 1 row affected (0.001 sec)

MariaDB [(none)]> use exportacion;
Database changed

MariaDB [exportacion]> show tables;
+-----------------------+
| Tables_in_exportacion |
+-----------------------+
| animales              |
| clientes              |
+-----------------------+
2 rows in set (0.001 sec)
```

Realizaremos la exportacion:

```
vagrant@mysql:~$ mysqldump -u ernesto -p exportacion > exportacion.sql
Enter password: 

vagrant@mysql:~$ ls
exportacion.sql
```
Tendremos que crear la nueva base de datos para la importacion.

```
vagrant@mysql:~$ sudo mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 62
Server version: 10.3.18-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE DATABASE importacion;
Query OK, 1 row affected (0.001 sec)

MariaDB [(none)]> Bye
```

Importación a otra base de datos.

```
vagrant@mysql:~$ mysql -u ernesto -p importacion < exportacion.sql 
Enter password: 

vagrant@mysql:~$ sudo mysql -u ernesto -p importacion
Enter password: 
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 65
Server version: 10.3.18-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [importacion]> show tables;
+-----------------------+
| Tables_in_importacion |
+-----------------------+
| animales              |
| clientes              |
+-----------------------+
2 rows in set (0.001 sec)
```

![](https://i.imgur.com/4vqq0bG.png)

Referencia: [Artículo sobre MySQL](https://www.stackscale.com/es/blog/importar-exportar-bases-datos-mysql-mariadb/)

**5. Realiza operaciones de importación y exportación de colecciones en MongoDB.**

Primero vamos a crear un usuario y una base de datos con algunas colecciones dentro para la exportación.

```
> use peliculas
switched to db peliculas

> db.peliculas.save({titulo:'Batman el caballero oscuro'});
WriteResult({ "nInserted" : 1 })

> db.peliculas.find();
{ "_id" : ObjectId("5e54d0003488f9e6b3ffc688"), "titulo" : "Batman el caballero oscuro" }

> show dbs;
admin      0.000GB
config     0.000GB
local      0.000GB
peliculas  0.000GB

> db
peliculas

> db.createUser(
... {
... user: "ernesto",
... pwd: "ernesto",
... roles: ["dbOwner"]
... }
... )
Successfully added user: { "user" : "ernesto", "roles" : [ "dbOwner" ] }
```

Ahora si muestro los usuarios tendremos la siguiente información:

```
> show users
{
	"_id" : "peliculas.ernesto",
	"userId" : UUID("0079cc4b-bd5c-4978-b6a7-8b32aed32965"),
	"user" : "ernesto",
	"db" : "peliculas",
	"roles" : [
		{
			"role" : "dbOwner",
			"db" : "peliculas"
		}
	],
	"mechanisms" : [
		"SCRAM-SHA-1",
		"SCRAM-SHA-256"
	]
}
```

Ahora vamos a realizar la exportación de dicha colección.

```
vagrant@mongodb:~$ mongodump -d peliculas -o exportacion -u ernesto -p ernesto
2020-02-25T07:53:26.070+0000	writing peliculas.peliculas to 
2020-02-25T07:53:26.071+0000	done dumping peliculas.peliculas (1 document)

vagrant@mongodb:~$ ls
exportacion
```

```
vagrant@mongodb:~/exportacion$ tree
.
└── peliculas
    ├── peliculas.bson
    └── peliculas.metadata.json

1 directory, 2 files
```

Como se puede apreciar se ha realizado correctamente la exportación.

Ahora vamos a importar la colección en otra base de datos. También se puede importar a otro servidor/máquina local que tenga un nombre de usuario y una contraseña.

    vagrant@mongodb:~$ mongorestore -d importacion exportacion/peliculas/ -u ernesto -p ernesto


