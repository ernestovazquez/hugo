---
title: "Introducción a la integración continua"
date: 2019-12-15T15:29:57+01:00
draft: false
---

Tarea 1: Despliegue de una página web estática (build, deploy) (4 puntos)

En esta práctica investiga como generar una página web estática con la herramienta que elegiste en la práctica 1 de la asignatura y desplegarla en el servicio que utilizaste en esa práctica.

- En el repositorio GitHub sólo tienen que estar los ficheros markdown.
- La página se debe generar en el sistema de integración continúa, por lo tanto debemos instalar las herramientas necesarias.
- Investiga si podemos desplegar de forma automática en el servicio elegido (si es necesario cambia el servicio de hosting para el despliegue).

Tarea 2: Integración continúa de aplicación django (Test + Deploy) (6 puntos)

Estudia las distintas pruebas que se han realizado, y modifica el código de la aplicación para que al menos una de ella no se ejecute de manera exitosa. (1 punto)

Crea un fichero .travis.yml para realizar de los tests en travis. Entrega el fichero .travis.yml, una captura de pantalla con un resltado exitoso de la IC y otro con un error.(1 punto)

Entrega un breve descripción de los pasos más importantes para realizar el despliegue desde travis. (3 puntos)

***

Vamos a crear el repositorio donde vamos a trabajar.
También vamos a crear la rama gh-pages, donde estarán los ficheros generados por Hugo.

Fichero de configuración **.travis.yml** .

```
ernesto@honda:~/blog$ sudo nano .travis.yml

install:
  - wget https://github.com/gohugoio/hugo/releases/download/v0.53/hugo_0.53_Linux-64bit.deb && sudo dpkg -i hugo*.deb
  - git clone https://github.com/ernestovazquez/hugo.git

script:
  - rm hugo*.deb
  - hugo new site sitio
  - mkdir -p sitio/content/post
  - mkdir -p sitio/static/img
  - mv content/post/* sitio/content/post
  - mv static/img/* sitio/static/img
  - mv config.toml sitio/
  - cd sitio/themes && git clone https://github.com/nirocfz/arabica && cd ..
  - hugo -t arabica
  - rm -rf themes/arabica

deploy:
  provider: pages
  local-dir: sitio/public
  skip-cleanup: true
  github-token: $GH_TOKEN
  keep-history: true
  on:
    branch: master
  fqdn: ernestovazquez.es
```

Fichero **config.toml** :

```
ernesto@honda:~/blog$ sudo nano config.toml

baseURL = "https://ernestovazquez.es" 
title = "De Rookie a Leyenda" 
author = "Ernesto Vázquez" 
paginate = 3
theme = "arabica" 

[params]
description = "Ernesto Vázquez García" 
twitter = "ernestovazgar" 
```

Pasamos los ficheros Markdown que vamos a utilizar.

```
ernesto@honda:~$ ls blog/content/post/

oraclevspostgresql.md  servidorest1.md  servidorest3.md  servidorest6.md  servidorest8.md
resources              servidorest2.md  servidorest5.md  servidorest7.md
```

Generamos el **TOKEN**

Nos dirigimos a **Settings, Developer settings y  Personal access tokens**

![](https://i.imgur.com/MWLteYr.png)

Vamos a **Travis** y entramos en los ajustes del repositorio:

![](https://i.imgur.com/WMu31Bj.png)

Generamos un fichero de pruebas:

```
ernesto@honda:~/blog$ sudo nano content/post/pruebatravis.md 
[sudo] password for ernesto: 
ernesto@honda:~/blog$ git add *
ernesto@honda:~/blog$ git commit -m "prueba"
[master b897f97] prueba
 1 file changed, 3 insertions(+), 2 deletions(-)
ernesto@honda:~/blog$ git push
Enumerando objetos: 9, listo.
Contando objetos: 100% (9/9), listo.
Compresión delta usando hasta 4 hilos
Comprimiendo objetos: 100% (4/4), listo.
Escribiendo objetos: 100% (5/5), 449 bytes | 449.00 KiB/s, listo.
Total 5 (delta 2), reusado 0 (delta 0)
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
To github.com:ernestovazquez/blog.git
   31cee85..b897f97  master -> master
```

Se inica travis automaticamente y una vez pasado la integración podremos ver la página.

![](https://i.imgur.com/OnKD1od.png)

![](https://i.imgur.com/9NiTbAx.png)

Una vez pasado la integración podremos ver la página.

![](https://i.imgur.com/VYCaj9g.png)

