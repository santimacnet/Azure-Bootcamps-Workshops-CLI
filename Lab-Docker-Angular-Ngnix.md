**PRACTICA ANGULAR & NGINX CON DOCKER**
------------------------------------------------------------------

Tutorial didáctico para eventos y meetups sobre ANGULAR y DOCKER.

Para este tutorial es necesario tener los siguientes requerimientos:

- Node: https://nodejs.org
- NPM: https://www.npmjs.com
- Angular CLI: https://cli.angular.io
- Angular Oficial : https://angular.io
- Angular Tutorial: https://angular.io/tutorial
- Docker en equipo local: https://www.docker.com
- DockerHub NODE official images: https://hub.docker.com/_/node
- DockerHub NGINX official images: https://hub.docker.com/_/nginx

### Configurar version node v13.7

Abrir una shell para instalar version node v13.7
```
$ node --version (para comprobar version actual)
$ nvm install 13.7.0 (ver nvm github para ayuda de cada comando)
$ nvm use 13.7.0
$ node --version 
```

### Crear Aplicacion Angular HelloWorld

Ejecutar los comandos desde consola para crear y ejecutar nueva aplicacion Angular
```
$ npm install -g @angular/cli (si no lo tenemos instalado
$ ng new angular-hello
$ cd angular-hello
$ ng serve
.....

** Angular Live Development Server is listening on localhost:4200, open your browser on http://localhost:4200/ **
Compiled successfully.
```
Abrimos en navegador http://localhost:4200 para ver la aplicacion en el browser.

### Crear Dockerfile para generar imagen Docker

Crear archivo Dockerfile en el mismo directorio que aplicacion Angular
```
# STAGE-1: Construccion de Angular App
FROM node:13.7-alpine AS build
WORKDIR /app
COPY ./ /app/

# Compilar Angular App para produccion
RUN npm ci
RUN npm run build --prod
RUN mv /app/dist/angular-hello/* /app/dist/

# STAGE-2: Desplegar en NGINX
FROM nginx:1.17.8-alpine
COPY --from=build /app/dist/ /usr/share/nginx/html

# Mejorar configuracion de NGINX copiado archivo nginx.conf
COPY ./nginx.conf /etc/nginx/nginx.conf
```

### Crear configuracion NGINX para copiar en imagen Docker

Crear archivo nginx.conf en el mismo directorio que aplicacion Angular y Dockerfile
```
http {
  server {
    listen 80;
    server_name  localhost;
 
    root   /usr/share/nginx/html;
    include /etc/nginx/mime.types;
    index  index.html;

    ## Logging Settings
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    ## Location
    location / {
      try_files $uri $uri/ /index.html;
    }
  }
}
```

### Construir la imagen definida en Dockerfile y veremos los Steps definidos
```
 $ docker build --rm -f "Dockerfile" -t angularhello:v1 "."
 
  # Simplico las trazas del Build para no tener tanto texto
    Sending build context to Docker daemon.....

    Step 1/8 : FROM node:13.7-alpine AS build 
   
    Step 2/8 : WORKDIR /app
    
    Step 3/8 : COPY ./ /app/
    
    Step 4/8 : RUN npm ci
    
    Step 5/8 : RUN npm run build --prod 
    
    Step 6/8 : RUN mv /app/dist/angular-hello/* /app/dist/
    
    Step 7/8 : FROM nginx:1.17.8-alpine
    
    Step 8/8 : COPY --from=build /app/dist/ /usr/share/nginx/html
 
    Removing intermediate container
    Successfully built e671f4d19609
    Successfully tagged angularhello:v1
```

### Mostrar las imagenes que acabamos de construir
```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
angularhello        v1                  e671f4d19609        11 minutes ago      34.5MB 
angularpoc          v1                  b162eb20c3f4        About an hour ago   499MB 
node                13.7-alpine         b809734bb743        2 weeks ago         113MB 
```

### Levantar el contenedor para ejecutar aplicacion Angular
```
docker run --rm -d -p 8888:80 angularhello:v1
```

Abrimos en navegador http://localhost:8888 para ver la aplicacion ejecutandose en el browser a traves de NGINX.

!!GENIAL!! Ya tenemos nuestra aplicacion Angular funcionando en Docker y accediendo meditante NGINX


### Entrar al contenedor para ver NGINX y aplicacion Angular
```
# Desde windows command line
c:\> docker exec -it <id-container> /bin/sh
c:\> docker exec -it <id-container> //bin//sh

# Desde Visual Studio Code git/bash
$ winpty docker exec -it <id-container> /bin/sh
$ winpty docker exec -it <id-container> //bin//sh

# Carpeta de nginx con aplicacion angular index.html
$ cd usr/share/nginx/html
$ ls index.html
$ cat index.html

# Configuracion de nginx 
$ cd etc/nginx
$ cat nginx.conf

# Prueba de aplicacion Angular con CURL
$ apk add curl
$ curl localhost
```

NOTA: Podemos usar /bin/bash o /bin/sh dependiendo de la shell de nuestro contenedor.

### Subir imagen a DockerHub Publico
Si queremos compartir nuestra imagen ya definitiva podemos subirla a DockerHub si disponemos de una cuenta
```
# Tageamos la imagen con formato nombre/imagen:version para DockerHub
$ docker tag angularhello:v1 santimacnet/angularhello:v1

# Accederemos al DockerHub desde nuestro equipo
$ docker login
username: nombre
password: *******

# Publicamos la imagen con Tag en DockerHub
$ docker push santimacnet/angularhello:v1
```
Nota: Necesitaremos hacer login en DockerHub con nuestro usuario/password
Ref Docker: https://docs.docker.com/engine/reference/commandline/login

### Subir imagen a Azure Container Registry (Privado)
Si queremos compartir nuestra imagen en un repo privado como ACR hay que realizar algunos cambios
```
# Importante: Tageamos imagen con formato **registryname.azurecr.io/imagen:version** para subirla
$ docker tag angularhello:v1 registryname.azurecr.io/angularhello:v1

# Accederemos al Docker privado en Azure recomendado usar --password-stdin.
$ docker login registryname.azurecr.io -u <ServPrincipalID-guid> -p <ServPrincipalPwd-guid> 
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Login Succeeded

# Publicamos la imagen con Tag en DockerHub
$ docker push registryname.azurecr.io/angularhello:v1
The push refers to repository [registryname.azurecr.io/angularhello]
af0b15c8625b: Pushed   
```
Nota: Necesitaremos hacer login con un Service Principal en vez de usuario y password
Ref Azure : https://docs.microsoft.com/es-es/azure/container-registry/container-registry-get-started-docker-cli


### Limpiando el campamento 

Para terminar la practica si queremos borrar todas las imagenes y contenedores en nuestro equipo local usaremos siguiente comandos.

Antes de lanzar estos comandos estar seguros que no teneis otras imagenes o contenedores porque borra todo lo referente a Docker

```
$ docker kill $(docker ps -q)

$ docker rm $(docker ps -a -q)

$ docker rmi -f $(docker images -q)
```

!!YEAH!! Si visitamos nuestra cuenta de DockerHub ya tendremos la imagen publicada.
