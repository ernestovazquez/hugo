---
title: "T6. DNS."
date: 2019-11-19T13:15:31+01:00
draft: false
---

Vamos a instalar un servidor dns que nos permita gestionar la resolución directa e inversa de nuestros nombres. Cada alumno va a poseer un servidor dns con autoridad sobre un subdominio de nuestro dominio principal gonzalonazareno.org, que se llamará tu_nombre.gonzalonazareno.org.

Indica al profesor el nombre de tu dominio para que pueda realizar la delegación en el servidor DNS principal papion-dns.

El servidor DNS se va a instalar en el servidor1 (croqueta). Y en un primer momento se configurará de la siguiente manera:

- El servidor DNS se llama croqueta.tu_nombre.gonzalonazareno.org y va a ser el servidor con autoridad para la zona tu_nombre.gonzalonazareno.org.
- El servidor debe resolver el nombre de los tres servidores.
- El servidor debe resolver los distintos servicios (virtualhost y servidor de base de datos).
- Debes determinar si la resolución directa se hace con dirección ip fijas o flotantes del cloud depediendo del servicio que se este prestando.
- Debes considerar la posibilidad de hacer dos zonas de resolución inversa para resolver ip fijas o flotantes del cloud.

Entrega el resultado de las siguientes consultas a los servidores de nuestra red :

- El servidor DNS con autoridad sobre la zona del dominio tu_nombre.gonzalonazareno.org
- La dirección IP de algún servidor
- Una resolución de un nombre de un servicio
- Un resolución inversa de IP fija, y otra resolución inversa de IP flotante. (Esta consulta la debes hacer directamente preguntando a tu servidor).



Asignamos **zonas** :

```
debian@croqueta:/etc/bind$ sudo nano named.conf.local
``` 

```
include "/etc/bind/zones.rfc1918";

zone "ernesto.gonzalonazareno.org"{
type master;
file "db.ernesto.gonzalonazareno.org";
};

// Zona inversa 172.22

zone "22.172.in-addr.arpa" {
type master;
file "db.172.22.rev";
};

// Zona inversa 10.0

zone "0.0.10.in-addr.arpa" {
type master;
file "db.10.0.rev";
};
```

Zona **directa** :

```
debian@croqueta:/var/cache/bind$ sudo nano db.ernesto.gonzalonazareno.org
``` 

```
;
; BIND reverse data file for broadcast zone
;
$TTL    604800
@       IN      SOA     croqueta.ernesto.gonzalonazareno.org. root.croqueta.ernesto.gonzalonazareno.org. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      croqueta.ernesto.gonzalonazareno.org.

$ORIGIN ernesto.gonzalonazareno.org.

croqueta        IN      A       172.22.200.111
croqueta-int    IN      A       10.0.0.3
tortilla        IN      A       172.22.200.21
tortilla-int    IN      A       10.0.0.14
salmorejo       IN      A       172.22.200.109
salmorejo-int   IN      A       10.0.0.7
www             IN      CNAME   salmorejo
cloud           IN      CNAME   salmorejo
mysql           IN      CNAME   tortilla-int
```

Zona inversa **172.22:**

```
debian@croqueta:/var/cache/bind$ sudo nano db.172.22.rev 
```

```
;
; BIND reverse data file for local loopback interface
;
$TTL    604800
@       IN      SOA     croqueta.ernesto.gonzalonazareno.org. root.croqueta.ernesto.gonzalonazareno.org. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      croqueta.ernesto.gonzalonazareno.org.
$ORIGIN 22.172.in-addr.arpa.
111.200 IN      PTR     croqueta.ernesto.gonzalonazareno.org.
21.200  IN      PTR     tortilla.ernesto.gonzalonazareno.org.
109.200     IN      PTR     salmorejo.ernesto.gonzalonazareno.org.
```

Zona inversa **10.0:**

```
debian@croqueta:/var/cache/bind$ sudo nano db.10.0.rev 
```

```
;
; BIND reverse data file for local loopback interface
;
$TTL    604800
@       IN      SOA     croqueta.ernesto.gonzalonazareno.org. root.croqueta.ernesto.gonzalonazareno.org. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      croqueta.ernesto.gonzalonazareno.org.
$ORIGIN 0.0.10.in-addr.arpa.
3       IN      PTR     croqueta.ernesto.gonzalonazareno.org.
14      IN      PTR     tortilla.ernesto.gonzalonazareno.org.
7       IN      PTR     salmorejo.ernesto.gonzalonazareno.org.
```

##### El servidor DNS con autoridad sobre la zona del dominio tu_nombre.gonzalonazareno.org

```
debian@croqueta:/var/cache/bind$ dig ns ernesto.gonzalonazareno.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> ns ernesto.gonzalonazareno.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7093
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 910ebe7458b30ba70a35cdcc5ddba6b48222d52ba5f3a7cc (good)
;; QUESTION SECTION:
;ernesto.gonzalonazareno.org. IN  NS

;; ANSWER SECTION:
ernesto.gonzalonazareno.org. 604775 IN  NS  croqueta.ernesto.gonzalonazareno.org.

;; Query time: 1 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: Mon Nov 25 10:02:28 UTC 2019
;; MSG SIZE  rcvd: 107
```

##### La dirección IP de algún servidor:

```
debian@croqueta:~$ dig tortilla.ernesto.gonzalonazareno.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> tortilla.ernesto.gonzalonazareno.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 13948
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 317eec718b77ae277a1712595ddba716e4d90fe1c1f402fd (good)
;; QUESTION SECTION:
;tortilla.ernesto.gonzalonazareno.org. IN A

;; ANSWER SECTION:
tortilla.ernesto.gonzalonazareno.org. 604800 IN A 172.22.200.21

;; AUTHORITY SECTION:
ernesto.gonzalonazareno.org. 604677 IN  NS  croqueta.ernesto.gonzalonazareno.org.

;; Query time: 2 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: Mon Nov 25 10:04:06 UTC 2019
;; MSG SIZE  rcvd: 132
```

##### Una resolución de un nombre de un servicio

```
debian@croqueta:~$ dig cloud.ernesto.gonzalonazareno.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> cloud.ernesto.gonzalonazareno.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32135
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 8779638e6eda4d5476fa44655ddba746277fcd6e3b0dc3f2 (good)
;; QUESTION SECTION:
;cloud.ernesto.gonzalonazareno.org. IN  A

;; ANSWER SECTION:
cloud.ernesto.gonzalonazareno.org. 602018 IN CNAME salmorejo.ernesto.gonzalonazareno.org.
salmorejo.ernesto.gonzalonazareno.org. 602018 IN A 172.22.200.109

;; AUTHORITY SECTION:
ernesto.gonzalonazareno.org. 604629 IN  NS  croqueta.ernesto.gonzalonazareno.org.

;; Query time: 1 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: Mon Nov 25 10:04:54 UTC 2019
;; MSG SIZE  rcvd: 153
```

##### Un resolución inversa de IP fija, y otra resolución inversa de IP flotante. (Esta consulta la debes hacer directamente preguntando a tu servidor).

```
debian@croqueta:/var/cache/bind$ dig @localhost -x 172.22.200.109

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> @localhost -x 172.22.200.109
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 6590
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, A
[Servicios] - Tarea 4 - Servidores Cloud. DNS.
DDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 2688c473e8d3d93c06178d025ddf8c1438467882cebd6ef0 (good)
;; QUESTION SECTION:
;109.200.22.172.in-addr.arpa. IN  PTR

;; ANSWER SECTION:
109.200.22.172.in-addr.arpa. 604800 IN  PTR salmorejo.ernesto.gonzalonazareno.org.

;; AUTHORITY SECTION:
200.22.172.in-addr.arpa. 604800 IN  NS  croqueta.ernesto.gonzalonazareno.org.

;; ADDITIONAL SECTION:
croqueta.ernesto.gonzalonazareno.org. 604800 IN A 172.22.200.111

;; Query time: 0 msec
;; SERVER: ::1#53(::1)
;; WHEN: Thu Nov 28 08:57:56 UTC 2019
;; MSG SIZE  rcvd: 174
```

```
debian@croqueta:~$ dig @localhost -x 10.0.0.7

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> @localhost -x 10.0.0.7
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 30943
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 57605f092835beb7ea5b09865ddba7ab3df602b76c96bde8 (good)
;; QUESTION SECTION:
;7.0.0.10.in-addr.arpa.   IN  PTR

;; ANSWER SECTION:
7.0.0.10.in-addr.arpa.  604800  IN  PTR salmorejo.ernesto.gonzalonazareno.org.

;; AUTHORITY SECTION:
0.0.10.in-addr.arpa.  604800  IN  NS  croqueta.ernesto.gonzalonazareno.org.

;; ADDITIONAL SECTION:
croqueta.ernesto.gonzalonazareno.org. 604800 IN A 172.22.200.111

;; Query time: 0 msec
;; SERVER: ::1#53(::1)
;; WHEN: Mon Nov 25 10:06:35 UTC 2019
;; MSG SIZE  rcvd: 168
```
