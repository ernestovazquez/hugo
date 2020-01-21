---
title: "Introducción a la integración continua"
date: 2019-12-15T15:29:57+01:00
draft: false
---

Vamos a crear el repositorio donde vamos a trabajar.
También vamos a crear la rama gh-pages, donde estarán los ficheros generados por Hugo.

Fichero de configuración **.travis.yml** .

```
ernesto@honda:~/blog$ sudo nano .travis.yml

install:
  - wget https://github.com/gohugoio/hugo/releases/download/v0.53/hugo_0.53_Linux-64bit.deb && sudo dpkg -i hugo*.deb
  - git clone https://github.com/ernestovazquez/blog.git

script:
  - rm hugo*.deb
  - hugo new site sitio
  - mkdir -p sitio/content/post
  - mv blog/content/post/* sitio/content/post
  - mv blog/config.toml sitio/
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
```

Fichero **config.toml** :

```
ernesto@honda:~/blog$ sudo nano config.toml

baseURL = "/blog"
title = "De Rookie a Leyenda"
author = "Ernesto Vázquez"
paginate = 3
theme = "arabica"

[params]
description = "Ernesto Vázquez García"
```

Pasamos los ficheros Markdown que vamos a utilizar.

```
ernesto@honda:~$ ls blog/content/post/

oraclevspostgresql.md  servidorest1.md  servidorest3.md  servidorest6.md  servidorest8.md
resources              servidorest2.md  servidorest5.md  servidorest7.md
```

Generamos el **TOKEN**

Nos dirigimos a **Settings, Developer settings y  Personal access tokens**

Vamos a **Travis** y entramos en los ajustes del repositorio:

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

