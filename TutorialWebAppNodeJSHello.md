**PRACTICA AZURE WEBAPP-NODEJS**
---------------------------------------------------

- Consola CLI:  [az login] o [https://shell.azure.com]
- Ejemplo: https://github.com/Azure-Samples/nodejs-docs-hello-world/archive/master.zip
- Documentacion: https://docs.microsoft.com/es-es/azure/app-service/app-service-web-get-started-nodejs


### Configurar entorno Azure para Node
Pasos para crear y configurar entorno AppService para Node

```
$ az group create --name WebAppsNode-recursos --location northeurope

$ az appservice plan create --name WebAppsNodePlan --resource-group WebAppsNode-recursos --sku FREE

$ az webapp create --resource-group WebAppsNode-recursos --plan WebAppsNodePlan --name appwebnode01

$ az webapp config appsettings set --resource-group WebAppsNode-recursos --name appwebnode01 --settings WEBSITE_NODE_DEFAULT_VERSION=10.14.1
```

Visitar la web creada (sin aplicacion): https://appwebnode01.azurewebsites.net

### Deploy directo con az CLI
- Compilar proyecto "ng build --prod" en la carpeta dist
- Comprimir proyecto generado en carpeta dist en una archivo ZIP
- Deploy con el comando de az:

```
$ az webapp deployment source config-zip --resource-group=WebAppsNode-recursos --name=appwebnode01 --src C:\rutaproyectos\...\dist\deploy.zip
```

### Deploy del proyecto HelloNode con Kudu y ZipDeployUI
- Compilar proyecto "ng build --prod" en la carpeta dist
- Comprimir proyecto generado en carpeta dist en una archivo ZIP
- Visitar Kudu: https://appwebnode01.scm.azurewebsites.net/ZipDeployUI
- Arrastrar el archivo ZIP sobre el entorno Kudu y se desplegará automaticamente.

### Ver WebApp funcionando en Azure
- Visitar la web creada: https://appwebnode01.azurewebsites.net
- Veremos ahora el mensaje "Hello World" que esta en index.js
- Podemos editar index.js directamente en Kudu y cambiar el mensaje

### Borrar recursos Azure

```
$ az group delete --name WebAppsNode-recursos 
```


