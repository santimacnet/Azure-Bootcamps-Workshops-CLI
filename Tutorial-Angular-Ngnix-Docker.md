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

Crear archivo Dockerfile con un editor/IDE en el mismo directorio que aplicacion Angular
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
# COPY ./nginx.conf /etc/nginx/conf.d/default.conf
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


### Subir imagen a DockerHub Publico
Si queremos compartir nuestra imagen ya definitiva podemos subirla a DockerHub si disponemos de una cuenta
```
# Accederemos al DockerHub desde nuestro equipo
$ docker login

# Tageamos la imagen con formato nombre/imagen:version para DockerHub
$ docker tag angularpoc:v2 santimacnet/angularpoc:v2

# Publicamos la imagen con Tag en DockerHub
$ docker push santimacnet/angularhello:v2
```

Nota: si queremos conectarnos a un repo privado usaremos: docker login registry.privado.com

!!YEAH!! Si visitamos nuestra cuenta de DockerHub ya tendremos la imagen publicada.
