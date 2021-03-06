---
title: "Métricas, logs y monitorización"
date: 2020-01-28T08:04:52+01:00
draft: false
---

Crea una nueva instancia de nombre serranito o reutiliza la que ya se estaba utilizando para copias de seguridad y realiza las partes que elijas entre las siguientes sobre de croqueta, salmorejo, tortilla y serranito (se puede hacer una, dos o las tres partes y el peso de la tarea se modificará consecuentemente):

- Métricas: recolección, gestión centralizada, filtrado y representación gráfica de los parámetros que consideres necesarios
- Monitorización: Configuración de alertas por uso excesivo de recursos (memoria, disco raíz, etc.) y disponibilidad de los servicios. Alertas por correo, telegram, etc.
- Gestión de logs: Implementa un sistema centralizado de filtrado que permita quedarse con registros con prioridad error, critical, alert o emergency. Representa gráficamente los datos relevantes extraidos de los logs o configura el envío por correo al administrador de los logs relevantes (una opción o ambas)

Detalla en la documentación claramente las características de la implementación elegida, así como la forma de comprobar las características exigidas.

***

En la nueva máquina llamada **Serranito** vamos a montar el sistema de monitorización. Con las siguientes combinaciones:

- **Telegraf**: Se encarga de recolectar todos los datos que le pasamos mediante el fichero de configuración.

- **InfluxDB**: Es donde Telegraf manda toda esta información.

- **Grafana**: Es el Dashboard que se encarga de mostrar toda la información que InfluxDB tiene almacenado en las Bases de Datos.

## Instalación de InfluxDB

Vamos a empezar con la instalación de la base de datos donde se va a guardar todos los datos que va ofrecer telegraf.

```
debian@serranito:~$ wget https://dl.influxdata.com/influxdb/releases/influxdb_1.7.9_amd64.deb
debian@serranito:~$ sudo dpkg -i influxdb_1.7.9_amd64.deb
```

Una vez ha terminado la instalación, arrancaremos el servicio.

![](https://i.imgur.com/bEW6Vnc.png)

## Instalación de Telegraf

```
debian@serranito:~$ wget https://dl.influxdata.com/telegraf/releases/telegraf_1.13.1-1_amd64.deb
debian@serranito:~$ sudo dpkg -i telegraf_1.13.1-1_amd64.deb
```

Reiniciamos los servicios:

```
root@serranito:~# systemctl restart telegraf
```

Si miramos las bases de datos, se ha creado una base de datos para telegraf.

```
> show databases
name: databases
name
----
_internal
telegraf
> 
```

Vamos a ver la configuración que tenemos que hacer en el fichero de configuración telegraf.

```
debian@serranito:~$ sudo nano /etc/telegraf/telegraf.conf 
```

Lo dejaremos por defecto pero cambiaremos lo siguiente:

```
[agent]
  interval = "10s"
  flush_interval = "10s"
  precision = "s"
  hostname = "serranito"

[[outputs.influxdb]]
  urls = ["http://127.0.0.1:8086"]
  database = "telegraf"
  retention_policy = "default"
  write_consistency = "any"
  timeout = "5s"
```

Reiniciamos los servicios:

```
root@serranito:~# systemctl restart telegraf
```

Ya tendremos listo dicho servidor.

## Instalación de grafana:

```
debian@serranito:~$ sudo apt-get install -y adduser libfontconfig1
debian@serranito:~$ wget https://dl.grafana.com/oss/release/grafana_6.5.3_amd64.deb
debian@serranito:~$ sudo dpkg -i grafana_6.5.3_amd64.deb
```

Iniciamos los servicios:

```
debian@serranito:~$ sudo systemctl restart grafana-server.service 
```

Tenemos que habilitar el puerto 3000 en OpenStack:

Si queremos usar el dominio solamente tendremos que añadirlo en el DNS:

```
grafana         IN      CNAME   serranito
```

Ya pondremos entrar a través del siguiente enlace

[grafana.ernesto.gonzalonazareno.org:3000](grafana.ernesto.gonzalonazareno.org:3000)

Ya podremos entrar desde el navegador a grafana

![](https://i.imgur.com/4KnpAul.png)

Accedemos con el usuario admin/admin y nos pedirá que cambiemos la contraseña.

![](https://i.imgur.com/INyHHfD.png)

A partir de ahora ya podremos configurar y enlazar la base de datos, InfluxDB y telegraf.

Le damos a *Add data source* y seleccionamos InfluxDB.

![](https://i.imgur.com/PVs2qcQ.png)

![](https://i.imgur.com/wyvY2Aq.png)

Guardamos y hacemos la prueba

Si todo ha salido bien es hora de continuar con la creación del dashboard.

Vamos a configurarlo:

![](https://i.imgur.com/HWTvwQt.png)

En la última parte podremos configurar las alertas al correo

![](https://i.imgur.com/hfcg9tD.png)

Le damos a guardar y ya tendremos el primero Dashboard guardado.

![](https://i.imgur.com/Y3oBrVb.png)

## Clientes

Vamos a ver la configuración que tenemos que realizar en los demás servidores para tener en la misma base de datos todos los servidores y se pueda ver y comparar las gráficas de una manera mas eficiente. 

### Croqueta:

Vamos a instalar telegraf y posteriormente vamos al fichero de configuración:

```
debian@croqueta:~$ sudo nano /etc/telegraf/telegraf.conf
```
Vamos a cambiar para que acceda al servidor de la base de datos:

```
[agent]
  interval = "15s"
  flush_interval = "15s"
  precision = ""
  hostname = "croqueta"

[[outputs.influxdb]]
  urls = ["http://10.0.0.8:8086"]
  database = "telegraf"
  skip_database_creation = true
  username = "admin"
  password = "admin"
```

Reiniciamos los servicios:

```
debian@croqueta:~$ sudo systemctl restart telegraf
```

### Tortilla:

Igual que antes, tenemos que instalar telegraf y nos vamos al fichero de configuración:

Vamos a cambiar para que acceda al servidor de la base de datos:

```
ubuntu@tortilla:~$ sudo nano /etc/telegraf/telegraf.conf

[agent]
  interval = "15s"
  flush_interval = "15s"
  precision = ""
  hostname = "tortilla"

[[outputs.influxdb]]
  urls = ["http://10.0.0.8:8086"]
  database = "telegraf"
  skip_database_creation = true
  username = "admin"
  password = "admin"
```

Reiniciamos los servicios:

```
ubuntu@tortilla:~$ sudo systemctl restart telegraf

ubuntu@tortilla:~$ sudo systemctl status telegraf
● telegraf.service - The plugin-driven server agent for reporting metrics into InfluxDB
   Loaded: loaded (/lib/systemd/system/telegraf.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2020-01-28 13:20:20 CET; 2s ago
     Docs: https://github.com/influxdata/telegraf
 Main PID: 5935 (telegraf)
    Tasks: 7 (limit: 547)
   CGroup: /system.slice/telegraf.service
           └─5935 /usr/bin/telegraf -config /etc/telegraf/telegraf.conf -config-directory /etc/telegraf/telegraf.d

Jan 28 13:20:20 tortilla systemd[1]: Started The plugin-driven server agent for reporting metrics into InfluxDB.
Jan 28 13:20:21 tortilla telegraf[5935]: 2020-01-28T12:20:21Z I! Starting Telegraf 1.13.1
Jan 28 13:20:21 tortilla telegraf[5935]: 2020-01-28T12:20:21Z I! Loaded inputs: system cpu disk diskio kernel mem processes swap
Jan 28 13:20:21 tortilla telegraf[5935]: 2020-01-28T12:20:21Z I! Loaded aggregators:
Jan 28 13:20:21 tortilla telegraf[5935]: 2020-01-28T12:20:21Z I! Loaded processors:
Jan 28 13:20:21 tortilla telegraf[5935]: 2020-01-28T12:20:21Z I! Loaded outputs: influxdb
Jan 28 13:20:21 tortilla telegraf[5935]: 2020-01-28T12:20:21Z I! Tags enabled: host=tortilla
Jan 28 13:20:21 tortilla telegraf[5935]: 2020-01-28T12:20:21Z I! [agent] Config: Interval:15s, Quiet:false, Hostname:"tortilla", Flush Interval:15s
```

### Salmorejo

Por último vamos a configurar el servidor con CentOS.

```
[centos@salmorejo ~]$ sudo nano /etc/telegraf/telegraf.conf 

[agent]
  interval = "15s"
  flush_interval = "15s"
  precision = ""
  hostname = "salmorejo"

[[outputs.influxdb]]
  urls = ["http://10.0.0.8:8086"]
  database = "telegraf"
  skip_database_creation = true
  username = "admin"
  password = "admin"
```

Reiniciamos y vemos el estado del servicio:

```
[centos@salmorejo ~]$ sudo systemctl restart telegraf

[centos@salmorejo ~]$ sudo systemctl status telegraf
● telegraf.service - The plugin-driven server agent for reporting metrics into InfluxDB
   Loaded: loaded (/usr/lib/systemd/system/telegraf.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2020-01-28 13:28:25 CET; 1s ago
     Docs: https://github.com/influxdata/telegraf
 Main PID: 17779 (telegraf)
    Tasks: 9 (limit: 4980)
   Memory: 19.8M
   CGroup: /system.slice/telegraf.service
           └─17779 /usr/bin/telegraf -config /etc/telegraf/telegraf.conf -config-directory /etc/telegraf/telegraf.d

ene 28 13:28:25 salmorejo systemd[1]: Stopped The plugin-driven server agent for reporting metrics into InfluxDB.
ene 28 13:28:25 salmorejo systemd[1]: Started The plugin-driven server agent for reporting metrics into InfluxDB.
ene 28 13:28:25 salmorejo telegraf[17779]: 2020-01-28T12:28:25Z I! Starting Telegraf 1.13.2
ene 28 13:28:25 salmorejo telegraf[17779]: 2020-01-28T12:28:25Z I! Loaded inputs: cpu disk kernel mem system net diskio processes swap
ene 28 13:28:25 salmorejo telegraf[17779]: 2020-01-28T12:28:25Z I! Loaded aggregators:
ene 28 13:28:25 salmorejo telegraf[17779]: 2020-01-28T12:28:25Z I! Loaded processors:
ene 28 13:28:25 salmorejo telegraf[17779]: 2020-01-28T12:28:25Z I! Loaded outputs: influxdb
ene 28 13:28:25 salmorejo telegraf[17779]: 2020-01-28T12:28:25Z I! Tags enabled: host=salmorejo
ene 28 13:28:25 salmorejo telegraf[17779]: 2020-01-28T12:28:25Z I! [agent] Config: Interval:15s, Quiet:false, Hostname:"salmorejo", Flush Interval:15s
[centos@salmorejo ~]$ 
```

Una vez, configurado dichos ficheros podemos ver que grafana ya reconoce el servidores.

![](https://i.imgur.com/9c1ifGN.png)

## Creación de los Dashboard

Vamos a crear un Dashboard nuevo y cambiamos el host para que salgan los diferentes servisores en la misma gráfica.

![](https://i.imgur.com/hsHtZMR.png)

En el último paso podemos definir las alertas:

![](https://i.imgur.com/IHgzIpA.png)

Le damos a guardar y nos saldrá en el panel principal de la siguiente forma:

![](https://i.imgur.com/krEgZi1.png)

Ahora vamos a ir configurando los diferentes **dashboard**.

![](https://i.imgur.com/5gqApFw.png)

![](https://i.imgur.com/7eQu9Ib.png)

![](https://i.imgur.com/V5hfAj7.png)

## Alertas por correo, telegram, etc.

Vamos a ver una utilidad que nos ofrece grafana, las alertas son muy interesantes y útiles para poder ver errores o una alerta de exceso de recursos de nuestros servidores.

Vamos a ver como se configuran:

Primero vamos a dirigirnos a la siguiente pestaña de grafana:

![](https://i.imgur.com/QjltVa9.png)

Le daremos a **"Add channel"**:

![](https://i.imgur.com/UhzsSHD.png)

En este panel podremos configurar los tipos de canales para transmitir las alertas, en mi caso voy a configurarlo para **"Discord"**, una aplicación muy útil (parecida a *Slack*), donde tendremos un servidor con usuarios y a ese canal de texto se le puede dar privilegios para ciertos usuarios. 

Aqui tendremos que poner el nombre de la alerta, el tipo de canal para leer los mensajes de alerta y un ***Webhook URL***, que es una URL que añades al sistema para recibir eventos de email. 

![](https://i.imgur.com/c0jrdZA.png)

A continuación, vamos a crear y configurar un servidor de ***Discord***, 

Una vez creado entramos en los **ajustes** del **canal de texto** que deseemos para tener ordenado las alertas.


Aquí le pondremos encontrar los ***Webhook*** que tenemos y poder crear más. Le pondremos un nombre para reconocerlo y copiamos la URL que nos proporciona para ponerlo en **Grafana**.

![](https://i.imgur.com/wRYYf0J.png)

Como podemos apreciar se ha creado correctamente el ***Webhook***, en nuetro canal de texto de **Discord**.

![](https://i.imgur.com/4HoMXWR.png)

Ahora solamente tendremos que poner la **URL**, en **Grafana**. y realizamos un **test**.

![](https://i.imgur.com/X9s52GP.png)

![](https://i.imgur.com/iOzk1xd.png)

Se ha **realizado correctamente el test** y nos lo muestra en nuestro **servidor de Discord** en el **canal de texto** que le hemos seleccionado.

Por último tendremos que indicarle el **nivel de alertas** en el **Dashboard de Grafana**.
Esta última opción nos permite configurar graficamente los niveles de alertas que vamos a recibir.

En mi caso, voy a poner que me avise si sobrepasa el valor en los diferentes servidores y si ese valor se mantiene durante un minuto me envie la alerta.

![](https://i.imgur.com/KThJsEr.png)

Aquí pondremos a quien se lo queremos enviar y el mensaje que va  recibir. En mi caso pondremos **"Discord"**, pero si queremos poner un **correo** solamente tendremos que activar **smtp** en **grafana.ini** y colocar la dirección de correo o también se podria enviar mediante Slack, para un uso más en equipo.

![](https://i.imgur.com/qZEm0Tm.png)


Vamos a ver un ejemplo de como envia la alerta:

![](https://i.imgur.com/yYLPRtb.png)

Como podemos apreciar, nos envia correctamente la **alerta**, sale el **nombre** y la **descripción** que hemos configurado previamente junto a una **imagen** donde podemos ver gráficamente el problema.

Aviso del correcto funcionamiento de los servidores:

![](https://i.imgur.com/eHv9Msg.png)

## Conclusión

En esta práctica hemos dado los primero pasos para monitorizar sistemas usando Grafana, Telegraf e InfluxDB. Es una herramienta bastante potente.

Podemos tener un **sistema de monitorización** de nuestros servidores de una manera bastante ordenada y gráfica. En mi caso he realizado los **Dashboard manualmente**, pero también podemos encontrar muchos realizados por la comunidad. Estos **Dashboard** nos permiten recoger datos de múltiples fuentes, almacenarlos en una base de datos y mostrar gráficos y estadísticas de manera eficiente, y nos ayuda a conocer mejor nuestro entorno.

En cuando a la gestión de logs, he probado con **"Graylog",** esta se tiene que instalar con MongoDB y no lo he conseguido con InfluxDB.

El **uso de alertas** en Grafana es muy útil a la hora de gestionar y monitorizar nuestra flota de servidores, las opciones de configuración van más alla que una simple configuración de limites, incluso a la hora de seleccionar las grandes cantidades de aplicaciones para que el cliente pueda recibir los avisos.

Como hemos podido apreciar los resultados en cuanto a nuestro *Dashboard*, son muy interesantes y útiles para nuestros pequeños servidores. Con tiempo y dedicación se puede conseguir un *Dashboard* perfecto.

Se puede realizar más configuraciones, otra sumamente interesante seria realizar la misma configuración pero en **Nodos remotos Windows** o monitorizar el **estado de una Raspberry Pi**, teniendo un Dashboard con toda la información de este pequeño equipo.
