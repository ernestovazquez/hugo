
---
title: "Proxy, proxy inverso y balanceadores de carga"
date: 2020-02-18T09:04:52+01:00
draft: false
---

## Balanceador de carga

Primero vamos a instalar **HAproxy** 

    vagrant@balanceador:~$ sudo apt install haproxy

Vamos al fichero de configuración y pondremos lo siguiente:

```
vagrant@balanceador:/etc/haproxy$ sudo nano haproxy.cfg 

global
    daemon
    maxconn 256
    user    haproxy
    group   haproxy
    log     127.0.0.1       local0
    log     127.0.0.1       local1  notice
defaults
    mode    http
    log     global
    timeout connect 5000ms
    timeout client  50000ms
    timeout server  50000ms
listen granja_cda
    bind 192.168.1.80:80
    mode http
    stats enable
    stats auth  cda:cda
    balance roundrobin
    server uno 10.10.10.11:80 maxconn 128 check port 80
    server dos 10.10.10.22:80 maxconn 128 check port 80

root@balanceador:~# systemctl restart haproxy

root@balanceador:~#  systemctl enable haproxy
Synchronizing state of haproxy.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable haproxy
```

![](https://i.imgur.com/wIRP8cm.png)
![](https://i.imgur.com/HoFJRsd.png)

Nos pedirá un usuario y un password, ambos **cda**.

![](https://i.imgur.com/UQi7ARw.png)

![](https://i.imgur.com/tZWK0SG.png)


Vamos a ver el **log de apache1**. Como podemos apreciar figura como única dirección IP, la IP interna de la máquina balanceador ya que dicho balanceador hemos espcificado un balanceo **tipo roundrobin**, este es que se encarga de hacer la peticiones a estos servidores.

```
root@apache1:~# cat /var/log/apache2/access.log 

10.10.10.1 - - [19/Feb/2020:15:43:13 +0000] "GET / HTTP/1.1" 200 436 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0"
10.10.10.1 - - [19/Feb/2020:15:43:14 +0000] "GET / HTTP/1.1" 200 436 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0"
10.10.10.1 - - [19/Feb/2020:15:43:14 +0000] "GET /favicon.ico HTTP/1.1" 404 435 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0"
10.10.10.1 - - [19/Feb/2020:15:45:25 +0000] "GET /favicon.ico HTTP/1.1" 404 435 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0"
```

### Configurar la persistencia de conexiones Web (sticky sessions)

```
vagrant@balanceador:~$ sudo nano /etc/haproxy/haproxy.cfg 

...
    cookie PHPSESSID prefix                             
    server uno 10.10.10.11:80 cookie EL_UNO maxconn 128 
    server dos 10.10.10.22:80 cookie EL_DOS maxconn 128    
```        

![](https://i.imgur.com/VbAymPi.png)

![](https://i.imgur.com/ZeYasXv.png)

Cabecera:

![](https://i.imgur.com/nTa2cWK.png)

