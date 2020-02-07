**PRACTICA ANGULAR CON NGNIX CON DOCKER PASANDO API-ENVIRONMENT**
------------------------------------------------------------------

Tutorial didáctico para eventos y meetups sobre ANGULAR y DOCKER.

Para este tutorial es necesario tener los siguientes requerimientos:

- Node: https://nodejs.org
- NPM: https://www.npmjs.com
- Angular CLI: https://cli.angular.io
- Angular Oficial : https://angular.io
- Angular Tutorial: https://angular.io/tutorial
- Docker en equipo local: https://www.docker.com

### Configurar version node v13.7

Abrir una shell para instalar version node v13.7
```
$ node --version (para comprobar version actual)
$ nvm install 13.7.0 (ver nvm github para ayuda de cada comando)
$ nvm use 13.7.0
$ node --version 
```

### Crear Aplicacion Angular HelloWorld

Ejecutar los comandos desde consola para crear nueva aplicacion Angular, esto compilará y ejecutar la aplicacion en http://localhost:4200
```
$ npm install -g @angular/cli (si no lo tenemos instalado
$ ng new angular-hello
$ cd angular-hello
$ ng serve
.....

** Angular Live Development Server is listening on localhost:4200, open your browser on http://localhost:4200/ **
Compiled successfully.
```
Abrimos un navegador y accedemos a http://localhost:4200 para ver la aplicacion ejecutandose en el browser.

### Crear Dockerfile para generar imagen simple y ejecutar contenedor Docker

Crear archivo Dockerfile en el mismo directorio que aplicacion Angular
```
FROM node:13.7-alpine
WORKDIR  /app
COPY ./ /app/
RUN npm install
EXPOSE 4200
ENTRYPOINT npm start
```

Construir la imagen del archivo Dockerfile y veremos los Steps definidos en el Dockerfile

```
 $ docker build --rm -f "Dockerfile" -t angularhello:v1 "."
 
  # Simplico las trazas del Build para no tener tanto texto
    Sending build context to Docker daemon.....

    Step 1/6 : FROM node:13.7-alpine
    Step 2/6 : WORKDIR  /app
    Step 3/6 : COPY ./ /app/
    Step 4/6 : RUN npm install
    Step 5/6 : EXPOSE 4200
    Step 6/6 : ENTRYPOINT npm start
    Removing intermediate container
    Successfully built e671f4d19609
    Successfully tagged angularhello:v1
```

Mostrar las imagenes que acabamos de construir

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
angularhello        v1                  e671f4d19609        11 minutes ago      468MB 
angularpoc          v1                  b162eb20c3f4        About an hour ago   499MB 
node                13.7-alpine         b809734bb743        2 weeks ago         113MB 
```

Levantar el contenedor para ejecutar aplicacion Angular


