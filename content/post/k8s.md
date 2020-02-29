---
title: "Despliegue de una aplicación en un Cluster nuevo de k8s"
date: 2020-02-29T11:04:52+01:00
draft: false
---

Elige un sistema sencillo de instalación multinodo de k8s de los disponibles en cncf.io y despliega una aplicación explicando detalladamente las características.

Este cluster puede estar ubicado en MVs en tu propio equipo o en instancias nuevas en OpenStack.

***

## Introducción

Es una **distribución certificada** de Kubernetes. Para instalarla solo nos bajaremos un binario donde solo hace falta **512 MB de RAM** ya que es muy **ligero**.

## Desplegar cluster kubernetes

Accedemos a la primera máquina y procedemos a **instalar k3s**.

```
vagrant@maquina1:~$ sudo su -
root@maquina1:~# cd /usr/local/bin/
root@maquina1:/usr/local/bin# wget https://github.com/rancher/k3s/releases/download/v0.2.0/k3s

...

Saving to: ‘k3s’

k3s                                   100%[=======================================================================>]  35.99M  5.35MB/s    in 7.4s    

2020-02-29 12:58:24 (4.84 MB/s) - ‘k3s’ saved [37735552/37735552]
```

Como se puede observar ocupa 35.99M.
A continuación vamos a darle **permisos de ejecución**.

    root@maquina1:/usr/local/bin# chmod +x k3s 

Para comenzar a **desplegar nuestro cluster** (el nodo master) ejecutaremos lo siguiente:

    root@maquina1:/usr/local/bin# k3s server &

Ahora tendremos el comando k3s que nos permite **gestionar el cluster** y sin necesidad de autenticarnos en el cluster podemos ejecutar comandos con kubectl.

    vagrant@maquina1:~$ k3s kubectl get nodes
    NAME       STATUS   ROLES    AGE   VERSION
    maquina1   Ready    <none>   88s   v1.13.4-k3s.1

Podemos observar que ya tendremos la **maquina1** dentro del cluster.

A continuación vamos a configurar la segunda máquina.

Para ello tendremos que acceder y descargarnos el **binario k3s** como hemos realizado previamente.

```
vagrant@maquina2:~$ sudo su -
root@maquina2:~# cd /usr/local/bin/
root@maquina2:/usr/local/bin# wget https://github.com/rancher/k3s/releases/download/v0.2.0/k3s
root@maquina2:/usr/local/bin# chmod +x k3s 
```

Ahora vamos a necesitar **el token de autentificación** de la **maquina1** para que podamos introducir la segunda máquina en nuestro custer.

```
root@maquina1:~# cat /var/lib/rancher/k3s/server/node-token 
K10e890d9eea043fe30e8ef86a508aef63e7a3b040f8973f30607ef734fa0958c28::node:3b293a3082963dc4d0ea71923fd7e93e
```

Ya podremos ejecutar el siguiente comando, donde pondremos **la ip y el token** de la primera máquina para que se conecta al cluster.

```
root@maquina2:/usr/local/bin# k3s agent --server https://192.168.33.10:6443 --token K10e890d9eea043fe30e8ef86a508aef63e7a3b040f8973f30607ef734fa0958c28::node:3b293a3082963dc4d0ea71923fd7e93e
```

Se ejecuta **k3s** en la **maquina2** y si accedemos a la **maquina1** y le decimos al cluster que nos devuelva los nodos de nuestro cluster los saldria lo siguiente.

```
vagrant@maquina1:~$ k3s kubectl get nodes
NAME       STATUS   ROLES    AGE     VERSION
maquina1   Ready    <none>   15m     v1.13.4-k3s.1
maquina2   Ready    <none>   2m32s   v1.13.4-k3s.1
```

![](https://i.imgur.com/VMlsQMu.png)

Ahora vamos a realizar los mismos pasos con la **maquina3**.

```
vagrant@maquina3:~$ sudo su -
root@maquina3:~# cd /usr/local/bin/
root@maquina3:/usr/local/bin# wget https://github.com/rancher/k3s/releases/download/v0.2.0/k3s
root@maquina3:/usr/local/bin# chmod +x k3s 
root@maquina3:/usr/local/bin# k3s agent --server https://192.168.33.10:6443 --token K10e890d9eea043fe30e8ef86a508aef63e7a3b040f8973f30607ef734fa0958c28::node:3b293a3082963dc4d0ea71923fd7e93e
```

Ya tendremos los tres nodos en nuestro cluster

![](https://i.imgur.com/J7OTCU4.png)

Tambien podemos ver los pods que tenemos ahora mismo en el sistema.

```
vagrant@maquina1:~$ k3s kubectl get pods -n kube-system
NAME                             READY   STATUS      RESTARTS   AGE
coredns-7748f7f6df-psglj         1/1     Running     0          22m
helm-install-traefik-5rjj2       0/1     Completed   0          22m
svclb-traefik-699465787c-tmwhs   2/2     Running     0          22m
traefik-5468f76b59-zqgtp         1/1     Running     0          22m
```

A continuación vamos entrar en una máquina independiente, instalaremos **kubectl** y desde ahí vamos a trabajar en nuestro cluster.

Para ello vamos a entrar en nuestra máquina cliente.

	sudo apt-get update && sudo apt-get install -y apt-transport-https
	curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
	echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
	sudo apt-get update
	sudo apt-get install -y kubectl
    
Ahora vamos a configurar el entorno para autentificarnos en el cluster que acabamos de crear.

VAmos a crear un directorio para los ficheros de configuración para acceder al cluster.

```
vagrant@cliente:~$ mkdir .kube
vagrant@cliente:~$ cd .kube/
vagrant@cliente:~/.kube$ 
```

Ahora copiamos el fichero `/etc/rancher/k3s/k3s.yaml` para cambiar la dirección del server.

En la línea de server cambiamos localhost por la ip del servidor del cluster.

![](https://i.imgur.com/hHSYIfd.png)

Para cargar las credenciales vamos a crear una variable de entorno que guarda el fichero.

    vagrant@cliente:~/.kube$ export KUBECONFIG=~/.kube/config 
    
Ya podremos hacer el siguiente comando para que nos devuelve los nodos de nuestro cluster.

```
vagrant@cliente:~/.kube$ kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
maquina1   Ready    <none>   49m   v1.13.4-k3s.1
maquina2   Ready    <none>   36m   v1.13.4-k3s.1
maquina3   Ready    <none>   28m   v1.13.4-k3s.1
```

![](https://i.imgur.com/CimyFqG.png)

Ya estaria interaccionando con nuestro cluster donde estarian los tres nodos.

## Despliegue de Nginx

Lo primero que vamos a hacer es instalar un contenedor con un servidor nginx, con la siguiente instrucción:

```
vagrant@cliente:~$ kubectl create deploy nginx --image=nginx
deployment.apps/nginx created
```

Vamos a usar un recurso del cluster que se llama deploy (despliegue), donde indicaremos la imagen docker que se va a utilizar para crear el pod. 

Este pod va a ejecutar un contenedor y este contenedor va a ejecutar un proceso que será un servidor nginx.

```
vagrant@cliente:~$ kubectl get deploy,pod
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/nginx   1/1     1            1           95s

NAME                       READY   STATUS    RESTARTS   AGE
pod/nginx-5c7588df-6x2ff   1/1     Running   0          95s
```

Con ese comando podremos ver queya está funcionando. Donde tendremos un deployment y un pod.

Vamos a crear un nuevo recurso en el cluster de kubernetes llamado servicios, donde nos va a permitir acceder a las aplicaciones que se están ejecutando en los contenedores. Vamos a utilizar los siguientes tipos:

- **NodePort**: Este nos permite acceder desde el exterior.
- **ClusterIP**: Este también proporciona la posibilidad de acceder a la aplicación o al proceso del pod, pero internamente dentro del cluster de kubernetes, no desde el exterior.

A continuación vamos a crear un servicio, donde indicaremos el nombre del despliegue, el puerto donde esta funcionando la aplicación y el tipo de servicio.

```
vagrant@cliente:~$ kubectl expose deploy nginx --port=80 --type=NodePort
service/nginx exposed
```

```
vagrant@cliente:~$ kubectl get services
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.43.0.1       <none>        443/TCP        3h53m
nginx        NodePort    10.43.142.205   <none>        80:31907/TCP   25s
```

Como podemos apreciar tenemos el servicio nginx, tipo NodePort  y nos ha mapeado el puerto 31907.

Ahora podremos acceder a la ip del master junto al puerto que nos ha dado.

A continuación vamos a realizar la escabilidad de los contendores.
Actualmente contamos con un pod con un contenedor que esta ejecutando nuestra aplicación, vamos a hacer que se ejecute en varios pods.

Para ello vamos a realizar el siguiente comando:

```
vagrant@cliente:~$ kubectl scale --replicas=3 deploy/nginx
deployment.extensions/nginx scaled

vagrant@cliente:~$ kubectl get deploy,pod
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/nginx   3/3     3            3           3h2m

NAME                       READY   STATUS    RESTARTS   AGE
pod/nginx-5c7588df-5hcf8   1/1     Running   0          29s
pod/nginx-5c7588df-6x2ff   1/1     Running   0          3h2m
pod/nginx-5c7588df-6znqp   1/1     Running   0          29s
```

Como podemos apreciar con el segundo comando, se ha creado dos nuevos pods. Ahora se esta balanceando la carga entre los tres contenedores.

Con el siguiente comando podremos también ver que estos pods se están ejecutando en los distintos nodos o máquinas de nuestro cluster.
```

vagrant@cliente:~$ kubectl get pod -o wide
NAME                   READY   STATUS    RESTARTS   AGE    IP          NODE       NOMINATED NODE   READINESS GATES
nginx-5c7588df-5hcf8   1/1     Running   0          3m1s   10.42.0.7   maquina1   <none>           <none>
nginx-5c7588df-6x2ff   1/1     Running   0          3h5m   10.42.1.2   maquina2   <none>           <none>
nginx-5c7588df-6znqp   1/1     Running   0          3m1s   10.42.2.3   maquina3   <none>           <none>
```

En la columna **NODE**, podemos ver que se están ejecutando cada pod en un nodo diferente, haciendo un mejor balanceo y rendimiento de nuestro cluster.

Vamos a crear un cuarto pod para ver donde se situaria.

```
vagrant@cliente:~$ kubectl scale --replicas=4 deploy/nginx
deployment.extensions/nginx scaled

vagrant@cliente:~$ kubectl get pod -o wide
NAME                   READY   STATUS    RESTARTS   AGE     IP          NODE       NOMINATED NODE   READINESS GATES
nginx-5c7588df-5hcf8   1/1     Running   0          5m44s   10.42.0.7   maquina1   <none>           <none>
nginx-5c7588df-6x2ff   1/1     Running   0          3h7m    10.42.1.2   maquina2   <none>           <none>
nginx-5c7588df-6znqp   1/1     Running   0          5m44s   10.42.2.3   maquina3   <none>           <none>
nginx-5c7588df-f6kq5   1/1     Running   0          10s     10.42.1.3   maquina2   <none>           <none>
```

![](https://i.imgur.com/In1rcrW.png)

Podremos ver que este último pod se esta ejecutando en el segundo nodo **(maquina2)**.

Esta herramiento también nos proporciona **tolerancia a fallo**, vamos a ver un ejemplo.

En el caso de borrar un pod, ya sea porque lo borramos a mano o falla, se va a crear un nuevo pod.

```
vagrant@cliente:~$ kubectl get pod -o wide
NAME                   READY   STATUS    RESTARTS   AGE     IP          NODE       NOMINATED NODE   READINESS GATES
nginx-5c7588df-5hcf8   1/1     Running   0          5m44s   10.42.0.7   maquina1   <none>           <none>
nginx-5c7588df-6x2ff   1/1     Running   0          3h7m    10.42.1.2   maquina2   <none>           <none>
nginx-5c7588df-6znqp   1/1     Running   0          5m44s   10.42.2.3   maquina3   <none>           <none>
nginx-5c7588df-f6kq5   1/1     Running   0          10s     10.42.1.3   maquina2   <none>           <none>

vagrant@cliente:~$ kubectl delete pod nginx-5c7588df-6x2ff
pod "nginx-5c7588df-6x2ff" deleted

vagrant@cliente:~$ kubectl get pod -o wide
NAME                   READY   STATUS    RESTARTS   AGE     IP          NODE       NOMINATED NODE   READINESS GATES
nginx-5c7588df-5hcf8   1/1     Running   0          11m     10.42.0.7   maquina1   <none>           <none>
nginx-5c7588df-6znqp   1/1     Running   0          11m     10.42.2.3   maquina3   <none>           <none>
nginx-5c7588df-f6kq5   1/1     Running   0          6m10s   10.42.1.3   maquina2   <none>           <none>
nginx-5c7588df-kh9ks   1/1     Running   0          4s      10.42.2.4   maquina3   <none>           <none>
```

Podemos ver que hemos eliminado un pod, pero el despliegue nos asegura que siempre vamos a tener los pods que hemos configurado.

## Despligue de una aplicación.

Ya sabemos **desplegar nginx en kubernetes** como un **escenario de pruebas**, ahora vamos a implantar una **aplicación real**, como es el caso de **Let's Chat**.

Los fichero que vamos a configurar son los siguientes. Donde pondremos en cada uno el nombre y las especificaciones que va a tener los despliegues.

```
vagrant@cliente:~/letschat$ ls
ingress.yaml              letschat-srv.yaml      mongo-srv.yaml
letschat-deployment.yaml  mongo-deployment.yaml
```

En el caso de nginx lo hemos configurado en la misma línea de ejecución del deploy, pero en este caso lo haremos de una manera más **ordenada y profesional** realizandolo en un fichero de configuración. Este uso es más **eficiente** por si tenemos que pasarlo a otro compañero y tener exactamente el mismo escenario.

Primero tenemos que tener el despliegue de mongodb, el cual es la base de datos que utiliza la aplicación let's chat.

Este fichero contiene lo siguiente:

```
vagrant@cliente:~/letschat$ nano mongo-deployment.yaml 

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: mongo
  labels:
    name: mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      name: mongo
  template:
    metadata:
      labels:
        name: mongo
    spec:
      containers:
      - name: mongo
        image: mongo
        ports:
          - name: mongo
            containerPort: 27017
```

Para crear este despliegue de mongodb pondremos lo siguiente: 

```
vagrant@cliente:~/letschat$ kubectl create -f mongo-deployment.yaml 
deployment.extensions/mongo created
```

Ya lo tendremos creado y ahora se estan creado los pods y el despligue.

```
vagrant@cliente:~/letschat$ kubectl get deploy,pod
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/mongo   1/1     1            1           92s

NAME                         READY   STATUS    RESTARTS   AGE
pod/mongo-854684b4c5-ffz66   1/1     Running   0          92s
```

A continuación, vamos a desplegar let's chat. Primero vamos a ver el fichero de configuración que vamos a utilizar para el despliegue.

```
vagrant@cliente:~/letschat$ nano letschat-deployment.yaml 

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: letschat
  labels:
    name: letschat
spec:
  replicas: 1
  selector:
    matchLabels:
      name: letschat
  template:
    metadata:
      labels:
        name: letschat
    spec:
      containers:
      - name: letschat
        image: sdelements/lets-chat
        ports:
          - name: http-server
            containerPort: 8080
```

Creamos el despligue:

```
vagrant@cliente:~/letschat$ kubectl create -f letschat-deployment.yaml 
deployment.extensions/letschat created
```

Ya tendremos el deploy y el pod creados. Como podemos ver solamente tarda unos segundos, a pensar de que tiene que bajar la imagen.

```
vagrant@cliente:~/letschat$ kubectl get deploy,pod
NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/letschat   1/1     1            1           27s
deployment.extensions/mongo      1/1     1            1           5m42s

NAME                           READY   STATUS    RESTARTS   AGE
pod/letschat-bc787f6b8-hxkld   1/1     Running   0          27s
pod/mongo-854684b4c5-ffz66     1/1     Running   0          5m42s
```

Ahora vamos a ver donde se esta ejecutando cada despliegue.

```
vagrant@cliente:~/letschat$ kubectl get pod -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP          NODE       NOMINATED NODE   READINESS GATES
letschat-bc787f6b8-hxkld   1/1     Running   3          2m57s   10.42.1.3   maquina2   <none>           <none>
mongo-854684b4c5-ffz66     1/1     Running   0          8m12s   10.42.3.2   maquina3   <none>           <none>
```

Podemos apreciar que se están ejecutando en nodos diferente.

Ya estarian los servicios funcionando ahora tendremos que **acceder a ellos**. Nosotros no necesitamos entrar a MongoDB desde el exterior, solamente necesitamos que la aplicación let's chat pueda acceder al servicio.

Vamos a crear un servicio tipo **ClusterIP para MongoDB** y para **Let's Chat será de tipo NodePort** para poder acceder desde el exterior.

Vamos a crear el siguiente fichero.

- **Servicio MongoDB**

```
vagrant@cliente:~/letschat$ nano mongo-srv.yaml 

apiVersion: v1
kind: Service
metadata:
  name: mongo
spec:
  ports:
  - name: mongo
    port: 27017
    targetPort: mongo
  selector:
    name: mongo
```

- **Servicio Let's Chat**

```
vagrant@cliente:~/letschat$ nano letschat-srv.yaml 

apiVersion: v1
kind: Service
metadata:
  name: letschat
spec:
  type: NodePort
  ports:
  - name: http
    port: 8080
    targetPort: http-server
  selector:
    name: letschat
```

Ya podremos crear los servicios para ambos.

```
vagrant@cliente:~/letschat$ kubectl create -f mongo-srv.yaml 
service/mongo created

vagrant@cliente:~/letschat$ kubectl create -f letschat-srv.yaml 
service/letschat created
```

Ahora vamos a ver los servicios que se han creado:

```
vagrant@cliente:~/letschat$ kubectl get services
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP   10.43.0.1      <none>        443/TCP          48m
letschat     NodePort    10.43.170.68   <none>        8080:31647/TCP   2m40s
mongo        ClusterIP   10.43.78.230   <none>        27017/TCP        3m1s
```

Como se puede apreciar para mongo no ha MongoDB no ha mapeado ninguna IP, ya que el unico servicio que va a poder acceder a él será letschat, mientras que para letschat si se ha mapeado el puerto.

A través de la IP del master y con el puerto ya podremos entrar en la aplicación.

## Conclusión

De una manera muy sencilla hemos instalado un cluster de kubernetes con la herramienta k3s y hemos desplegado una aplicación de pruebas. 

Con esta demostración, se puede observar las posibilidades que nos ofrece esta herramienta.
