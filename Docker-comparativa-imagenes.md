**PRACTICA COMPARAR TAMAÑO DIFERENTES IMAGES CON DOCKER**
------------------------------------------------------------------

Tutorial didáctico para eventos y meetups sobre DOCKER.

Para este tutorial es necesario tener los siguientes requerimientos:

- Cuenta en DockerHub: https://hub.docker.com
- Docker en equipo local: https://www.docker.com
- Visual Studio Code, otros editores
- Docker Tutorial: https://www.tutorialspoint.com/docker/index.htm

### Descargar diferentes imagenes desde linea de comandos

Ejecutar los comandos desde consola donde para cada una de las imagenes veremos la info en consola "Using default tag: latest..." y mas logs que no muestro para ahorrar texto.

```
$ docker pull alpine
  
$ docker pull busybox

$ docker pull debian

$ docker pull centos

$ docker pull ubuntu

$ docker pull oraclelinux

$ docker pull nginx

$ docker pull mysql 

$ docker pull bitnami/minideb:latest
```

### Tamaño de las diferentes imagenes 

Ahora consultaremos el tamaña de las imagenes descargadas, como vemos cada imagen tiene tamaños distintos y dependerán de las capas/contenidos que el autor ha incluido para crear la imagen completa.

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
bitnami/minideb     latest              65f1ca5e4251        1 weeks ago        53.7MB
mysql               latest              29e0ae3b69b9        1 weeks ago         484MB
oraclelinux         latest              5bfa048e0f42        1 weeks ago         234MB
centos              latest              5182e96772bf        1 weeks ago         200MB
ubuntu              latest              735f80812f90        1 weeks ago        83.5MB
nginx               latest              c82521676580        1 weeks ago         109MB
debian              latest              3bbb526d2608        1 weeks ago         101MB
busybox             latest              af2f74c517aa        1 weeks ago        3.41MB
alpine              latest              11cd0b38bc3c        1 weeks ago        4.41MB
```

### Descargar y ejecutar aplicaciones docker para ejemplos

Ejemplo para comparar imagenes NGINX y aplicaciones web/api en nuestro equipo local y despues en AKS.
Descargamos y ejecutamos las imagenes directamente con docker run...
```
# varias imagenes de nginx
$ docker run –p 8080:80 –d nginx
$ docker run -p 8081:80 -d kitematic/hello-world-nginx
$ docker run -p 8082:80 -d dockerbogo/docker-nginx-hello-world

# imagenes santi de ANGULAR, ASP.NET Core MVC y WEBAPI - 230MB
$ docker run -d -p 8083:80 santimacnet/angularhello:v2
$ docker run -d -p 8084:80 santimacnet/meetupweb
$ docker run -d -p 8085:80 santimacnet/aspnetcoremvc-hello
$ docker run -d -p 8086:80 santimacnet/aspnetcorewebapi-hello

# imagen grande Welcome to Azure Container Service (AKS) - 900MB
$ docker run -d -p 8088:80  neilpeterson/aks-helloworld:v1

# consultar tamaños de las imagenes descargadas
$ docker images
```

### Deployment aplicaciones docker en AKS desce Dashboard

Ejemplo para deployment en AKS y saber los recursos cpu/ram de cada pod en AKS

Ver repo con más ejemplos: https://hub.docker.com/search?q=santimacnet&type=image

Cree un archivo *demo-akshello* y cópielo en el ejemplo siguiente de YAML:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aks-helloworld2020
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aks-helloworld2020
  template:
    metadata:
      labels:
        app: aks-helloworld2020
    spec:
      containers:
      - name: aks-helloworld2020
        image: neilpeterson/aks-helloworld:v1
        ports:
        - containerPort: 80
        env:
        - name: TITLE
          value: "Welcome to Azure Kubernetes Service (AKS)"
---
apiVersion: v1
kind: Service
metadata:
  name: aks-helloworld2020
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: aks-helloworld2020
```



### Consultando contenido de las diferentes imagenes 

Para saber que incluye cada imagen, según la distro utilizada encontraremos diferentes paquetes instalados en cada una de ellas. 

El comando para ver los paquetes instalados depende de cada distro: Centos (yum), Debian (apt) y todos los demas que encontrareis en esta lista: https://es.wikipedia.org/wiki/Categor%C3%ADa:Gestores_de_paquetes_Linux

Para saber la version del Kernel Linux que esta corriendo usaremos algo como en este ejemplo:

```
# uname -a
Linux c8620b896e2d 4.9.0-7-amd64 #1 SMP Debian 4.9.110-3+deb9u1 (2018-08-03) x86_64 GNU/Linux
```




